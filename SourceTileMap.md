## Basic Tiled Map using SFML's VertexArrays

This Tile Map example consists of two classes and one struct.  
StaticTiledMap - class that holds chunksize and tilesize constants and will load vertices, store, display and texture them.  
TileLoader - abstract base class, StaticTiledMap loads tiles by calling virtual method of class derieved from TileLoader that you pass to it.  
MapData - struct that holds measurements of map(in tiles) and the name of texture that is to be used by StaticTiledMap, it's placed in TileLoader.h because MapData makes sense only when recieved/used by StaticTiledMap/TileLoader so putting it in it's own header would be redundant.  

All the code is missing `#include`s of sfml, vector, string, you must include them or precompiled header with them yourself.   
TileLoader.h:
```cpp
	struct MapData
	{
		std::string TextureName;
		int MapX,MapY;
		MapData(void):MapX(0),MapY(0){}
	};
	class TileLoader//this is the class you must derieve from and complete following 2 tasks :
	{
	protected:
		MapData m_mapdata;//1.fill this struct with name of texture and map dimensions(in tiles)
	public:
		virtual ~TileLoader(void){};
		virtual void AppendTile(int gx,int gy,sf::VertexArray& garr)=0;
		/*
		2.implement this method to append(or skip appending) tiles(4 vertices) to given vertex array,
		you may assume tiles will be requested in row major order:((0,0),(1,0),..,(map_x-1,0),(0,1),..,(map_x-1,map_y-1)),
		this method will be called exactly MapX*MapY times,
		for correct displaying and culling it's assumed your appended tile will be square with side equal to tilesize
		and have it's top left point at (tilesize*gx,tilesize*gy),
		*/
		const MapData& GetData(void)const{return m_mapdata;}
	};
```
StaticTiledMap.h:   
```cpp
	class TileLoader;//forward declaration for pointer
	class StaticTiledMap : public sf::Drawable
	{
	private:
		enum {tilesize=32,chunksize=32};//change values of these to match your needs and improve performance
		virtual void draw(sf::RenderTarget& target,sf::RenderStates states)const;
		sf::Texture m_texture;
		int map_x,map_y,chunks_x,chunks_y;//map x and y - dimensions of map in tiles, chunks x and y - amount of chunks
		std::vector<std::vector<sf::VertexArray> > m_chunks;
	public:
		StaticTiledMap(void);
		virtual ~StaticTiledMap(void){};
		void LoadFrom(TileLoader* gloader);
	};
```
StaticTiledMap.cpp:
```cpp
	#include "StaticTiledMap.h"
	#include "TileLoader.h"
	void StaticTiledMap::draw(sf::RenderTarget& target,sf::RenderStates states)const
	{

		int left=0,right=0,top=0,bottom=0;
		//get chunk indices into which top left and bottom right points of view fall:
		sf::Vector2f temp=target.getView().getCenter()-(target.getView().getSize()/2.f);//get top left point of view
		left=static_cast<int>(temp.x/(chunksize*tilesize));
		top=static_cast<int>(temp.y/(chunksize*tilesize));
		temp+=target.getView().getSize();//get bottom right point of view
		right=1+static_cast<int>(temp.x/(chunksize*tilesize));
		bottom=1+static_cast<int>(temp.y/(chunksize*tilesize));
		//clamp these to fit into array bounds:
		left=std::max(0,std::min(left,chunks_x));
		top=std::max(0,std::min(top,chunks_y));
		right=std::max(0,std::min(right,chunks_x));
		bottom=std::max(0,std::min(bottom,chunks_y));
		//set texture and draw visible chunks:
		states.texture=&m_texture;
		for(int ix=left;ix<right;++ix)
		{
			for(int iy=top;iy<bottom;++iy)
			{
				target.draw(m_chunks[ix][iy],states);
			}
		}
	}
	StaticTiledMap::StaticTiledMap(void):
	map_x(0),map_y(0),chunks_x(0),chunks_y(0)
	{

	}
	void StaticTiledMap::LoadFrom(TileLoader* gloader)
	{
		m_texture.loadFromFile(gloader->GetData().TextureName);
		map_x=gloader->GetData().MapX;
		map_y=gloader->GetData().MapY;
		if((map_x*map_y)==0)//empty map - possibly forgotten to fill data struct
		{
			//to stop displaying at all after failed loading:
			chunks_x=0;
			chunks_y=0;
			m_chunks.clear();
			return;
		}
		chunks_x=(map_x/chunksize)+1;
		chunks_y=(map_y/chunksize)+1;
		m_chunks.assign(chunks_x,std::vector<sf::VertexArray>(chunks_y,sf::VertexArray(sf::Quads)));//ready up empty 2d arrays
		for(int iy=0;iy<map_y;++iy)
		{
			for(int ix=0;ix<map_x;++ix)
			{
				gloader->AppendTile(ix,iy,m_chunks[ix/chunksize][iy/chunksize]);
			}
		}
	}
```
Example implementation of TileLoader:  
```cpp
	#include "TileLoader.h"
	#include "StaticTiledMap.h"
	class ExampleLoader : public TileLoader
	{
	public:
		ExampleLoader(void)
		{
			m_mapdata.MapX=100;
			m_mapdata.MapY=100;
			m_mapdata.TextureName="JustBrick.tga";//a simple 32x32 seamless image of brick 
		}
		virtual void AppendTile(int gx,int gy,sf::VertexArray& garr)
		{
			sf::Vertex ver;
			ver.position=sf::Vector2f(gx*32.f,gy*32.f);
			ver.texCoords=sf::Vector2f(0.f,0.f);
			garr.append(ver);

			ver.position=sf::Vector2f(gx*32.f+32.f,gy*32.f);
			ver.texCoords=sf::Vector2f(32.f,0.f);
			garr.append(ver);

			ver.position=sf::Vector2f(gx*32.f+32.f,gy*32.f+32.f);
			ver.texCoords=sf::Vector2f(32.f,32.f);
			garr.append(ver);

			ver.position=sf::Vector2f(gx*32.f,gy*32.f+32.f);
			ver.texCoords=sf::Vector2f(0.f,32.f);
			garr.append(ver);
		}
	};

	int main(int argc, char* argv[])
	{	
		sf::RenderWindow app(sf::VideoMode(600,600),"Tilemap Example");
		app.setVerticalSyncEnabled(true);
		sf::View cam=app.getDefaultView();

		StaticTiledMap testmap;
		ExampleLoader example;
		testmap.LoadFrom(&example);
		while(app.isOpen())
		{
			sf::Event eve;
			while(app.pollEvent(eve))if(eve.type==sf::Event::Closed)app.close();
			if(sf::Keyboard::isKeyPressed(sf::Keyboard::Q))cam.zoom(1.05f);
			if(sf::Keyboard::isKeyPressed(sf::Keyboard::W))cam.move(0.f,-10.f);
			if(sf::Keyboard::isKeyPressed(sf::Keyboard::E))cam.zoom(0.95f);
			if(sf::Keyboard::isKeyPressed(sf::Keyboard::A))cam.move(-10.f,0.f);
			if(sf::Keyboard::isKeyPressed(sf::Keyboard::S))cam.move(0.f,10.f);
			if(sf::Keyboard::isKeyPressed(sf::Keyboard::D))cam.move(10.f,0.f);
			app.setView(cam);
			app.clear();
			app.draw(testmap);
			app.display();
		}
	}
```