# GlyphTileMap
## Introduction

Roguelike games oftentimes present the game world to the player as a grid of characters
with coloured backgrounds. This Drawable/Transformable class uses SFML's Font, Glyph, and
VertexArray to manage such a grid in a performance-friendly way.

![Example usage of GlyphTileMap showing a basic roguelike game UI](https://raw.githubusercontent.com/jpadkins/SFML_GlyphTileMap/master/res/images/example1.png)

(The above image uses 3 GlyphTileMaps: a 1x1 map for the cursor, a large map with 32x32
spacing for the game world, and a less wide but just as tall map with 16x32 spacing for
the information pane.)

Note that GlyphTileMap is meant to be a thin render layer, and does not maintain a container
of the GlyphTileMap::Tile(s) that it displays. Hence the lack of a `getTile(...)` method, etc.

[Hope you enjoy! Comments and criticism are welcome.](#)


[Source Repo](https://github.com/jpadkins/SFML_GlyphTileMap)

## Header
```cpp
///////////////////////////////////////////////////////////////////////////////
/// Author:         Jacob P Adkins (github.com/jpadkins)
/// Filename:       GlyphTileMap.h
/// License:        MIT
/// Description:    An SFML-based drawable/transformable class for displaying
///                 a grid of characters with colored backgrounds and several
///                 spacing options.
///////////////////////////////////////////////////////////////////////////////

#ifndef GLYPH_TILE_MAP_H
#define GLYPH_TILE_MAP_H

///////////////////////////////////////////////////////////////////////////////
/// Headers
///////////////////////////////////////////////////////////////////////////////

#include <SFML/System.hpp>
#include <SFML/Graphics.hpp>

class GlyphTileMap : public sf::Drawable, public sf::Transformable {
public:

    struct Tile {
        ///////////////////////////////////////////////////////////////////////
        /// Tile::Type is used to determine how a Tile's character should be
        /// placed within its bounds. The types are:
        ///
        /// Text:   Spacing is based on font kerning (e.g. characters like g
        ///         and y will intrude partially on the Tile below them)
        /// Exact:  Spacing is based on the Tile's offset values, which are
        ///         relative to the coords of Type::Center
        /// Floor:  The Tile's character is centered horizontally but aligned
        ///         vertically with the bottom of the tile
        /// Center: The Tile's character is centered both horizontally and
        ///         vertically
        ///////////////////////////////////////////////////////////////////////
        enum Type { Text, Exact, Floor, Center };

        ///////////////////////////////////////////////////////////////////////
        /// \brief Default Tile constructor
        ///
        /// The default values of a GlyphTileMap::Tile are:
        ///
        /// character:  L' '
        /// type:       Type::Center
        /// foreground: sf::Color::White
        /// background: sf::Color::Black
        /// offset:     {0, 0}
        ///////////////////////////////////////////////////////////////////////
        Tile();

        ///////////////////////////////////////////////////////////////////////
        /// \brief Constructor for a Tile
        ///
        /// \param character    Character of the Tile
        /// \param type         Type of the Tile (determines spacing)
        /// \param foreground   Color of the Tile's character
        /// \param background   Color to display behind the Tile's character
        /// \param offset       Manual spacing offset values of the Tile
        ///////////////////////////////////////////////////////////////////////
        Tile(wchar_t character, Type type, const sf::Color& foreground,
            const sf::Color& background, const sf::Vector2i& offset = {0, 0});

        ///////////////////////////////////////////////////////////////////////
        Type type;
        wchar_t character;
        sf::Vector2i offset;
        sf::Color foreground;
        sf::Color background;
    };

    ///////////////////////////////////////////////////////////////////////////
    /// \brief Constructor
    ///
    /// \param font             Reference to a loaded sf::Font to use for
    //                          glyph data
    /// \param area             Width and height of the GlyphTileMap in # of
    //                          tiles
    /// \param spacing          Width and height of each tile in pixels
    /// \param characterSize    Size of each glyph
    ///////////////////////////////////////////////////////////////////////////
    GlyphTileMap(sf::Font& font, const sf::Vector2u& area,
        const sf::Vector2u& spacing, sf::Uint32 characterSize);

    ///////////////////////////////////////////////////////////////////////////
    /// \brief Returns a const reference to the area of the GlyphTileMap
    ///
    /// \return a const reference to the area of the GlyphTileMap
    ///////////////////////////////////////////////////////////////////////////
    const sf::Vector2u& getArea() const;

    ///////////////////////////////////////////////////////////////////////////
    /// \brief Returns a const reference to the spacing of the GlyphTileMap
    ///
    /// \return a const reference to the spacing of the GlyphTileMap
    ///////////////////////////////////////////////////////////////////////////
    const sf::Vector2u& getSpacing() const;

    ///////////////////////////////////////////////////////////////////////////
    /// \brief Returns the character size of the GlyphTileMap
    ///
    /// \return the character size of the GlyphTileMap
    ///////////////////////////////////////////////////////////////////////////
    sf::Uint32 getCharacterSize() const;

    ///////////////////////////////////////////////////////////////////////////
    /// \brief Updates the GlyphTileMap at a coords with data from a Tile
    ///
    /// \param coords   Coordinates in the GlyphTileMap to update
    /// \param tile     GlyphTileMap::Tile to update the GlyphTileMap with
    ///////////////////////////////////////////////////////////////////////////
    void setTile(const sf::Vector2u& coords, const Tile& tile);

    ///////////////////////////////////////////////////////////////////////////
    /// \brief Updates the character at a coords in the GlyphTileMap
    ///
    /// \param coords       Coordinates in the GlyphTileMap to update
    /// \param character    New character for the tile
    /// \param type         Tile::Type of the new character (default Center)
    /// \param offset       Exact spacing offset value of the new character
    ///                     (default {0, 0})
    ///////////////////////////////////////////////////////////////////////////
    void setTileCharacter(const sf::Vector2u& coords, wchar_t character,
        Tile::Type type = Tile::Center, const sf::Vector2i& offset = {0, 0});

    ///////////////////////////////////////////////////////////////////////////
    /// \brief Updates the foreground color at a coords in the GlyphTileMap
    ///
    /// \param coords   Coordinates in the GlyphTileMap to update
    /// \param color    New foreground color for the tile
    ///////////////////////////////////////////////////////////////////////////
    void setTileForeground(const sf::Vector2u& coords, const sf::Color& color);

    ///////////////////////////////////////////////////////////////////////////
    /// \brief Updates the background color at a coords in the GlyphTileMap
    ///
    /// \param coords   Coordinates in the GlyphTileMap to update
    /// \param color    New background color for the tile
    ///////////////////////////////////////////////////////////////////////////
    void setTileBackground(const sf::Vector2u& coords, const sf::Color& color);

private:

    ///////////////////////////////////////////////////////////////////////////
    ///
    ///////////////////////////////////////////////////////////////////////////
    virtual void draw(sf::RenderTarget& target, sf::RenderStates states) const;

    ///////////////////////////////////////////////////////////////////////////
    ///
    ///////////////////////////////////////////////////////////////////////////
    sf::Uint32 getIndex(const sf::Vector2u& coords) const;

    ///////////////////////////////////////////////////////////////////////////
    ///
    ///////////////////////////////////////////////////////////////////////////
    void checkCharacter(wchar_t character);

    ///////////////////////////////////////////////////////////////////////////
    ///
    ///////////////////////////////////////////////////////////////////////////
    void updateTile(const sf::Vector2u& coords, const Tile& tile);

    ///////////////////////////////////////////////////////////////////////////
    ///
    ///////////////////////////////////////////////////////////////////////////
    void updateCharacter(const sf::Vector2u& coords, wchar_t character,
        Tile::Type type, const sf::Vector2i& offset);

    ///////////////////////////////////////////////////////////////////////////
    ///
    ///////////////////////////////////////////////////////////////////////////
    void updateForegroundPosition(const sf::Vector2u& coords,
        const sf::IntRect& textureRect, const sf::Vector2i& offset);

    ///////////////////////////////////////////////////////////////////////////
    ///
    ///////////////////////////////////////////////////////////////////////////
    void updateForegroundColor(const sf::Vector2u& coords,
        const sf::Color& color);

    ///////////////////////////////////////////////////////////////////////////
    ///
    /////////////////////////////////////////////////////////////////////////// 
    void updateBackgroundPosition(const sf::Vector2u& coords);

    ///////////////////////////////////////////////////////////////////////////
    ///
    ///////////////////////////////////////////////////////////////////////////
    void updateBackgroundColor(const sf::Vector2u& coords,
        const sf::Color& color);

    ///////////////////////////////////////////////////////////////////////////
    sf::Font& m_font;
    sf::Vector2u m_area;
    sf::Vector2u m_spacing;
    sf::Uint32 m_characterSize;
    sf::VertexArray m_foreground;
    sf::VertexArray m_background;
};

#endif
```

## Source
```cpp
///////////////////////////////////////////////////////////////////////////////
/// Author:         Jacob P Adkins (github.com/jpadkins)
/// Filename:       GlyphTileMap.cpp
/// License:        MIT
/// Description:    An SFML-based drawable/transformable class for displaying
///                 a grid of characters with colored backgrounds and several
///                 spacing options.
///////////////////////////////////////////////////////////////////////////////

///////////////////////////////////////////////////////////////////////////////
/// Headers
///////////////////////////////////////////////////////////////////////////////

#include "GlyphTileMap.h"

///////////////////////////////////////////////////////////////////////////////
GlyphTileMap::Tile::Tile()
    : type(Type::Center)
    , character(L'?')
    , offset(0, 0)
    , foreground(sf::Color::White)
    , background(sf::Color::Black)
{}

///////////////////////////////////////////////////////////////////////////////
GlyphTileMap::Tile::Tile(wchar_t character, Type type,
    const sf::Color& foreground, const sf::Color& background,
    const sf::Vector2i& offset)
    : type(type)
    , character(character)
    , offset(offset)
    , foreground(foreground)
    , background(background)
{}

///////////////////////////////////////////////////////////////////////////////
GlyphTileMap::GlyphTileMap(sf::Font& font, const sf::Vector2u& area,
    const sf::Vector2u& spacing, sf::Uint32 characterSize)
    : m_font(font)
    , m_area(area)
    , m_spacing(spacing)
    , m_characterSize(characterSize)
    , m_foreground(sf::Quads, area.x * area.y * 4)
    , m_background(sf::Quads, area.x * area.y * 4)
{}

///////////////////////////////////////////////////////////////////////////////
const sf::Vector2u& GlyphTileMap::getArea() const
{
    return m_area;
}

///////////////////////////////////////////////////////////////////////////////
const sf::Vector2u& GlyphTileMap::getSpacing() const
{
    return m_spacing;
}

///////////////////////////////////////////////////////////////////////////////
sf::Uint32 GlyphTileMap::getCharacterSize() const
{
    return m_characterSize;
}

///////////////////////////////////////////////////////////////////////////////
void GlyphTileMap::setTile(const sf::Vector2u& coords, const Tile& tile)
{
    updateTile(coords, tile);
}

///////////////////////////////////////////////////////////////////////////////
void GlyphTileMap::setTileCharacter(const sf::Vector2u& coords,
    wchar_t character, Tile::Type type, const sf::Vector2i& offset)
{
    updateCharacter(coords, character, type, offset);
}

///////////////////////////////////////////////////////////////////////////////
void GlyphTileMap::setTileForeground(const sf::Vector2u& coords,
    const sf::Color& color)
{
    updateForegroundColor(coords, color);
}

///////////////////////////////////////////////////////////////////////////////
void GlyphTileMap::setTileBackground(const sf::Vector2u& coords,
    const sf::Color& color)
{
    updateBackgroundColor(coords, color);
}

///////////////////////////////////////////////////////////////////////////////
void GlyphTileMap::draw(sf::RenderTarget& target, sf::RenderStates states)
    const
{
    states.transform *= getTransform();
    states.texture = &m_font.getTexture(m_characterSize);
    target.draw(m_background, states);
    target.draw(m_foreground, states);
}

///////////////////////////////////////////////////////////////////////////////
sf::Uint32 GlyphTileMap::getIndex(const sf::Vector2u& coords) const
{
    return (coords.y * m_area.x) + coords.x;
}

///////////////////////////////////////////////////////////////////////////////
void GlyphTileMap::updateTile(const sf::Vector2u& coords, const Tile& tile)
{
    const sf::Glyph& glyph = m_font.getGlyph(tile.character, m_characterSize,
        false);

    sf::Vector2i adjustedOffset = {0, 0};

    switch (tile.type) {
    case Tile::Text:
        {
        adjustedOffset.x = glyph.bounds.left;
        adjustedOffset.y = m_spacing.y + glyph.bounds.top;
        }
        break;
    case Tile::Exact:
        adjustedOffset.x = (m_spacing.x - glyph.textureRect.width) / 2;
        adjustedOffset.y = (m_spacing.y - glyph.textureRect.height) / 2;
        adjustedOffset.x += tile.offset.x;
        adjustedOffset.y += tile.offset.y;
        break;
    case Tile::Floor:
        adjustedOffset.x = (m_spacing.x - glyph.textureRect.width) / 2;
        adjustedOffset.y = m_spacing.y - glyph.textureRect.height;
        break;
    case Tile::Center:
        adjustedOffset.x = (m_spacing.x - glyph.textureRect.width) / 2;
        adjustedOffset.y = (m_spacing.y - glyph.textureRect.height) / 2;
        break;
    }

    updateForegroundPosition(coords, glyph.textureRect, adjustedOffset);
    updateForegroundColor(coords, tile.foreground);
    updateBackgroundPosition(coords);
    updateBackgroundColor(coords, tile.background);
}

///////////////////////////////////////////////////////////////////////////////
void GlyphTileMap::updateCharacter(const sf::Vector2u& coords,
        wchar_t character, Tile::Type type, const sf::Vector2i& offset)
{
    const sf::Glyph& glyph = m_font.getGlyph(character, m_characterSize,
        false);

    sf::Vector2i adjustedOffset = {0, 0};

    switch (type) {
    case Tile::Text:
        {
        adjustedOffset.x = glyph.bounds.left;
        adjustedOffset.y = m_spacing.y + glyph.bounds.top;
        }
        break;
    case Tile::Exact:
        adjustedOffset.x = (m_spacing.x - glyph.textureRect.width) / 2;
        adjustedOffset.y = (m_spacing.y - glyph.textureRect.height) / 2;
        adjustedOffset.x += offset.x;
        adjustedOffset.y += offset.y;
        break;
    case Tile::Floor:
        adjustedOffset.x = (m_spacing.x - glyph.textureRect.width) / 2;
        adjustedOffset.y = m_spacing.y - glyph.textureRect.height;
        break;
    case Tile::Center:
        adjustedOffset.x = (m_spacing.x - glyph.textureRect.width) / 2;
        adjustedOffset.y = (m_spacing.y - glyph.textureRect.height) / 2;
        break;
    }

    updateForegroundPosition(coords, glyph.textureRect, adjustedOffset);
}

///////////////////////////////////////////////////////////////////////////////
void GlyphTileMap::updateForegroundPosition(const sf::Vector2u& coords,
    const sf::IntRect& textureRect, const sf::Vector2i& offset)
{
    sf::Uint32 index = getIndex(coords) * 4;

    m_foreground[index].position = {
        static_cast<float>((coords.x * m_spacing.x) + offset.x),
        static_cast<float>((coords.y * m_spacing.y) + offset.y)
        };
    m_foreground[index + 1].position = {
        static_cast<float>((coords.x * m_spacing.x) + textureRect.width
            + offset.x),
        static_cast<float>((coords.y * m_spacing.y) + offset.y)
        };
    m_foreground[index + 2].position = {
        static_cast<float>((coords.x * m_spacing.x) + textureRect.width
            + offset.x),
        static_cast<float>((coords.y * m_spacing.y) + textureRect.height
            + offset.y)
        };
    m_foreground[index + 3].position = {
        static_cast<float>((coords.x * m_spacing.x) + offset.x),
        static_cast<float>((coords.y * m_spacing.y) + textureRect.height
            + offset.y)
        };
    m_foreground[index].texCoords = {
        static_cast<float>(textureRect.left),
        static_cast<float>(textureRect.top)
        };
    m_foreground[index + 1].texCoords = {
        static_cast<float>(textureRect.left + textureRect.width),
        static_cast<float>(textureRect.top)
        };
    m_foreground[index + 2].texCoords = {
        static_cast<float>(textureRect.left + textureRect.width),
        static_cast<float>(textureRect.top + textureRect.height)
        };
    m_foreground[index + 3].texCoords = {
        static_cast<float>(textureRect.left),
        static_cast<float>(textureRect.top + textureRect.height)
        };
}

///////////////////////////////////////////////////////////////////////////////
void GlyphTileMap::updateForegroundColor(const sf::Vector2u& coords,
    const sf::Color& color)
{
    sf::Uint32 index = getIndex(coords) * 4;

    m_foreground[index].color = color;
    m_foreground[index + 1].color = color;
    m_foreground[index + 2].color = color;
    m_foreground[index + 3].color = color;
}

///////////////////////////////////////////////////////////////////////////////
void GlyphTileMap::updateBackgroundPosition(const sf::Vector2u& coords)
{
    sf::Uint32 index = getIndex(coords) * 4;

    m_background[index].position = {
        static_cast<float>(coords.x * m_spacing.x),
        static_cast<float>(coords.y * m_spacing.y)
        };
    m_background[index + 1].position = {
        static_cast<float>((coords.x * m_spacing.x) + m_spacing.x),
        static_cast<float>(coords.y * m_spacing.y)
        };
    m_background[index + 2].position = {
        static_cast<float>((coords.x * m_spacing.x) + m_spacing.x),
        static_cast<float>((coords.y * m_spacing.y) + m_spacing.y)
        };
    m_background[index + 3].position = {
        static_cast<float>(coords.x * m_spacing.x),
        static_cast<float>((coords.y * m_spacing.y) + m_spacing.y)
        };
}

///////////////////////////////////////////////////////////////////////////////
void GlyphTileMap::updateBackgroundColor(const sf::Vector2u& coords,
    const sf::Color& color)
{
    sf::Uint32 index = getIndex(coords) * 4;

    m_background[index].color = color;
    m_background[index + 1].color = color;
    m_background[index + 2].color = color;
    m_background[index + 3].color = color;
}
```

## Example Usage
```cpp
...
sf::Font font;
font.loadFromFile("font.ttf");

///////////////////////////////////////////////////////////////////////////////
/// Create a GlyphTileMap with the following arguments:
///
/// font        The loaded sf::Font to use for sf::Glyph data
/// {40, 30}    sf::Vector2u containing the dimensions of the map (in tiles)
/// {16, 16}    sf::Vector2u containing the dimensions of each tile (in pixels)
/// 16          sf::Uint32 character size of the sf::Glyphs for each tile
///////////////////////////////////////////////////////////////////////////////
GlyphTileMap tileMap(font, {40, 30}, {16, 16}, 16);

///////////////////////////////////////////////////////////////////////////////
/// Create a GlyphTileMap::Tile, a struct containing all the data required to
/// populate a tile of a GlyphTileMap.
///
/// L'╬'                        wchar_t of the character to display
/// GlyphTileMap::Tile::Exact   Spacing of the character within the tile,
///                             either Text, Exact, Floor, or Center
/// sf::Color::White            Color of the tile's character
/// sf::Color::Black            Color of the tile's background
/// {3, -4}                     Optional exact spacing of the tile's character
///                             (relative to the center)
///////////////////////////////////////////////////////////////////////////////
GlyphTileMap::Tile tile(
    L'╬',
    GlyphTileMap::Tile::Exact,
    sf::Color::White,
    sf::Color::Black,
    {3, -4}
    );

///////////////////////////////////////////////////////////////////////////////
/// Update a tile on the GlyphTileMap to values from the GlyphTileMap::Tile
///////////////////////////////////////////////////////////////////////////////
tileMap.setTile({0, 0}, tile);

///////////////////////////////////////////////////////////////////////////////
/// Tiles on the GlyphTileMap can be updated without a GlyphTileMap::Tile
///////////////////////////////////////////////////////////////////////////////
tileMap.setBackground({0, 1}, sf::Color::Blue);
tileMap.setCharacter({1, 0}, L'!', GlyphTileMap::Tile::Floor);

...

///////////////////////////////////////////////////////////////////////////////
/// Draw/Transform the GlyphTileMap like any other SFML drawable/transformable
///////////////////////////////////////////////////////////////////////////////
tileMap.setScale(0.5f, 0.5f);
tileMap.setPosition(100, 100);

window.clear();
window.draw(tileMap);
window.display();
...
```