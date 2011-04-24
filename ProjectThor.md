![Thor Logo](http://www.bromeon.ch/thor/thor.png)

Thor is a cross-platform, open-source library which provides several extensions to SFML. It includes many features on top of the basic multimedia functionality supplied with SFML. The Thor C++ Library is intended to relieve SFML users from implementing often-used functionality related to graphics and game programming again and again.

## Features

The Thor C++ library comes with the following modules:

### Vectors
- 2D Vector algebra functions: Dot product, compute length and angles, ...
- 3D Vector algebra functions: Dot and cross product, compute length and angles, ...
- PolarVector: 2D vector class that stores polar coordinates

### Geometry
- Zone class hierarchy for geometric figures
- Base classes for positionable, rotatable and scalable objects

### Particles
- ParticleSystem class based on sf::Image, stores and manages particles (lifetime, transformations, color, ...)
- Customizable Emitter and Affector classes

### Events
- SfmlEventSystem class to handle SFML events via callbacks
- EventSystem template for user-defined events

### Resources
- ResourceManager class template that allows safe and easy loading, storage and access to resources
- ResourcePtr provides shared access to resources
- Predefined helper classes for SFML resources like sf::Image

### Multimedia
- ConcaveShape class: As an extension to sf::Shape, it works with concave shapes
- Arrow class to visualize vectors
- Color blending and simple construction of color gradients
- String conversion functions for a few SFML classes like sf::Vector

### Time
- StopWatch class: Wrapper around sf::Clock for pausable clocks
- Timer class: Countdown mechanism
- TriggeringTimer class: Countdown invoking callbacks at expiration

### Tools
- SingleDispatcher: Dynamic dispatch to unary functions
- DoubleDispatcher: Dynamic dispatch to binary functions (multimethods)
- Other utility like foreach macros

### SmartPtr
- ScopedPtr: Local RAII smart pointer
- MovedPtr: Movable smart pointer implementation
- CopiedPtr: Smart pointer with deep copy semantics

### Math
- Generalization of trigonometry functions
- Random number generator based on TR1
- Delaunay triangulation

## License
Thor is completely free to use for open-source and closed-source projects. Like SFML, it is licensed under the very permissive zlib/libpng license.

## Download
The project homepage of Thor is located at [[http://www.bromeon.ch/thor]]. There you come across an API documentation, some tutorials and several download packages. The current version of the SDK is 1.0. The library can be built using CMake, for a few configurations there are also precompiled binaries available.

If you like to experience the latest features and modifications, you can also get the code via SVN at [[http://www.bromeon.ch/svn/thor/trunk]].

To install Thor, you need a TR1 conforming standard library implementation and SFML 2. A compatible Git revision of SFML is provided on the homepage.

## Examples
Full-fledged example codes are included in the official SDK, but here are a few code snippets that give you a quick impression of the functionality Thor provides.

### Vectors

    sf::Vector2f a(4.2f, 5.6f);
    sf::Vector2f b(7.8f, 1.3f);
    float d = thor::DotProduct(a, b);
    float w = thor::Angle(a, b);
    sf::Vector2f u = thor::UnitVector(a);

### Particles

    // Color gradient from red to blue to green
    thor::ColorGradient gradient = thor::CreateGradient(sf::Color::Red)(1)(sf::Color::Blue)(2)(sf::Color::Green);

    // Load image, create particle system, add emitters and affectors
    sf::Image image = ...;
    thor::ParticleSystem system(image);
    system.AddEmitter( thor::DirectionalEmitter::Create(50.f, 3.f) ); // 50 per second, each lives 3s
    system.AddAffector( thor::ColorAffector::Create(gradient) )
    system.AddAffector( thor::FadeOutAffector::Create() );

    // Inside main loop
    system.Update(window.GetFrameTime());
    system.Draw(window);
    window.Display()

### Time

    // Create pausable clock
    thor::StopWatch stopwatch;
    stopwatch.Start();
    sf::Sleep(3.f);
    stopwatch.Stop();
    float time = stopwatch.GetElapsedTime();

### Resources

    // Create image manager and load two images
    thor::ResourceManager<sf::Image> imageMgr;
    thor::ResourcePtr<sf::Image> p = imageMgr.Acquire( thor::Resources::ImageKey::FromFile("image.jpg") );
    thor::ResourcePtr<sf::Image> q = imageMgr.Acquire( thor::Resources::ImageKey::FromSize(300, 200) );

    // Use shared smart pointers p and q to refer to the image instances
    sf::Sprite sprite1(*p);
    sf::Sprite sprite2(*p);