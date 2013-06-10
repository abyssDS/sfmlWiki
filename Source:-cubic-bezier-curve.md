This is an example implementation to calculate the points on a [[Cubic Bézier Curve|http://en.wikipedia.org/wiki/B%C3%A9zier_curve]], losely based on [[this AS3 example|http://www.paultondeur.com/2008/03/09/drawing-a-cubic-bezier-curve-using-actionscript-3/]].

###Parameters
**start, end:**
These vectors define the points between which the bézier curve is to be calculated.

**startControl, endControl:**
These vectors define the control points determining the slopes at the end points.

**numSegments:**
The number of segments to be drawn. This has to be a number bigger than or equal to `1` (where `1` will result in a straight line).

## Code
```
std::list<sf::Vector2f> CalcCubicBezier(const sf::Vector2f &start, const sf::Vector2f &end, const sf::Vector2f &startControl, const sf::Vector2f &endControl, const size_t numSegments) {
    std::list<sf::Vector2f> ret;
    if (!numSegments) // Any points at all?
        return ret;

    ret.push_back(start); // First point is fixed
    float p = 1.f / numSegments;
    float q = p;
    for (size_t i = 1; i < numSegments; i++, p += q) // Generate all between
        ret.push_back(p * p * p * (end + 3.f * (startControl - endControl) - start) + 3.f * p * p * (start - 2.f * startControl + endControl) + 3.f * p * (startControl - start) + start);
    ret.push_back(end); // Last point is fixed
    return ret;
}
```
###Notes
This function won't draw the resulting bezier. It will only calculate the points (which might be used for drawing or something different, e.g. to be used for color correction).

The easiest way to draw such a curve would be adding the points to a `sf::VertexArray`:

```
// Create a vertex array for drawing; a line strip is perfect for this
sf::VertexArray vertices(sf::LinesStrip, 0);

// Calculate the points on the curve (10 segments)
std::list<sf::Vector2f> points = CalcCubicBezier(sf::Vector2f(0, 100), sf::Vector2f(300, 200), sf::Vector2f(0, 150), sf::Vector2f(300, 150), 10);

// Append the points as vertices to the vertex array
for (std::list<sf::Vector2f>::const_iterator a = points.begin(); a != points.end(); ++a)
    vertices.append(sf::Vertex(*a, sf::Color::White));

// ...

// Draw the vertex array
window.draw(vertices);
```