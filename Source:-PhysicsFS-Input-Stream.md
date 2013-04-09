```
#include <iostream>
#include <physfs/physfs.h>
#include <SFML/Graphics.hpp>
class PhysicsStream : public sf::InputStream
{
public:
	PhysicsStream(PHYSFS_File * file):m_File(file){}
	virtual ~PhysicsStream()
	{
		if(m_File) PHYSFS_close(m_File);
	}
	virtual sf::Int64 read(void* data, sf::Int64 size)
	{
		if(!m_File) return -1;
		//physics returns -1 on error or number of 'objects' read, we assume our objects are single bytes
		//and request to read size amount of them
		return PHYSFS_read(m_File,data,1,static_cast<PHYSFS_uint32>(size));
	}
	virtual sf::Int64 seek(sf::Int64 position)
	{
		//seek return 0 on error and non zero on success
		if(!m_File||!PHYSFS_seek(m_File,position)) return -1;
		return tell();
	}
	virtual sf::Int64 tell()
	{
		if(!m_File) return -1;
		//physics returns -1 on error or offset in bytes just like SFML wants
		return PHYSFS_tell(m_File);
	}
	virtual sf::Int64 getSize()
	{
		if(!m_File) return -1;
		//physics also returns length of file or -1 on error just like SFML wants
		return PHYSFS_fileLength(m_File);
	}
private:
	PHYSFS_File * m_File;
};
int main(int argc,char * argv[])
{
	PHYSFS_init(argv[0]);
	PHYSFS_addToSearchPath("MyWonderfulArchive.7z",1);
	PhysicsStream wonderfullStream(PHYSFS_openRead("MyWonderfulPictureInArchive.png"));
	sf::Texture tex;
	tex.loadFromStream(wonderfullStream);
	sf::Sprite sprite(tex);
	sf::RenderWindow app(sf::VideoMode(600,480),"PhysFS");
	while(app.isOpen())
	{
		sf::Event eve;
		while(app.pollEvent(eve)) if(eve.type==sf::Event::Closed) app.close();
		app.clear();
		app.draw(sprite);
		app.display();
	}
	PHYSFS_deinit();
}
```