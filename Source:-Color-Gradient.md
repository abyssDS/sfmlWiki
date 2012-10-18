# Drawing color gradient

## Use Case
### Create scale Color

This class allow you to easily create a color gradient by adding a color and its position. It handle the creation of a color table. You can choose between two interpolation methods : Linear and Cosinus  

This picture show you the result of the next source code.  
 [[http://www.sfml-dev.org/wiki/_media/fr/sources/basicscalecolor.png]]

### main.cpp
```cpp
#include "ColorScale.h"

int main()
{

	const int SIZE = 500;
	ColorScale gradient;
	gradient.insert(0.0,sf::Color::Red);
	gradient.insert(1.0,sf::Color::Magenta);
	gradient.insert(2.0,sf::Color::Blue);
	gradient.insert(3.0,sf::Color::Cyan);
	gradient.insert(4.0,sf::Color::Green);
	gradient.insert(5.0,sf::Color::Yellow);
	gradient.insert(6.0,sf::Color::Red);
	
	
	sf::Color* tab =new sf::Color[SIZE];
	gradient.fillTab(tab,SIZE);
	
	sf::RenderWindow App(sf::VideoMode(SIZE, SIZE, 32), "Gradient");
	App.SetFramerateLimit(30);
    
	sf::Image image(SIZE,SIZE);
	for(int i=0;i<SIZE;i++)
	{
		for(int j=0;j<SIZE;j++)
		{
			image.SetPixel(i,j,tab[j]);
		}		
	}
	
	sf::Sprite sprite(image);
	
	while(App.IsOpened())
	{
		sf::Event Event;
		while (App.GetEvent(Event)) 
		{
			if (Event.Type==sf::Event::Closed)
			{
				App.Close();
			} 
		}
		
		App.Clear();
		App.Draw(sprite);
		App.Display();
	}
	
	
	delete[] tab;	
	return 0;
}
```

## Automatic Gradient

Previously we have created a scale color and were trying to draw it by ourselves. But this class also knows how to draw it on a picture. First create your scale color then your picture and choose two point (which can be outside the picture). And this class will draw it for you. You can draw for type of gradient: linear, radial, circle and reflexion.  
There you have an example of each of them and the source code which has created them. 
[[http://www.sfml-dev.org/wiki/_media/fr/sources/reflexgradiant.png]]
[[http://www.sfml-dev.org/wiki/_media/fr/sources/circlegradiant.png]]
[[http://www.sfml-dev.org/wiki/_media/fr/sources/lineargradiant.png]]
[[http://www.sfml-dev.org/wiki/_media/fr/sources/radialgradiant.png]]

```cpp
#include <SFML/Graphics.hpp>
#include "ColorScale.h"

int main()
{
	
	const int SIZE = 500;
	ColorScale gradient;
	gradient.insert(0.0,sf::Color::Red);
	gradient.insert(1.0,sf::Color::Magenta);
	gradient.insert(2.0,sf::Color::Blue);
	gradient.insert(3.0,sf::Color::Cyan);
	gradient.insert(4.0,sf::Color::Green);
	gradient.insert(5.0,sf::Color::Yellow);
	gradient.insert(6.0,sf::Color::Red);
	
	
	sf::Image image(SIZE,SIZE);
	gradient.draw(image,sf::Vector2f(250.0,250.0),sf::Vector2f(200.0,0.0),GradientStyle::Radial);
	
	sf::Sprite sprite(image);
	sf::RenderWindow App(sf::VideoMode(SIZE, SIZE, 32), "Gradient");
	App.Draw(sprite);
	App.Display();
	App.SetFramerateLimit(1);
	
	while(App.IsOpened())
	{
		sf::Event Event;
		while(App.GetEvent(Event));
		App.Display();
	}
	
	return 0;
}
```

## ColorScale Class
### ColorScale.h
```cpp
/*
 *  ColorScale.h
 *
 *  Created by mooglwy on 31/10/10.
 *
 *  This software is provided 'as-is', without any express or
 *  implied warranty. In no event will the authors be held
 *  liable for any damages arising from the use of this software.
 *  
 *  Permission is granted to anyone to use this software for any purpose,
 *  including commercial applications, and to alter it and redistribute
 *  it freely, subject to the following restrictions:
 *  
 *  1. The origin of this software must not be misrepresented;
 *     you must not claim that you wrote the original software.
 *     If you use this software in a product, an acknowledgment
 *     in the product documentation would be appreciated but
 *     is not required.
 *  
 *  2. Altered source versions must be plainly marked as such,
 *     and must not be misrepresented as being the original software.
 *  
 *  3. This notice may not be removed or altered from any
 *     source distribution.
 *
 */

#ifndef _COLOR_SCALE_
#define _COLOR_SCALE_
#include <map>
#include <SFML/Graphics/Color.hpp>
#include <SFML/Graphics.hpp>

namespace InterpolationFunction 
{
	enum InterpolationFunction
	{
		Linear,
		Cosinus
	};
}

namespace GradientStyle 
{
	enum GradientStyle 
	{
		Linear,
		Circle,
		Radial,
		Reflex
	};
}

class ColorScale : protected std::map<double,sf::Color>
{
public:
	
	ColorScale();
	bool insert(double position, sf::Color color);
	
	void fillTab(sf::Color* colorTab, int size, InterpolationFunction::InterpolationFunction function = InterpolationFunction::Cosinus) const;
	void draw(sf::Image&,const sf::Vector2f& start,const sf::Vector2f& end,GradientStyle::GradientStyle style=GradientStyle::Linear, int size = 500) const;
	
};


#endif //end of _COLOR_SCALE_
```

### ColorScale.cpp
```cpp
/*
 *  ColorScale.cpp
 *
 *  Created by mooglwy on 31/10/10.
 *
 *  This software is provided 'as-is', without any express or
 *  implied warranty. In no event will the authors be held
 *  liable for any damages arising from the use of this software.
 *  
 *  Permission is granted to anyone to use this software for any purpose,
 *  including commercial applications, and to alter it and redistribute
 *  it freely, subject to the following restrictions:
 *  
 *  1. The origin of this software must not be misrepresented;
 *     you must not claim that you wrote the original software.
 *     If you use this software in a product, an acknowledgment
 *     in the product documentation would be appreciated but
 *     is not required.
 *  
 *  2. Altered source versions must be plainly marked as such,
 *     and must not be misrepresented as being the original software.
 *  
 *  3. This notice may not be removed or altered from any
 *     source distribution.
 *
 */

#include "ColorScale.h"
#include <cmath>
#include <algorithm>

const double PI = 3.1415926535;

double linearInterpolation(double v1, double v2, double mu)
{
	return v1*(1-mu)+v2*mu;
}

double interpolateCosinus(double y1, double y2, double mu)
{
	double mu2;
	
	mu2 = (1-cos(mu*PI))/2;
	return y1*(1-mu2)+y2*mu2;
}

sf::Color GradientLinear(sf::Color* colorTab,int size,const sf::Vector2f& start,const sf::Vector2f& end,int x,int y)
{
	sf::Vector2f dir  = end-start;
	sf::Vector2f pix  = sf::Vector2f(x,y)-start; 
	double dotProduct = pix.x*dir.x+pix.y*dir.y;
	dotProduct       *= (size-1)/(dir.x*dir.x+dir.y*dir.y);
	
	if((int)dotProduct < 0.0      ) return colorTab[0];
	if((int)dotProduct > (size-1) ) return colorTab[size-1];
	return colorTab[(int)dotProduct];
}

sf::Color GradientCircle(sf::Color* colorTab,int size,const sf::Vector2f& start,const sf::Vector2f& end,int x,int y)
{
	sf::Vector2f v_radius  = end-start;
	double radius          = std::sqrt(v_radius.x*v_radius.x+v_radius.y*v_radius.y);
	sf::Vector2f pix       = sf::Vector2f(x,y)-start;
	double dist            = std::sqrt(pix.x*pix.x+pix.y*pix.y);
	dist                  *= (size-1)/radius;
	
	if((int)dist < 0.0      ) return colorTab[0];
	if((int)dist > (size-1) ) return colorTab[size-1];
	return colorTab[(int)dist];
}

sf::Color GradientRadial(sf::Color* colorTab,int size,const sf::Vector2f& start,const sf::Vector2f& end,int x,int y)
{
	sf::Vector2f base     = end-start;
	base                 /= (float)std::sqrt(base.x*base.x+base.y*base.y);
	sf::Vector2f pix      = sf::Vector2f(x,y)-start;
	pix                  /= (float)std::sqrt(pix.x*pix.x+pix.y*pix.y);
	double angle          = std::acos(pix.x*base.x+pix.y*base.y);
	double aSin           = pix.x*base.y-pix.y*base.x;
	if( aSin < 0) angle   = 2*PI-angle;
	angle                *= (size-1)/(2*PI);
	
		
	if((int)angle < 0.0      ) return colorTab[0];
	if((int)angle > (size-1) ) return colorTab[size-1];
	return colorTab[(int)angle];
}

sf::Color GradientReflex(sf::Color* colorTab,int size,const sf::Vector2f& start,const sf::Vector2f& end,int x,int y)
{
	sf::Vector2f dir  = end-start;
	sf::Vector2f pix  = sf::Vector2f(x,y)-start; 
	double dotProduct = pix.x*dir.x+pix.y*dir.y;
	dotProduct       *= (size-1)/(dir.x*dir.x+dir.y*dir.y);
	dotProduct        = std::abs(dotProduct);
	
	if((int)dotProduct < 0.0      ) return colorTab[0];
	if((int)dotProduct > (size-1) ) return colorTab[size-1];
	return colorTab[(int)dotProduct];
}

ColorScale::ColorScale()
{
}

bool ColorScale::insert(double position, sf::Color color)
{
	std::pair< ColorScale::iterator,bool > ret = std::map<double,sf::Color>::insert(std::make_pair(position,color));
	return ret.second;
}


#define ABS(a) (std::max(a, 0))

void ColorScale::fillTab(sf::Color* colorTab, int size,InterpolationFunction::InterpolationFunction function) const
{
	
	ColorScale::const_iterator start = std::map<double,sf::Color>::begin();
	ColorScale::const_iterator last  = std::map<double,sf::Color>::end();
	last--;
		
	double pos = 0.0;
	double distance = last->first - start->first;
	ColorScale::const_iterator it =  start;
	
	double(*pFunction)(double,double,double);
 	
	switch (function) 
	{
		case InterpolationFunction::Cosinus: pFunction = interpolateCosinus;  break;
		case InterpolationFunction::Linear : pFunction = linearInterpolation; break;
		default: pFunction = interpolateCosinus;  break;
			
	}
	while(it!=last)
	{
		sf::Color startColor = it->second;
		double    startPos   = it->first;
		it++;
		sf::Color endColor   = it->second;
		double    endPos     = it->first;
		double nb_color         = ((endPos-startPos)*(double)size/distance);
		
		for(int i = (int)pos;i<=(int)(pos+nb_color);i++)
		{
			colorTab[i].r = (unsigned char)pFunction(startColor.r,endColor.r,ABS((double)i-pos)/(nb_color-1.0));
			colorTab[i].g = (unsigned char)pFunction(startColor.g,endColor.g,ABS((double)i-pos)/(nb_color-1.0));
			colorTab[i].b = (unsigned char)pFunction(startColor.b,endColor.b,ABS((double)i-pos)/(nb_color-1.0));
			colorTab[i].a = (unsigned char)pFunction(startColor.a,endColor.a,ABS((double)i-pos)/(nb_color-1.0));
		}
		pos+=nb_color;
	}
	
}

#undef ABS

void ColorScale::draw(sf::Image& img,const sf::Vector2f& start,const sf::Vector2f& end,GradientStyle::GradientStyle style, int size) const
{
	
	sf::Color (*pFunction)(sf::Color*,int,const sf::Vector2f&,const sf::Vector2f&,int,int);
	
	sf::Color* tab =new sf::Color[size];
	fillTab(tab,size);

	switch (style) 
	{
		case GradientStyle::Linear : pFunction = GradientLinear; break;
		case GradientStyle::Circle : pFunction = GradientCircle; break;
		case GradientStyle::Radial : pFunction = GradientRadial; break;
		case GradientStyle::Reflex : pFunction = GradientReflex; break;

		default: pFunction = GradientLinear;  break;
	}
	
	for(int i=0;i<img.GetWidth();i++)
	{
		for(int j=0;j<img.GetHeight();j++)
		{
			img.SetPixel(i,j,pFunction(tab,size,start,end,i,j));
		}		
	}
	delete[] tab;
}
```

## Credits
This article was originally written by the user mooglwy.