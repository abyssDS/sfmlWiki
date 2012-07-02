[C++] Zoost & Zoom: An easy way to create and handle geometric objets, animate and use them for better graphics 


# Zoost & Zoom

Zoost & Zoom are two libraries that allow you to animate, intersect, test, enlight and render the geometric objects you create in your code. They are as simple to use than the SFML2 and compatible with its last revision.
For example, you can create a Geom , give it the desired shape, and use it to create a drawable Shape with custom properties (colors, segments thickness, etc...).
Of course, you can create your own lib that use the Geom objects in the way you want (like a Physic engine) !

![Zin](http://uppix.net/1/d/f/de6652b31c9e7133cdd84c8aacd6d.png)

###Quick guide###

1. [How to compile and install the libs ?](#howto)
2. [Create a Geom and handle it](#geom)
3. [Work with the Curves](#curve)
4. [The best way to animate your Geom](#animation)
5. [Using a Shape to represent your Geom](#shape)
6. [Let's play with the light system](#light)


## <a name="howto" />How to compile and install the libs ? [ [Top] ](#top)

The best and easy way to compile the libs is to generate a makefile with CMake (there are many tutorials for that on the Internet) and to launch the build with the appropriate make command. Note that the SFML 2.0 rc is required to compile and use the libs, so the SFML must be present on your system and the variable SFMLDIR defined with the SFML's installation path.
Finally, use the install command to install the libs in the desired location.

## <a name="geom" />Create a Geom and handle it[ [Top] ](#top)

The following sample shows you how to proceed :

```cpp
#include <Zoost/Geom.hpp>

// Create an empty Geom
Geom geom; 

// Add some vertices to the Geom
// Note that you can't create a Vertex instance directly, but you can store its reference (returned by the addVertex method) for a later use
Vertex& v1 = geom.addVertex({25, 25});
Vertex& v2 = geom.addVertex({80, 10});

// Apply some transformation to the Geom
geom.rotate(toRads(45));
geom.scale({2, 2});
geom.move({20, 30});

// Convert local coords into global coords
Coords coords = geom.convertToGlobal(v1.getCoords());

// Add a liaison (segment) between two vertices
geom.addLiaison(v1, v2);

// Note that you can get the same result with
geom.addLiaison(geom.getVertex(0), geom.getVertex(1));

// or if the vertices weren't defined before
geom.addLiaison(geom.addVertex({25, 25}), geom.addVertex({80, 10}));

// Here we just have created a segment and we would to make an intersection test with another geom. Just do like that
bool isIntersecting = myCustomGeom.instersects(geom);

// Or if you want to know the intersection points, use the overloaded function :
std::vector<Geom::Intersection> intersections;
geom.instersects(geom, intersections);

// You can add one or more faces to you Geom like this
Vertex& v3 = addVertex({60, 18});
geom.addFace(v1, v2, v3);

// Faces are used to test if the Geom contains a point :
bool isContaining = geom.contains({20, 20});

// And you can get all the faces (if they exist) who are containing the point like that :
std::vector<Face*> faces;
bool isContaining = geom.contains({20, 20}, faces);

for( auto& facePtr : faces )
{
    Face& face = *facePtr;
    ...
}

// Of course, geoms are addable ans assignable
Geom geom = geom1 + geom2;
geom+=geom3;

Geom geom4 = geom3;

// Finaly you can create pre made Geom easilly :
Geom star = Geom::star(); // Create a star Shape

// Here is the complete list of the method you can call to get all the pre made Geom :
 Geom::segment(const Point& point1, const Point& point2);
 Geom::triangle(const Point& point1, const Point& point2, const Point& point3);
 Geom::quad(const Point& point1, const Point& point2, const Point& point3, const Point& point4);
 Geom::rectangle(const Point& size);
 Geom::rectangle(const Rect& rect);
 Geom::rectangle(const Point& point1, const Point& point2, unsigned int width);
 Geom::scare(double length);
 Geom::circle(double radius = 20);
 Geom::star(double width1 = 30, double width2 = 60, unsigned int complexity = 5);
 Geom::polygon(double radius = 20, unsigned int complexity = 5); -> Regular polygon
 Geom::polygon(const std::initializer_list<Point>& points); -> Complex polygon (convex & concav)

```

## <a name="curve" />Work with the curves [ [Top] ](#top)

A curve is like a Geom (Curve is inheriting from Geom) but lines and faces cannot be manually added.
You just have to add a vertex and the liaison is automatically added. Plus, you can get the length of the Curve and get the coords at the [0, 1] location by interpolation using the [] operator. If you want to form a broken line, you can create it by addition of several parts of curve.

```cpp
#include <Zoost/Curve.hpp>

Curve curve;
curve.addVertex(...);

Curve curve2({{20, 20}, {31, 43}, {5, -10}});

Vector2d middle = curve2[.5f];

Curve curve3 = curve1 + curve2;
double length = curve3.getLength();

Curve bezier = Curve::bezier({{767, 410}, {1000, 900}, {1000, 30}, {500, 30}, {1, 30}, {50, 560}, {169, 410}}); // This coords are the control points of the Bezier curve

```

## <a name="animation" />The best way to animate your Geom [ [Top] ](#top)

Two classes are available to help you to animate your objects :

###Variation###

Variation is the simplier of both. It just applies a linear interpolation to a value over time. It can be repeated and/or reversed.
See the example :

```cpp
#include <Zoost/Variation.hpp>

...

Variation<double> variation(0, toRads(45), toRads(185));
sf::Clock clock;

while( ... )
{
    sf::Time timeStep = clock.restart();
    double angle = variation.update(timeStep);
    myCustomGeom.setRotation(angle);
}

```

###Kinetic###

With Kinetic you can track a point running on a curve over time :

```cpp
#include <Zoost/Kinetic.hpp>

...

Kinetic kinetic(curve); // You can use a Bezier Curve here; so fun !
sf::Clock clock;

while( ... )
{
    sf::Time timeStep = clock.restart();
    Coords coords = kinetic.update(timeStep);
    myCustomGeom.setPosition(coords);
}

```

## <a name="shape" />Using a Shape to represent your Geom [ [Top] ](#top)

```cpp
#include <Zoom/Shape.hpp>

...

Shape shape(geom); // You must create a shape using a Geom

// You can custom your shape in the way you want :
shape.setVerticesColor(Color::getFromWebCode(#B9121B));
shape.setLiaisonsColor(Color::DarkSlateBlue);
shape.setFacesColor(sf::Color::White);
shape.setVerticesSize(2)
setLiaisonsWidth(1);
shape.showVertices(true);
shape.showFaces(true);
shape.enableFacesColor();

// And for more personalization you can do the same calls for a specific item (Vertex/Liaison/Face) :
shape.setVertexColor(0, sf::Color::Green);
...

app.draw(shape); // Shape is inheriting from sf::Drawable, so you can draw it directly in a sf::RenderTarget

```

## <a name="light" />Let's play with the light system [ [Top] ](#top)

The light engine is composed of a Light manager and the light/spot classes.
Here is an example of use :

```cpp

#include <Zoom/LightManager.hpp>

Geom walls = Geom::segment({100, 100}, {125, 125})
           + Geom::segment({300, 200}, {450, 325})
           + Geom::segment({250, 245}, {400, 365})
           + Geom::segment({500, 200}, {600, 300})
           + Geom::segment({125, 430}, {125, 475})
           + Geom::segment({500, 300}, {600, 200})
           + Geom::segment({125, 475}, {175, 475});

LightManager lightManager;

Light light(150, Color::Green);
light.setPosition({400, 0});

Spot spot(400, toRads(45), Color(190, 150, 150));
spot.setPosition({700, 400});

lightManager.attach(light);
lightManager.attach(spot);
lightManager.attach(walls);

while( ... )
{
    lightManager.update();

    ...

    app.draw(lightManager);
}

```

Unfortunatly there is a bug when a light is rotated (It will be corrected in the future) !

To be continued...