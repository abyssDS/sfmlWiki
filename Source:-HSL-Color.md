This wiki is about the usage of HSL (Hue, Saturation, Luminance/Lightness/Brightness) in SFML, not to be confused with HSV which is similar but still different.

Let's start with a sort of long explanation. If you truly want to learn about HSL you have to read a lot, so believe it or not I'll save you a bit of that. Believe me, reading and trying to comprehend the formula is way harder than taking your time and just reading a bit of how this works.

Some Eyecandy before you get to this huge wall of text:
![Normal Image](http://www.tannerhelland.com/wp-content/uploads/Enslaved-Odyssey-to-the-West-normal.jpg)


![Colorized Image](http://www.tannerhelland.com/wp-content/uploads/Enslaved-Odyssey-to-the-West-blue-original-saturation.jpg)

HSL is color standard that works on a polar color hexagon (check wikipedia for further explanation), The hue is a color that is represented in the said hexagon. The color given is a raw one, not taking shading or lighting directly. All hue does is grant you a 0 to 360 range between colors allowing you to take intermediate values between base colors. Base colors are: Red, Green, Blue, Magenta, Yellow and Cyan.

Next up is the Saturation. Saturation says how much "color component" your color will have. Grayscale colors have no saturation, which means that regardless of the Hue they have they'll always be "colorless". Saturation is what divides pure color from weaker forms of it. For example a color with 0 Hue (remember it's degrees in a polar hexagon, 0 means a redish tone), 100% Saturation and 50% Luminance is equal to an 
RGB (255, 0, 0), which is pure red, while if you take HSL (0, 50%, 50%) you will have a slightly dark tone of pink. If you want to test it a little further check [this](http://serennu.com/colour/hsltorgb.php) page, it has an RGB/HSL/Hex converter and will show you the color you type in either format.

Finally there is Luminance/Lightness/Brightness. This defines the if the color is bright or not. 50% is the brightness all base colors have. Brightness is what differences a dark tone pink to a bright one. Those two would have the same Hue and Saturation, but a different L parameter is what makes the difference.

HSL allows a good coloring system and is perhaps more intuitive than RGB once you get used to it, however _it is not practical_. First of all, the most used color standard is RGB for a reason, that's how most GPU's view color. Secondly, while there are conversions from HSL to RGB and viceversa, they are a pain. Not only are the formulas hard to understand, but as RGB is not represented in floats it will give you many problems with number types if you don't handle them well, regardless if you use float or int for the HSL class. Yet if this was all there was to them there would be no purpose in making the HSL class. HSL grants you something plain RGB will not be able to do at all, _colorizing_. (Check [this](http://www.tannerhelland.com/3552/colorize-image-vb6/) page for a graphic example.)

Colorizing is a magnificent tool granted by HSL. Why? Because all you have to do is change the hue of a color and it shifts entirely according to it's saturation and brightness, meaning you can change a shaded green mountain to a blue one with just that shift in every pixel of the image.

There's many uses to HSL in general, from colorizing grayscale images to colored ones, shifting the overall hue, making a colorful image a grayscale one and so on.

After the rework, the code works for many use cases now, not just simple images or color code checking. There will be some fixing to make it even more accurate for the purpose of image handling, so this fix-up will be temporary unless I find no other way to make it better, feel free to use it for image handling or whatever you may like.

Hue shifting doesn't work as well as photoshop's just yet, but it now actually produces a nice looking image when setting upon a complex image with tons of colors. All the annoying pixel artifacts are gone.

```cpp
#ifndef HSL_COLOR
#define HSL_COLOR

#include <SFML/Graphics/Color.hpp>
#include <algorithm>
#include <cmath>

struct HSL
{
    double Hue;
    double Saturation;
    double Luminance;

    HSL();
    HSL(int H, int S, int L);

    sf::Color TurnToRGB();

    private:

    double HueToRGB(double arg1, double arg2, double H);

};

HSL TurnToHSL(const sf::Color& C);

#endif // HSL_COLOR


/// That's the header, next up is the cpp.

#include "HSL.hpp"
const double D_EPSILON = 0.00000000000001; 
///Feel free to move this to your constants .h file or use it as a static constant if you don't like it here.

HSL::HSL() :Hue(0) ,Saturation(0) ,Luminance(0) {}

HSL::HSL(int H, int S, int L)
{
    ///Range control for Hue.
    if (H <= 360 && H >= 0) {Hue = H;}
    else
    {
    if(H > 360) { Hue = H%360; }
    else if(H < 0 && H > -360) { Hue = -H; }
    else if(H < 0 && H < -360) { Hue = -(H%360); }
    }

    ///Range control for Saturation.
    if (S <= 100 && S >= 0) {Saturation = S;}
    else
    {
    if(S > 100) { Saturation = S%100;}
    else if(S < 0 && S > -100) { Saturation = -S; }
    else if(S < 0 && S < -100) { Saturation = -(S%100); }
    }

    ///Range control for Luminance
    if (L <= 100 && L >= 0) {Luminance = L;}
    else
    {
    if(L > 100) { Luminance = L%100;}
    if(L < 0 && L > -100) { Luminance = -L; }
    if(L < 0 && L < -100) { Luminance = -(L%100); }
    }

}

double HSL::HueToRGB(double arg1, double arg2, double H)
{
   if ( H < 0 ) H += 1;
   if ( H > 1 ) H -= 1;
   if ( ( 6 * H ) < 1 ) { return (arg1 + ( arg2 - arg1 ) * 6 * H); }
   if ( ( 2 * H ) < 1 ) { return arg2; }
   if ( ( 3 * H ) < 2 ) { return ( arg1 + ( arg2 - arg1 ) * ( ( 2.0 / 3.0 ) - H ) * 6 ); }
   return arg1;
}

sf::Color HSL::TurnToRGB()
{
    ///Reconvert to range [0,1]
    double H = Hue/360.0;
    double S = Saturation/100.0;
    double L = Luminance/100.0;

    float arg1, arg2;

    if (S <= D_EPSILON)
    {
        sf::Color C(L*255, L*255, L*255);
        return C;
    }
    else
    {
        if ( L < 0.5 ) { arg2 = L * ( 1 + S ); }
        else { arg2 = ( L + S ) - ( S * L ); }
        arg1 = 2 * L - arg2;

        sf::Uint8 r =( 255 * HueToRGB( arg1, arg2, (H + 1.0/3.0 ) ) );
        sf::Uint8 g =( 255 * HueToRGB( arg1, arg2, H ) );
        sf::Uint8 b =( 255 * HueToRGB( arg1, arg2, (H - 1.0/3.0 ) ) );
        sf::Color C(r,g,b);
        return C;
    }

}

HSL TurnToHSL(const sf::Color& C)
{
    ///Trivial cases.
    if(C == sf::Color::White)
    { return HSL(0, 0, 100); }

    if(C == sf::Color::Black)
    { return HSL(0, 0, 0); } 

    if(C == sf::Color::Red)
    { return HSL(0, 100, 50); }

    if(C == sf::Color::Yellow)
    { return HSL(60, 100, 50); }

    if(C == sf::Color::Green)
    { return HSL(120, 100, 50); }

    if(C == sf::Color::Cyan)
    { return HSL(180, 100, 50); }

    if(C == sf::Color::Blue)
    { return HSL(240, 100, 50); }

    if(C == sf::Color::Cyan)
    { return HSL(300, 100, 50); }

    double R, G, B;
    R = C.r/255.0;
    G = C.g/255.0;
    B = C.b/255.0;
    ///Casos no triviales.
    double max, min, l, s;

    ///Maximos
    max = std::max(std::max(R,G),B);

    ///Minimos
    min = std::min(std::min(R,G),B);


    HSL A;
    l = ((max + min)/2.0);

    if (max - min <= D_EPSILON )
    {
        A.Hue = 0;
        A.Saturation = 0;
    }
    else
    {
        double diff = max - min;

        if(A.Luminance < 0.5)
        { s = diff/(max + min); }
    else
    { s = diff/(2 - max - min); }

	double diffR = ( (( max - R ) * 60) + (diff/2.0) ) / diff;
	double diffG = ( (( max - G ) * 60) + (diff/2.0) ) / diff;
	double diffB = ( (( max - B ) * 60) + (diff/2.0) ) / diff;


    if (max - R <= D_EPSILON) { A.Hue = diffB - diffG; }
    else if ( max - G <= D_EPSILON ) { A.Hue = (1*360)/3.0 + (diffR - diffB); }
    else if ( max - B <= D_EPSILON ) { A.Hue = (2*360)/3.0 + (diffG - diffR); }

    if (A.Hue <= 0 || A.Hue >= 360) { fmod(A.Hue, 360); }

    s *= 100;
    }

    l *= 100;
    A.Saturation = s;
    A.Luminance = l;
    return A;
}

```

I think that all that remains is the '=' operator, but it follows the same syntax the constructor does, all you have to do is use the same logic to it.