```cpp
/*
 *	Loads an sf::Image from RCDATA in a resource file
 *
 *	eg. resource.rc:
 *	spritebmp	RCDATA	"sprite.bmp"
 *	spritepng	RCDATA	"sprite.png"
 *
 *	eg. usage:
 *	sf::Image* spriteBmp = LoadImageFromResource("spritebmp");
 *	sf::Image* spritePng = LoadImageFromResource("spritepng");
 *
 */
sf::Image* LoadImageFromResource(std::string name)
{
	sf::Image* image = NULL;

	HRSRC rsrcBmp = FindResource(NULL, name.c_str(), RT_RCDATA);
	if (!rsrcBmp)
		throw std::string("Failed to find resource.");
		
	DWORD rsrcBmpSize = SizeofResource(NULL, rsrcBmp);
	if (rsrcBmpSize <= 0)
		throw std::string("Size of resource is 0.");

	HGLOBAL grsrcBmp = LoadResource(NULL, rsrcBmp);
	if (!grsrcBmp)
		throw std::string("Failed to load resource.");

	LPVOID firstByte = LockResource(grsrcBmp);
	if (!firstByte)
		throw std::string("Failed to lock resource.");

	image = new sf::Image();
	if (!image.loadFromMemory(firstByte, rsrcBmpSize))
		throw std::string("Failed to load image from memory.");

	return image;
}
```