The `CurvedText` class offers the ability to text arranged in a circular arc. The interface is similar to SFML's native Text class, but the constructor takes two additional parameters:

  * `radius`: The radius of the circle this text is arranged upon
  * `spacing`: Extra character spacing, in pixels

You can also set the text to be "center justified" (put the origin at the middle of the arc), by specifying `CurvedText.Centered = true`.

## How it works

Roughly, the same way as [sf::Text](https://github.com/LaurentGomila/SFML/blob/master/src/SFML/Graphics/Text.cpp). In the `UpdateGeometry` method, a `VertexArray` of Quads is created, to be textured by the `Draw` method. The main difference is the X coordinate of the vertices is rotated around a point a distance `Radius` below the baseline of the text. This is done incrementally, saving the `Transform` for each  previous character.

```c#
            float angle = 2f * (float)Math.Atan((x - previousX) / 2f / Radius)
                * 180f / (float)Math.PI;
            angleCovered += angle;

            transform.Rotate(angle, x0 - g.Bounds.Width / 2f, Radius / 2f);

            Vector2f topLeft     = new Vector2f(x0, y0);
            Vector2f bottomLeft  = new Vector2f(x0, y1);
            Vector2f topRight    = new Vector2f(x1, y0);
            Vector2f bottomRight = new Vector2f(x1, y1);

            topLeft     = transform.TransformPoint(topLeft);
            topRight    = transform.TransformPoint(topRight);
            bottomLeft  = transform.TransformPoint(bottomLeft);
            bottomRight = transform.TransformPoint(bottomRight);
```

###Known bugs

 * `Centered` does not work as expected for multi-line text.

Full code below. Please let me know [through the forums](http://en.sfml-dev.org/forums/index.php?topic=12100.0) if you find any bugs or improve the class.
```c#
/// Copyright 2013, Camilo Polymeris. License: ZLIB/PNG (Same as SFML)
public class CurvedText : Transformable, Drawable
{
    public CurvedText(
        string str,
        float radius,
        float spacing,
        Font font,
        uint charSize) : this(str, radius, font, charSize)
    {
        Spacing = spacing;
    }

    public CurvedText(
        string str,
        float radius,
        Font font = null,
        uint charSize = 30)
    {
        vertices = new VertexArray(PrimitiveType.Quads);
        DisplayedString = str;
        CharacterSize = charSize;
        Radius = radius;
        Spacing = 0;
        Font = font;
    }

    public void Draw(RenderTarget target, RenderStates states)
    {
        if (Font == null)
            return;
        states.Transform *= Transform;
        if (Centered)
            states.Transform *= centerTransform;
        states.Texture = Font.GetTexture(CharacterSize);
        target.Draw(vertices, states);
    }

    protected void UpdateGeometry()
    {
        if (font == null)
            return;
        vertices.Clear();
        float hspace = Font.GetGlyph(' ', CharacterSize, true).Advance;
        float vspace = Font.GetLineSpacing(CharacterSize);
        float previousX = 0;
        float x = 0;
        float y = CharacterSize;
        char previousChar = '\0';
        float angleCovered = 0;

        Transform transform = Transform.Identity;

        foreach (char c in DisplayedString)
        {
            x += Font.GetKerning(previousChar, c, CharacterSize) + Spacing;
            previousChar = c;

            switch (c)
            {
                case ' ':
                    x += hspace;
                    continue;
                case '\t':
                    x += 4 * hspace;
                    continue;
                case '\n':
                    y += vspace;
                    previousX = 0;
                    angleCovered = 0;
                    transform = Transform.Identity;
                    x = 0;
                    continue;
                case '\v':
                    y += vspace * 4;
                    continue;
            }

            Glyph g = Font.GetGlyph(c, CharacterSize, false);
            float x0 = x  + g.Bounds.Left;
            float y0 = y  + g.Bounds.Top;
            float x1 = x0 + g.Bounds.Width;
            float y1 = y0 + g.Bounds.Height;
            float u0 =      g.TextureRect.Left;
            float v0 =      g.TextureRect.Top;
            float u1 = u0 + g.TextureRect.Width;
            float v1 = v0 + g.TextureRect.Height;

            float angle = 2f * (float)Math.Atan((x - previousX) / 2f / Radius)
                * 180f / (float)Math.PI;
            angleCovered += angle;

            transform.Rotate(angle, x0 - g.Bounds.Width / 2f, Radius / 2f);

            Vector2f topLeft     = new Vector2f(x0, y0);
            Vector2f bottomLeft  = new Vector2f(x0, y1);
            Vector2f topRight    = new Vector2f(x1, y0);
            Vector2f bottomRight = new Vector2f(x1, y1);

            topLeft     = transform.TransformPoint(topLeft);
            topRight    = transform.TransformPoint(topRight);
            bottomLeft  = transform.TransformPoint(bottomLeft);
            bottomRight = transform.TransformPoint(bottomRight);

            vertices.Append(new Vertex(topLeft,     Color, new Vector2f(u0, v0)));
            vertices.Append(new Vertex(topRight,    Color, new Vector2f(u1, v0)));
            vertices.Append(new Vertex(bottomRight, Color, new Vector2f(u1, v1)));
            vertices.Append(new Vertex(bottomLeft,  Color, new Vector2f(u0, v1)));

            previousX = x;
            x+= g.Advance;
        }

        centerTransform = Transform.Identity;
        centerTransform.Rotate(-angleCovered / 2f, 0, Radius);
        centerTransform.Translate(0, -Radius / 2f);

    }

    public float Radius
    {
        get { return radius; }
        set
        {
            radius = value;
            UpdateGeometry();
        }
    }

    public string DisplayedString
    {
        get { return displayedString; }
        set
        {
            displayedString = value;
            UpdateGeometry();
        }
    }

    public Font Font
    {
        get { return font; }
        set
        {
            font = value;
            UpdateGeometry();
        }
    }

    public uint CharacterSize
    {
        get { return characterSize; }
        set
        {
            characterSize = value;
            UpdateGeometry();
        }
    }

    public Color Color
    {
        get { return color; }
        set
        {
            color = value;
            for (uint i = 0; i < vertices.VertexCount; i++)
            {
                Vertex v = vertices[i];
                v.Color = color;
                vertices[i] = v;
            }
        }
    }

    public float Spacing
    {
        get { return spacing; }
        set
        {
            spacing = value;
            UpdateGeometry();
        }
    }

    public bool Centered
    {
        get { return centered; }
        set
        {
            centered = value;
        }
    }

    VertexArray vertices;
    float radius;
    string displayedString;
    Font font;
    uint characterSize;
    Color color;
    float spacing;
    bool centered;
    Transform centerTransform;
}
```