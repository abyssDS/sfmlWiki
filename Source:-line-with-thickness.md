##Notes
 * Encapsulate the code below in your own class(es)
 * This draws a polygon outline, if you want an "open" line, make `i` count to only `points.Count + 1`. The code can, of course, easily be modified to support both.
 * Sharp corners might look bad, depending on your application. One option is to replace
`d *= thickness / 2f / dot;` with e.g `d *= thickness / 2f / (float)Math.Max(.8, dot);`. This will make the line thinner in sharp angles. A better option would be to implement flat or (harder) rounded miter joints.

``` c#
VertexArray GenerateTrianglesStrip(List<Vector2f> points, Color color, float thickness)
{
	var array = new VertexArray(PrimitiveType.TrianglesStrip);
	for (int i = 1; i < points.Count + 2; i++)
	{
		Vector2f v0 = points[(i - 1) % points.Count];
		Vector2f v1 = points[i % points.Count];
		Vector2f v2 = points[(i + 1) % points.Count];
		Vector2f v01 = (v1 - v0).Normalized();
		Vector2f v12 = (v2 - v1).Normalized();
		Vector2f d = (v01 + v12).GetNormal();
		float dot = d.Dot(v01.GetNormal());
		d *= thickness / 2f / dot; //< TODO: Add flat miter joint in extreme cases
		array.Append(new Vertex(v1 + d, color));
		array.Append(new Vertex(v1 - d, color));
	}
	return array;
}
```

##SFML Extensions

The code above uses the following extensions to Vector2f:
``` cs
public static float GetMagnitude(this Vector2f v)
{
	return (float)Math.Sqrt(v.X * v.X + v.Y * v.Y);
}

public static float Dot(this Vector2f v0, Vector2f v1)
{
	return v0.X * v1.X + v0.Y * v1.Y;
}

public static Vector2f GetNormal(this Vector2f v)
{
	return new Vector2f(-v.Y, v.X);
}

public static Vector2f Normalized(this Vector2f v)
{
	return v / v.GetMagnitude();
}
```

<sub>Copyright 2013, Camilo Polymeris. License: [ZLIB/PNG](http://www.sfml-dev.org/license.php) (Same as SFML)</sub>