# Create a simple image manager
I often see, that beginners are using object-classes which contains both sf::Image and sf::Sprite. Images need  a lot of memory on the one hand and are often used by more than one object on the other. So it is not sensible at all, to create the image within the object-class. If you have two objects which are using the same image, this image would still be loaded twice. So if you're using 100 ground-objects, your ground-image would be loaded 100 times, though it would be enough to load it once only and use it for all the ground sprites. 
Hence it would be very helpful to have an image-manager which handles the loading of all the images. But because it is easier to understand what an image-manger has to do, if you have an example, I wrote this little tutorial on creating such a manager.

## What do we want to do?
We want to write a very simple image-manager. Of course you could write such a manager for many types of resources(as music, fonts, shaders etc.), but this would be rather more complex. And my aim is to show beginners an easy way to handle images. So it would rather be confusing, if I wrote a very complex resource manager. 

## How am I supposed to use an image-manager?
Before we're getting into programming our image-manager, we should think of the usability and the design of it. It should not be very complicated to use it. 
So it would be nice if you could just do something like this, to create an image:
```c++
image_manager img_mgr;
sf::Sprite test_sprite;
test_sprite.SetImage( img_mgr.get_image( "test.png" ) );
```
This is what we're aiming at. We want to be able to load an image with only a single line of code. And it should be safe in use(even if an image does not exist) and not produce any runtime errors.

## Let's start programming!
Now we know, what we want to do, it is quite easy to create an image-manager-class:
```c++
class image_manager
{
public:
	image_manager();
	~image_manager();

private:
	image_manager( const image_manager& );
	image_manager& operator =( const image_manager& );

public:
	const sf::Image&	get_image( const std::string& filename );

private:
	std::map< std::string, sf::Image > images_;
};
```

"images_" is a map which binds every image to a std::string(its filename). If you call get_image, the image-map will be searched for the desired image and if it is not found, the image will be loaded and added to the map. Obviously get_image has to be able to do both finding an image which is already in the map and loading an image, if it hasn't been loaded before.
So let's talk about the implementation of get_image:
```c++
const sf::Image& image_manager::get_image( const std::string& filename )
{
	// Check, whether the image already exists
	for( std::map<std::string, sf::Image>::const_iterator it = images_.begin();
		 it != images_.end(); 
		 ++it)
	{
		if( filename == it->first )
		{
			std::cout << "DEBUG_MESSAGE: using existing image.\n";
			return it->second;
		}
	}
	
	// The image doesen't exists. Create it and save it.
	sf::Image image;
	if( image.LoadFromFile( filename ) )
	{
		images_[filename] = image;
		std::cout << "DEBUG_MESSAGE: loading image.\n";
		return images_[filename];
	}

	std::cout << "GAME_ERROR: Image was not found. It is filled with an empty image.\n";
	images_[filename] = image;
	return images_[filename];
}
```
In the first part of this function, as I've already mentioned, we're looking for the image and check, weather it has already been loaded. If not we have to load and register it. And if it is not possible to load any image it will be filled with an empty image to prevent runtime-errors(of course it would also be possible to throw an exception, if you want to).
Maybe it is sometimes necessary to unload an image. For this reason let's add some functions which have to remove some images:
```c++
void image_manager::delete_image( const sf::Image& image )
{
	for( std::map<std::string, sf::Image>::const_iterator it = images_.begin();
		 it != images_.end(); 
		 )
	{
		// compare the adresses
		if( &image == &it->second )
		{
			it = images_.erase( it );
		}
		else
		{
			++it;
		}
	}
}

void image_manager::delete_image( const std::string& filename )
{
	std::map<std::string, sf::Image>::const_iterator it = images_.find( filename );
	if( it != images_.end() )
		images_.erase( it );
}
```
But you should be really careful to use these functions. If you delete an image which is still in use it will be replaced with a nice white rectangle.

## But what about different directories?
You should now easily be able to load images using our code from above. But what to do, if you want to use images that are not in the main-directory of your project?
Of course you could simply put your directory in front of your image filename:
```c++
test_sprite.SetImage( img_mgr.get_image( "media/images/test.png" ) );
```
Or you could do this in the get_image function to load images from "media/images" all the time:
```c++
if( image.LoadFromFile( "media/images"/ + filename ) )
```
But this is not the best way to do it. It would be nice, if the user was able to decide where the images should be loaded from. The user should define some "resource-directorys" where all the images should be placed.
```c++
image_manager img_mgr;
img_mgr.add_resource_directory("media/" );
img_mgr.add_resource_directory("media/images/" );
```
This requires of course that we have to add some functions to our image-manager. At first we need a new list to handle all the directories:
```c++
std::vector< std::string > resource_directories_;
```
To add a new directory we will use:
```c++
void image_manager::add_resource_directory( const std::string& directory )
{
	// Check weather the path already exists
	for( std::vector<std::string>::const_iterator it  = resource_directories_.begin();
		 it != resource_directories_.end();
		++it )
	{
		// The path exists. So it isn't necessary to add id once more.
		if( directory == (*it) )
			return;
	}

	// insert the directory
	resource_directories_.push_back( directory );
}
```
Of course the user should also be able to remove directories:
```c++
void image_manager::remove_resource_directory( const std::string& directory )
{
	for( std::vector<std::string>::const_iterator it  = resource_directories_.begin();
		 it != resource_directories_.end(); )
	{
		// The path exists. So it isn't necessary to add id once more.
		if( directory == (*it) )
			it = resource_directories_.erase( it );
		else
			++it;
	}
}
```
And of course we now have to search in all directories for the images. Hence we must adjust our get_image-function:
```c++
const sf::Image& image_manager::get_image( const std::string& filename )
{
	// Check, whether the image already exists
	for( std::map<std::string, sf::Image>::const_iterator it = images_.begin();
		 it != images_.end(); 
		 ++it)
	{
		if( filename == it->first )
		{
			std::cout << "DEBUG_MESSAGE: " << filename << " using existing image.\n";
			return it->second;
		}
	}
	
	// The image doesen't exists. Create it and save it.
	sf::Image image;

	// Search project's main directory
	if( image.LoadFromFile( filename ) )
	{
		images_[filename] = image;
		std::cout << "DEBUG_MESSAGE: " << filename << " loading image.\n";
		return images_[filename];
	}

	// If the image has still not been found, search all registered directories
	for( std::vector< std::string >::iterator it = resource_directories_.begin();
		 it != resource_directories_.end();
		 ++it )
	{
		if( image.LoadFromFile( (*it) + filename ) )
		{
			images_[filename] = image;
			std::cout << "DEBUG_MESSAGE: " << filename << " loading image.\n";
			return images_[filename];
		}

	}

	std::cout << "GAME_ERROR: Image was not found. It is filled with an empty image.\n";
	images_[filename] = image;
	return images_[filename];
}
```
And that's it. Now for example it ise very easy to use an external script which defines the resource directories. It will be parsed when the application has started and it will be very easy to test a new tileset or a few new images by changing the directory in the script. 

## Time for testing!
To make sure our new image-manager works correctly, we will write a test-application which is supposed to load a few images.
```c++
int main()
{
	sf::RenderWindow window(sf::VideoMode( 1024, 768 ), "Image-Manager" );

	image_manager img_mgr;
	img_mgr.add_resource_directory("media/" );
	img_mgr.add_resource_directory("media/images/" );
	// Just for testing we're removing a directory
	img_mgr.remove_resource_directory("media/images/" );
	
	sf::Sprite test_sprite[20];
	sf::Sprite other_sprite[20];
	// initialise all sprites
	for( int i = 0; i < 20; ++i )
	{
		test_sprite[i].SetImage( img_mgr.get_image( "test.png" ) );
		other_sprite[i].SetImage( img_mgr.get_image( "other.png" ) );

		test_sprite[i].SetPosition( i*40, 0 );
		other_sprite[i].SetPosition( i*50, 100 );
	}

    while (window.IsOpened())
    {
        sf::Event Event;
        while (window.GetEvent(Event))
        {
            if (Event.Type == sf::Event::Closed)
                window.Close();
			// You should not do this, but we will; just to verify that it works correctly ;)
			if (Event.Type == sf::Event::KeyPressed && Event.Key.Code == sf::Key::Space )
				img_mgr.delete_image( "test.png" );
			if (Event.Type == sf::Event::KeyPressed && Event.Key.Code == sf::Key::LControl )
				img_mgr.delete_image( img_mgr.get_image( "other.png" ) );
        }

        window.Clear();

		// Draw sprites
		for( int i = 0; i < 20; ++i )
		{
			window.Draw( test_sprite[i] );
			window.Draw( other_sprite[i] );
		}

        window.Display();
    }

    return EXIT_SUCCESS;
}
```

I hope that I was able to help you and that you understood how to handle images to prevent reloading them all the time. You can also download this article as [PDF](http://dl.dropbox.com/u/2541480/image_manager_tutorial.pdf).

Have a look at the whole source code of the image-manager:

## image_manager.h
```c++
#ifndef IMAGE_MANAGER_H_
#define IMAGE_MANAGER_H_
class image_manager
{
public:
	image_manager();
	~image_manager();

private:
	image_manager( const image_manager& );
	image_manager& operator =( const image_manager& );

public:
	const sf::Image&	get_image( const std::string& filename );
	void				delete_image( const sf::Image& image );
	void				delete_image( const std::string& filename );
	void				add_resource_directory( const std::string& directory );
	void				remove_resource_directory( const std::string& directory );

private:
	std::map< std::string, sf::Image > images_;
	std::vector< std::string > resource_directories_;
};
#endif
```

## image_manager.cpp
```c++
#include <map>
#include <iostream>
#include <SFML/Graphics.hpp>
#include <boost/filesystem.hpp>
#include "image_manager.h"

image_manager::image_manager() : images_(), resource_directories_()
{
}

image_manager::~image_manager()
{
	images_.clear();
	resource_directories_.clear();
}

const sf::Image& image_manager::get_image( const std::string& filename )
{
	// Check, whether the image already exists
	for( std::map<std::string, sf::Image>::const_iterator it = images_.begin();
		 it != images_.end(); 
		 ++it)
	{
		if( filename == it->first )
		{
			std::cout << "DEBUG_MESSAGE: " << filename << " using existing image.\n";
			return it->second;
		}
	}
	
	// The image doesen't exists. Create it and save it.
	sf::Image image;

	// Search project's main directory
	if( image.LoadFromFile( filename ) )
	{
		images_[filename] = image;
		std::cout << "DEBUG_MESSAGE: " << filename << " loading image.\n";
		return images_[filename];
	}

	// If the image has still not been found, search all registered directories
	for( std::vector< std::string >::iterator it = resource_directories_.begin();
		 it != resource_directories_.end();
		 ++it )
	{
		if( image.LoadFromFile( (*it) + filename ) )
		{
			images_[filename] = image;
			std::cout << "DEBUG_MESSAGE: " << filename << " loading image.\n";
			return images_[filename];
		}

	}

	std::cout << "GAME_ERROR: Image was not found. It is filled with an empty image.\n";
	images_[filename] = image;
	return images_[filename];
}

void image_manager::delete_image( const sf::Image& image )
{
	for( std::map<std::string, sf::Image>::const_iterator it = images_.begin();
		 it != images_.end(); 
		 )
	{
		if( &image == &it->second )
		{
			it = images_.erase( it );
		}
		else
		{
			++it;
		}
	}
}

void image_manager::delete_image( const std::string& filename )
{
	std::map<std::string, sf::Image>::const_iterator it = images_.find( filename );
	if( it != images_.end() )
		images_.erase( it );
}

void image_manager::add_resource_directory( const std::string& directory )
{
	// Check weather the path already exists
	for( std::vector<std::string>::const_iterator it  = resource_directories_.begin();
		 it != resource_directories_.end();
		++it )
	{
		// The path exists. So it isn't necessary to add id once more.
		if( directory == (*it) )
			return;
	}

	// insert the directory
	resource_directories_.push_back( directory );
}

void image_manager::remove_resource_directory( const std::string& directory )
{
	for( std::vector<std::string>::const_iterator it  = resource_directories_.begin();
		 it != resource_directories_.end(); )
	{
		// The path exists. So it isn't necessary to add id once more.
		if( directory == (*it) )
			it = resource_directories_.erase( it );
		else
			++it;
	}
}
```