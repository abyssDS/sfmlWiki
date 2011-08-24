[[A simple module|http://www.sfmluploads.org/file/43]] that loads resources using the physfs library for be used from ::LoadFromStream(). Also this project is an example that how must be implemented the sf::InputStream class. This module can be used freely.

**Usage**

```
sf::Music coolmusic;
sf::physfs musicstream("coolmusic.ogg");
coolmusic.OpenFromStream(musicstream);
```

**Notes**

* You must initialize first the physfs library and mount the paths.

* The file will be open only for reading.
