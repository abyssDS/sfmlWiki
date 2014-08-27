# SFML Tile-based lightning system

## Sources

https://github.com/achpile/sfml-lighting

## Main idea

The main idea of the whole system is calculation lighting without dependency of light emitters. May be it's not perfect, but fast.

There's some points you have to know

1. Every tile is light emitter
2. Every wall tile emits black color with zero intensity
3. Every 'sky' tile emits 'ambient' color with 'ambient' intensity
4. There's fixed amount of intensity values

So, after those main points we can proceed.

So we can create our 'tile' class:

``` cpp
enum MapTileType {
	mtAir,
	mtWall,
	mtSolid
};



struct MapTile {
	MapTileType  type;
	sf::Color    light;
	sf::Vector2i index;
	char         intensity;
	char         absorb;
};
```

* Type is how our tile looks like
* Light is the lighting color of the current tile
* Index is tile position
* Intensity is  tile's light intensity
* Absorb is how much this tile decreases intensity

## Initialization

At first we have to reset all lighting values:

``` cpp
	sf::Vector2i from(0, 0);
	sf::Vector2i to(MAP_SIZE_X, MAP_SIZE_Y);

	// Calculating ambient color depending on it's intensity
	sf::Color    color = applyIntensity(ambientColor, ambientIntensity);

	// Reset all the counters
	for (int i = 0; i < LIGHT_MAX_LIGHTLEVEL; lightCounts[i++] = 0);

	// Each tile with bricks have black color with zero intensity
	// Each 'sky' tile have ambient color with ambient intensity
	for (int i = from.x; i < to.x; i++)
		for (int j = from.y; j < to.y; j++)
			if (tiles[i][j].type == mtAir) {
				tiles[i][j].intensity = ambientIntensity;
				tiles[i][j].light     = color;
			} else {
				tiles[i][j].intensity = 0;
				tiles[i][j].light     = sf::Color::Black;
			}
```

Next step is to apply all additional emitters:

``` cpp
	for (unsigned int i = 0; i < sources.size(); i++)
		addIntensity(sources[i]->position, sources[i]->getIntensity(), sources[i]->color);
```

``` cpp
void Map::addIntensity(sf::Vector2i index, char intensity, sf::Color color) {
	if (index.x < 0 || index.x >= MAP_SIZE_X || index.y < 0 || index.y >= MAP_SIZE_X)
		return;

	color = applyIntensity(color, intensity);
	tiles[index.x][index.y].light = mixColors(tiles[index.x][index.y].light, color);

	if (tiles[index.x][index.y].intensity < intensity)
		tiles[index.x][index.y].intensity = intensity;
}
```

Here we are checking if the emitter is inside the map, calculating color it can give, mixing it and applying to the tile the greatest intensity (it is emitter's or the intensity tile already have).

So now we have set color and intensity of the 'ambient' and of all the light emitters to the map. Next step will be a propogation of our light.

## Initialization data structures

At first we have to init all the data structures used in propogation.

``` cpp
	sf::Vector2i from(0, 0);
	sf::Vector2i to(MAP_SIZE_X, MAP_SIZE_Y);

	for (int i = from.x; i < to.x; i++)
		for (int j = from.y; j < to.y; j++)
			initIntensity(&tiles[i][j]);
```

``` cpp
void Map::initIntensity(MapTile *tile) {
	int index = tile->intensity - 1;
	if (index < 0 || index >= LIGHT_MAX_LIGHTLEVEL) return;

	lightTiles[index][lightCounts[index]++] = tile;
}
```

At this step we are adding tile pointers to special arrays. This is the only reason of using fixed amount of intensity values. So we have 'MAX_INTENSITY' lists of the tiles, which intensity is equal to the list index.

## Propogation

And now we have to propogate out light. So, let's start from the tiles with greater intensity and go to the lower.

``` cpp
	for (int i = LIGHT_MAX_LIGHTLEVEL - 1; i >= 0; i--)
		for (int j = 0; j < lightCounts[i]; j++) {
			if (lightTiles[i][j]->intensity != i + 1) continue;
			checkNeighbours(lightTiles[i][j]);
		}
```

Here we are checking all the neighbour tiles to current tile if we can give them some light

``` cpp
void Map::checkNeighbours(MapTile *tile) {
	int x = tile->index.x;
	int y = tile->index.y;

	char intensity = tile->intensity - tile->absorb;
	if (intensity < 0) return;
	sf::Color color = reapplyIntensity(tile->light, tile->intensity, intensity);

	if (x > 0             ) setIntensity(&tiles[x-1][y], intensity, color);
	if (x < MAP_SIZE_X - 1) setIntensity(&tiles[x+1][y], intensity, color);
	if (y > 0             ) setIntensity(&tiles[x][y-1], intensity, color);
	if (y < MAP_SIZE_Y - 1) setIntensity(&tiles[x][y+1], intensity, color);


	// Diagonal-related tiles should gain lesser light
	color.r *= 0.9f;
	color.g *= 0.9f;
	color.b *= 0.9f;

	if (x > 0              && y < MAP_SIZE_Y - 1) setIntensity(&tiles[x-1][y+1], intensity, color);
	if (x < MAP_SIZE_X - 1 && y > 0             ) setIntensity(&tiles[x+1][y-1], intensity, color);
	if (y > 0              && x > 0             ) setIntensity(&tiles[x-1][y-1], intensity, color);
	if (y < MAP_SIZE_Y - 1 && x < MAP_SIZE_X - 1) setIntensity(&tiles[x+1][y+1], intensity, color);
}
```

So, we're decreasing intensity by 'absorb' value, recalculating light color and trying to give it to all the neighbours.

``` cpp
void Map::setIntensity(MapTile *tile, char intensity, sf::Color color) {
	if (intensity > tile->intensity || canMixColors(tile->light, color)) {
		tile->light = mixColors(tile->light, color);

		if (intensity != tile->intensity) {
			tile->intensity = intensity;

			int index = tile->intensity - 1;

			if (index < 0) return;
			if (index >= LIGHT_MAX_LIGHTLEVEL) return;

			lightTiles[index][lightCounts[index]] = tile;
			lightCounts[index]++;
		}
	}
}
```

Each neighbour checks, if it can mix own color with given. And if the intensity is not the same - we should process this neighbour tile one more time (to avoid color loss).

After processing all the lists we have set lighting color to all the tiles.

## Rendering

Next (and final) step is rendering the colormap. And there's nothing hard.

``` cpp
void Map::renderLight() {
	sf::Vector2i from(0, 0);
	sf::Vector2i to(MAP_SIZE_X, MAP_SIZE_Y);

	for (int i = from.x - 1; i < to.x; i++)
		for (int j = from.y - 1; j < to.y; j++) {
			lightMask[0].position = getTilePos(i  , j  );
			lightMask[1].position = getTilePos(i+1, j  );
			lightMask[2].position = getTilePos(i+1, j+1);
			lightMask[3].position = getTilePos(i  , j+1);

			lightMask[0].color = getTileLight(i  , j  );
			lightMask[1].color = getTileLight(i+1, j  );
			lightMask[2].color = getTileLight(i+1, j+1);
			lightMask[3].color = getTileLight(i  , j+1);

			app->draw(lightMask, 4, sf::Quads, sf::BlendMultiply);
		}
}
```

We just rendering squares (like tiles), but the vertices of each square positioned in the center of each tile.
So each vertice have position like (tile.pos + tile.size / 2) and color equals to tile light. To draw we should use sf::BlendMultiply.

Now run it and enjoy your colorful lighting ;)