# Loading images in a thread (and displaying progress)

```cpp

    #include <SFML/System.hpp>
    #include <SFML/Graphics.hpp>
    #include <boost/filesystem.hpp>
    #include <cstring>
    
    typedef std::vector<std::string> FileList;
    typedef FileList::iterator FileListIterator;
    
    unsigned int findFiles(FileList& Files, const std::string& Directory)
    {
    	boost::filesystem::path Dir(Directory);
    
    	if (!boost::filesystem::exists(Dir))
    		return 0;
    
    	boost::filesystem::directory_iterator itEnd;
    	for (boost::filesystem::directory_iterator it(Dir); it != itEnd; ++it)
    	{
    		if (boost::filesystem::is_directory(it->status()))
    			findFiles(Files, it->path().string());
    		else if (boost::filesystem::is_regular(it->status()))
    			Files.push_back(it->path().string());
    		else; //Not a directory or regular file
    	}
    	return Files.size();
    }
    
    typedef std::vector<sf::Image*> ImageList;
    
    struct LoadingThreadInfo
    {
    	sf::RenderWindow& Window;
    	//Mutex for OpenGL context
    	sf::Mutex& GLMutex;
    
    	std::string& Filename;
    	//Mutex for Filename in main()
    	sf::Mutex& FilenameMutex;
    
    	//Progress (percentage)
    	float& Progress;
    	//Are we still loading?
    	bool& Loading;
    	ImageList& Images;
    };
    
    void LoadImages(void* Data)
    {
    	LoadingThreadInfo* Info = (LoadingThreadInfo*)Data;
    	sf::RenderWindow& Window = Info->Window;
    	float& Progress = Info->Progress;
    	bool& isLoading = Info->Loading;
    	sf::Mutex& GLMutex = Info->GLMutex;
    	sf::Mutex& FilenameMutex = Info->FilenameMutex;
    	ImageList& Images = Info->Images;
    	std::string& Filename = Info->Filename;
    
    	FileList ImageFiles;
    	findFiles(ImageFiles, "Images/");
    	std::size_t numFiles = ImageFiles.size();
    	printf("Found %d images\n", numFiles);
    	for (std::size_t i = 0; i < numFiles; ++i)
    	{
    		sf::Lock lockGL(GLMutex);
    		Window.SetActive();
    
    		FilenameMutex.Lock();
    		Filename = ImageFiles[i];
    		FilenameMutex.Unlock();
    
    		printf("Loading %s\n", ImageFiles[i].c_str());
    		sf::Image* newImage = new sf::Image;
    		if (newImage->LoadFromFile(ImageFiles[i]))
    			Images.push_back(newImage);
    		else
    			delete newImage;
    
    		Progress = static_cast<float>(i + 1) / static_cast<float>(numFiles) * 100.0f;
    		Window.SetActive(false);
    	}
    	isLoading = false;
    	FilenameMutex.Lock();
    	Filename.clear();
    	FilenameMutex.Unlock();
    }
    
    int main(int argc, char* argv[])
    {
    	sf::RenderWindow Window(sf::VideoMode(800, 600, 32), "", sf::Style::Close);
    	sf::String ProgressString;
    	sf::String FilenameString;
    	char Buffer[32];
    
    	bool isLoading = true;
    	float loadingProgress = 0.0f;
    	std::string Filename;
    	sf::Mutex GLMutex;
    	sf::Mutex FilenameMutex;
    	ImageList Images;
    	LoadingThreadInfo LoadingInfo = {Window, GLMutex, Filename, FilenameMutex, loadingProgress, isLoading, Images};
    
    	sf::Thread LoadingThread(&LoadImages, &LoadingInfo);
    	LoadingThread.Launch();
    
    	sf::Event Event;
    	bool Done = false;
    	while (!Done)
    	{
    		if (isLoading)
    		{
    			GLMutex.Lock();
    			Window.SetActive();
    		}
    		while (Window.GetEvent(Event))
    		{
    			switch (Event.Type)
    			{
    				case sf::Event::Closed:
    					Done = true;
    				break;
    			}
    		}
    		if (isLoading)
    		{
    			if (Filename.empty())
    			{
    				FilenameString.SetText("Loading...");
    			}
    			else
    			{
    				FilenameMutex.Lock();
    				FilenameString.SetText("Loading " + Filename);
    				FilenameMutex.Unlock();
    			}
    			sprintf_s(Buffer, sizeof(Buffer), "%.2f%%", loadingProgress);
    			ProgressString.SetText(Buffer);
    		}
    		else
    		{
    			sprintf_s(Buffer, sizeof(Buffer), "Succesfully loaded %d images", Images.size());
    			ProgressString.SetText(Buffer);
    		}
    		ProgressString.SetPosition(400 - ProgressString.GetRect().GetWidth() / 2, 500);
    		Window.Draw(ProgressString);
    		if (!Filename.empty())
    		{
    			FilenameString.SetPosition(400 - FilenameString.GetRect().GetWidth() / 2, 550);
    			Window.Draw(FilenameString);
    		}
    		Window.Display();
    		if (isLoading)
    		{
    			Window.SetActive(false);
    			GLMutex.Unlock();
    		}
    	}
    	for (std::size_t i = 0; i < Images.size(); ++i)
    		delete Images[i];
    
        return 0;
    }

```