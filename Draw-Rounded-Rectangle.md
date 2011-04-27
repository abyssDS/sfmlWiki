```c++
enum RoundedCorners {
	BR = 1,
	BL = 2,
	TL = 4,
	TR = 8,
};

sf::Shape RRectangle(float P1X, float P1Y, 
			float P2X, float P2Y, 
			float Radius, int Corners, 
			const sf::Color& Col, float Outline = 1.f, const sf::Color& OutlineCol = Color::Cyan)
{
	static const int NbSegments = 15;
	static const float PI_HALF = 3.141592654f / 2.0f;
	static const float SegmentRadian = PI_HALF/ (float)NbSegments;

	// Create the shape's points
	sf::Shape S;
	if((Corners&BR) == BR) {
		
		sf::Vector2f Center(P2X - Radius, P2Y - Radius);
		for (int i = 0; i <= NbSegments; ++i)
		{
			float Angle = (float)i * SegmentRadian;
			sf::Vector2f Offset(cos(Angle), sin(Angle));

			S.AddPoint(Center + Offset * Radius, Col, OutlineCol);
		}
		
	} else {
		S.AddPoint(sf::Vector2f(P2X, P2Y), Col, OutlineCol);
	}
	if((Corners&BL) == BL) {
		
		sf::Vector2f Center(P1X + Radius, P2Y - Radius);
		for (int i = 0; i <= NbSegments; ++i)
		{
			float Angle = (float)i * SegmentRadian + PI_HALF;
			sf::Vector2f Offset(cos(Angle), sin(Angle));

			S.AddPoint(Center + Offset * Radius, Col, OutlineCol);
		}
		
	} else {
		S.AddPoint(sf::Vector2f(P1X, P2Y), Col, OutlineCol);
	} 
	if((Corners&TL) == TL) {
		sf::Vector2f Center(P1X + Radius, P1Y + Radius);
		for (int i = 0; i <= NbSegments; ++i)
		{
			float Angle = (float)i * SegmentRadian + 2.0f * PI_HALF;
			sf::Vector2f Offset(cos(Angle), sin(Angle));

			S.AddPoint(Center + Offset * Radius, Col, OutlineCol);
		}
	} else {
		S.AddPoint(sf::Vector2f(P1X, P2Y), Col, OutlineCol);
	} 
	if((Corners&TR) == TR) {
		sf::Vector2f Center(P2X - Radius, P1Y + Radius);
		for (int i = 0; i <= NbSegments; ++i)
		{
			float Angle = (float)i * SegmentRadian + 3.0f * PI_HALF;
			sf::Vector2f Offset(cos(Angle), sin(Angle));

			S.AddPoint(Center + Offset * Radius, Col, OutlineCol);
		}
	} else {
		S.AddPoint(sf::Vector2f(P2X, P1Y), Col, OutlineCol);
	}
	
    
	S.SetOutlineWidth(Outline);

	return S;
}
```