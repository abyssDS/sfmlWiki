# Load Image From Resource

An example of how to load data from a resource (.rc) into sf::Image.

## Usage

```cpp
sf::Image* spriteBmpImage = LoadImageFromResource("spritebmp");
sf::Image* spritePngImage = LoadImageFromResource("spritepng");
```

resource.rc
```cpp
spritebmp	RCDATA	"sprite.bmp"
spritepng	RCDATA	"sprite.png"
```

## C++ Source

```cpp
/**
 *	Creates a new sf::Image and
 *	loads it with image data from a resource (.rc) file
 */
sf::Image* LoadImageFromResource(std::string name)
{
	sf::Image* image = NULL;

	HRSRC rsrcData = FindResource(NULL, name.c_str(), RT_RCDATA);
	if (!rsrcData)
		throw std::string("Failed to find resource.");
		
	DWORD rsrcDataSize = SizeofResource(NULL, rsrcData);
	if (rsrcDataSize <= 0)
		throw std::string("Size of resource is 0.");

	HGLOBAL grsrcData = LoadResource(NULL, rsrcData);
	if (!grsrcData)
		throw std::string("Failed to load resource.");

	LPVOID firstByte = LockResource(grsrcData);
	if (!firstByte)
		throw std::string("Failed to lock resource.");

	image = new sf::Image();
	if (!image.loadFromMemory(firstByte, rsrcDataSize))
		throw std::string("Failed to load image from memory.");

	return image;
}
```