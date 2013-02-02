## Tilemap using glsl shader
Posted under MIT License.
TileMap.h
```cpp
#ifndef EE_TILEMAP_H_INCLUDED
#define EE_TILEMAP_H_INCLUDED
#include <SFML/Graphics/Texture.hpp>
#include <SFML/Graphics/Drawable.hpp>
#include <SFML/Graphics/Sprite.hpp>
#include <SFML/Graphics/Shader.hpp>
#include <string>
namespace edy{
	namespace video{
		class TileMap : public sf::Drawable
		{
		public:
			TileMap();
			void loadData(const std::string& filename);
			void loadGraphics(const std::string& filename);
		private:
			void prepUp();
			virtual void draw(sf::RenderTarget& target,sf::RenderStates states)const;
			sf::Texture m_Map,m_Graphics;
			sf::Sprite m_Sprite;
			sf::Shader m_DrawingShader;
		};
	}
}
#endif
```
TileMap.cpp:
```cpp
#include "TileMap.h"
#include <SFML/Graphics/RenderTarget.hpp>
namespace {
	//The code on which this class implementation and shader code is based are subject to following copyright notice:

	//
	// Copyright (c) 2012 by Mickaël Pointier. (http://www.the2dgame.com)
	// This work is made available under the terms of the Creative Commons Attribution-ShareAlike 3.0 Unported license,
	// http://creativecommons.org/licenses/by-sa/3.0/.
	//

	//Which is not included in shader source literals below to save space in final executable -FRex
	char const*const fragSource="#version 130\n\
								uniform sampler2D m;\n\
								uniform sampler2D t;\n\
								uniform vec2 w;\n\
								uniform vec2 z;\n\
								void main()\n\
								{\n\
								vec4 c=texture2D(m,gl_TexCoord[0].xy/32.0);\n\
								float i=floor(c.r*255.0+c.g*65280);\n\
								vec2 a=vec2(mod(i,w.x),floor(i/w.x))/w;\n\
								vec2 b=mod((gl_TexCoord[0].xy*z)/32.0,1.0);\n\
								gl_FragColor=texture2D(t,a+b/w);\n\
								gl_FragColor.a*=c.a;\n\
								}";
	char const*const vertSource="#version 130\n\
								void main()\n\
								{\n\
								gl_Position=gl_ModelViewProjectionMatrix*gl_Vertex;\n\
								gl_TexCoord[0]=gl_TextureMatrix[0]*gl_MultiTexCoord0;\n\
								gl_FrontColor=gl_Color;\n\
								}";
	//TODO: use blue for /something/?
}
namespace edy{
	namespace video{
		TileMap::TileMap()
		{
			m_DrawingShader.loadFromMemory(vertSource,fragSource);
			m_DrawingShader.setParameter("m",m_Map);
			m_DrawingShader.setParameter("t",m_Graphics);
			m_Sprite.setTexture(m_Map);
		}
		void TileMap::loadData(const std::string& filename)
		{
			m_Map.loadFromFile(filename);
			prepUp();
		}
		void TileMap::loadGraphics(const std::string& filename)
		{
			m_Graphics.loadFromFile(filename);
			prepUp();
		}
		void TileMap::prepUp()
		{
			m_DrawingShader.setParameter("w",sf::Vector2f(m_Graphics.getSize()/32u));
			m_DrawingShader.setParameter("z",sf::Vector2f(m_Map.getSize()));
			m_Sprite.setTextureRect(sf::IntRect(0,0,m_Map.getSize().x*32,m_Map.getSize().y*32));
		}
		void TileMap::draw(sf::RenderTarget& target,sf::RenderStates states)const
		{
			states.shader=&m_DrawingShader;
			target.draw(m_Sprite,states);
		}
	}
}
```