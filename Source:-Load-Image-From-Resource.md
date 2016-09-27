# Load Image From Resource

An example of how to load data from a [Visual Studio resource script](http://msdn.microsoft.com/en-us/library/7zxb70x7) (file extension .rc) into sf::Image.

## Usage

```cpp
sf::Image spriteBmpImage = LoadImageFromResource("spritebmp");
sf::Image spritePngImage = LoadImageFromResource("spritepng");
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
sf::Image LoadImageFromResource(const std::string& name)
{
	HRSRC rsrcData = FindResource(NULL, name.c_str(), RT_RCDATA);
	if (!rsrcData)
		throw std::runtime_error("Failed to find resource.");
		
	DWORD rsrcDataSize = SizeofResource(NULL, rsrcData);
	if (rsrcDataSize <= 0)
		throw std::runtime_error("Size of resource is 0.");

	HGLOBAL grsrcData = LoadResource(NULL, rsrcData);
	if (!grsrcData)
		throw std::runtime_error("Failed to load resource.");

	LPVOID firstByte = LockResource(grsrcData);
	if (!firstByte)
		throw std::runtime_error("Failed to lock resource.");

	sf::Image image;
	if (!image.loadFromMemory(firstByte, rsrcDataSize))
		throw std::runtime_error("Failed to load image from memory.");

	return image;
}
```
Note: The shown code may lead to an unnecessary copy if the compiler doesn't support NRVO. In such a case, it would be possible to use the image as an output parameter, or return ``std::unique_ptr<sf::Image>``.

Note 2: The resources may need to be named with all capitals letters inside your resources.rc file
``SPRITEBMP RCDATA "sprite.bmp"``