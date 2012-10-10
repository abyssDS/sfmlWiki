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

My approach when coding HSL was to colorize grayscale images for my game. I found it dumb to have an image for every color I wanted, and I dislike having to use a big image where everything is lumped in and I have to go through the dimensions of each individual image just to take it out. What I did was use a single grayscale image and colorize it through this method. With this comes a not so obvious problem, which is that grayscale colors have no saturation, so even if you change the hue of the pixel you will get a grayscale image. 

The problem is fixed by adding a fixed saturation to them. The gray pixels have no saturation of their own, but they do have Brightness which defines how close they are to either black or white. Taking that into account you can take an image and colorize it and know that it's shades will remain even if it is a grayscale image.

My code was focused on the usage I will give to my game, which only requires arbitrary precision, therefore I used int as component for the three numbers required by HSL. If you want more precision you can change them to float. However I _do not_ guarantee that it will work correctly. If you want it to work with greater precision and it doesn't work right out the box, you're on your own. Usually the offset in color conversion is not something you can actually perceive so it's usually fine that way.


`
     struct HSL
    { 
    int Hue;
    int Saturation;
    int Luminance;

    HSL();
    HSL(int H, int S, int L);

    sf::Color TurnToRGB();

    private:
    float HueToRGB(float arg1, float arg2, float H);
    /**This function is private for a reason. 
    *You can't just take the hue of a color and hope it will work. 
    *The parameters it receives are the problem.
    *I know not a safe way to calculating Hue without Saturation and Luminance. 
    */

    };
    HSL TurnToHSL(const sf::Color& C);

    ///That's the header, next up is the cpp.

    HSL::HSL()
    :Hue(0)
    ,Saturation(0)
    ,Luminance(0)
    {}

    HSL::HSL(int H, int S, int L)
    {  
    /**Standard used range is (360, 100, 100), you may change it if you like.
    *In the given case you have to make all the proper changes to the convert function.    
    */
        ///Range control for Hue.
        if (H <= 360 && H >= 0) {Hue = H;}
        else
        {
        if(H > 360) { Hue = H%360; }
        else if(H < 0 && H > -360) { Hue = -H; }
        else if(H < 0 && H < -360) { Hue = -(H&360); }
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

    float HSL::HueToRGB(float arg1, float arg2, float H)
    {
    if ( H < 0 ) H += 1;
    if ( H > 1 ) H -= 1;
    if ( ( 6 * H ) < 1 ) { return (arg1 + ( arg2 - arg1 ) * 6 * H); }
    if ( ( 2 * H ) < 1 ) { return arg2; }
    if ( ( 3 * H ) < 2 ) { return ( arg1 + ( arg2 - arg1 ) * ( ( 2 / 3 ) - H ) * 6 ); }
    return arg1;
    }

    sf::Color HSL::TurnToRGB()
    {
    ///Reconvert to range [0,1]
    float H = (float)Hue/360;
    float S = (float)Saturation/100;
    float L = (float)Luminance/100;
    float arg1, arg2;

    if (S <= EPSILON) ///float comparison annoyance.
    {
    sf::Color C(L*255, L*255, L*255);
    return C;
    }
    else
    {
    if ( L < 0.5 ) { arg2 = L * ( 1 + S ); }
    else { arg2 = ( L + S ) - ( S * L ); }
    arg1 = 2 * L - arg2;

    sf::Uint8 r =( 255 * HueToRGB( arg1, arg2, (H + (float)1/3 ) ) );
    sf::Uint8 g =( 255 * HueToRGB( arg1, arg2, H ) );
    sf::Uint8 b =( 255 * HueToRGB( arg1, arg2, (H - (float)1/3 ) ) );
        
    sf::Color C(r,g,b);
    return C;
    }
    }

    HSL TurnToHSL(const sf::Color& C)
    { 
    ///Trivial cases
    /**Note that you can add more than these, like one for each of the basic colors, but that's up to you
       and how you wish to use the class. */

    if(C == sf::Color::White)
    { return HSL(0, 0, 100); } 
    ///Even if these colors had hue, as they have no saturation they'd remain the same

    if(C == sf::Color::Black)
    { return HSL(0, 0, 0); } ///Saturation 100%, Luminance 0%

    float R, G, B;
    R = (float)C.r/255; ///You need that cast there. period.
    G = (float)C.g/255;
    B = (float)C.b/255;

    ///Non trivial cases.
    float max, min, l, s;

    ///Maximum
    max = std::max(std::max(R,G),B);

    ///Minimum
    min = std::min(std::min(R,G),B);

    HSL A;
    l = ((max + min)/2); /// Lightness/Luminance calculation.

    if (max - min <= EPSILON ) ///EPSILON is a macro for 0.0001 or however precision you want.
    ///Needed for the closest thing to an equality comparison between floats.
    {
    A.Hue = 0;
    A.Saturation = 0;
    }
    else
    {
        float diff = max - min;

        if(A.Luminance < .5)
        { s = diff/(max + min); }
    else
    { s = diff/(2 - max - min); }

	float diffR = ( (( max - R ) * 60) + (diff/2) ) / diff;
	float diffG = ( (( max - G ) * 60) + (diff/2) ) / diff;
	float diffB = ( (( max - B ) * 60) + (diff/2) ) / diff;

    /**60 and 360 are values that can vary. It depends if you want the range in 360 or anything
       other than that.  
    *If you change the value then it must be a multiple of six. 360/6 = 60. That's where the 60 comes from.
    *The values below are just the range you want the hue in. 
    */
    if (max - R <= EPSILON) { A.Hue = diffB - diffG; }
    else if ( max - G <= EPSILON ) { A.Hue = (1*360)/3 + (diffR - diffB); }
    else if ( max - B <= EPSILON ) { A.Hue = (2*360)/3 + (diffG - diffR); }

    if (A.Hue < 0) { A.Hue += 360; }
    else if(A.Hue > 360) { A.Hue -= 360; }

    s *= 100; 
    /**Saturation and lightness are given in a [0 , 1] interval. 
      *Needed to get it in the right percentage.     
      *Then again, up to you, you may change it if you want.*/
    }

    l *= 100;
    A.Saturation = s;
    A.Luminance = l;
    return A;
    }
`

I think that all that remains is the '=' operator, but it follows the same syntax the constructor does, all you have to do is use the same logic to it.