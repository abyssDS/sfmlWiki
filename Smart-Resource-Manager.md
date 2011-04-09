This is a system to manage resources with a bit of more intelligence put into it. You specify a maximum size for the resource manager to work with and when it reaches that size it will look after if any of the already loaded resources is not in use any more and unload them and use their space instead. The biggest advantage of this system is that the system already have a large allocated chunk of memory so the resources don't get fragmented and we don't lock any threads that might exist every time we want to allocate a new resource. The **ResourceManager::Get** method has been made thread-safe but the resource handles haven't. Resources should never be shared between threads!

### Simple Example

```cpp
class ImageManager : public ResourceManager<sf::Image, std::string, 100> 
{ 
protected: 
   bool Load(sf::Image &resource, const std::string &path) const 
   { 
      return resource.LoadFromFile(path); 
   } 
}; 

typedef Resource<sf::Image, std::string, 100> ImageResource; 

int main() 
{ 
   std::string paths[1000] = {/* Define a 1000 paths */}; 
   ImageManager manager; 
   for(unsigned int index = 0; index < 1000; index++) 
   { 
      ImageResource test = manager.Get(paths[index]); 
      std::cout << test->GetWidth() << ", " << test->GetHeight() << std::endl; 
      sf::Sprite sprite(*test); 
   } 
   return 0; 
}
```

This manager will hold the images and can only contain 100 of them but if we look at the for-loop we distinctively can see that we will load all images specified by the array **paths**. The image is loaded and kept track of, then returned trough it's **Resource handle**. We can interact with the image just like if the resource handle was a pointer to the image. And at the end of the for-loop the resource object will be destructed and the only reference to it terminated. And thus the memory is free'd up for grabs again if we need it. In reality the image will stay in memory until we run out of space which we will at image 100. If we would have saved another handle outside the for-loop that doesn't get destructed then the image can't be loaded out of memory until that resource handle is also gone thus giving us the _"smart"_ in this resource manager.

Feel free to download it, modify and use it. I encourage you to experiment with it.
[Smart Resource Manager](http://www.groogy.se/smart-resource-manager.rar)