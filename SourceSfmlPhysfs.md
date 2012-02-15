# A class that adapts PhysFS to SFML with a custom sf::InputStream

[[A simple module|http://www.sfmluploads.org/file/89] that loads resources using the physfs library for be used with ::OpenFromStream(). Also this module is an example that how must implemented the InputStream class. This module can be used freely.

**Usage**

```
sf::Music coolmusic;
sf::physfs musicstream("coolmusic.ogg");
coolmusic.OpenFromStream(musicstream);
```

**Notes**

* You must initialize first the physfs library and mount the paths.

* The file will be open only for reading.
