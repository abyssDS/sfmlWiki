## Class summary
This is an SFML-based class that produces a fast-rendering shape drawn with display lists and GL_LNES.
Inherits from sf::Drawable, but some of those functions don't work, specifically SetColor since it's overrided per line segment.
To change the color, use the SetLineSegmentColor method.
To create a shape, use AddPoint to add all the vertices, catching the returned ID.  Then connect them with AddLineSegment.
The shape can then be modified by adding/removing/moving points and line segments

This class meant to work with SFML (www.sfml-dev.org) and is released under the same liscence as SFML, zlib/png.

Class written by "Quasius"

## DrawableLineShape.h

```cpp
	/*
	A class that produces a fast-rendering shape drawn with display lists and GL_LNES.
	Inherits from sf::Drawable, but some of those functions don't work, specifically SetColor since that's overridden per line-segment (default white).
	To change the color, use the SetLineSegmentColor method.
	To create a shape, use AddPoint to add all the vertices, catching the returned ID.  Then connect them with AddLineSegment.
	 
	This class meant to work with SFML (www.sfml-dev.org) and is released under the same liscence as SFML, zlib/png.
	Class written by "Quasius"
	*/
	 
	 
	#ifndef DRAWABLE_LINE_SHAPE
	#define DRAWABLE_LINE_SHAPE
	 
	#include <vector>
	 
	#include <SFML/Graphics.hpp>
	 
	using namespace std;
	using namespace sf;
	 
	class DrawableLineShape : public sf::Drawable
	{
		//DEFINITIONS, STRUCTS, ETC
	protected:
	 
		struct LineSegmentInfo
		{
			unsigned short nPoint1, nPoint2;
			sf::Color color;
			//false if this line segment is in a recycled state
			bool bActive;
	 
			LineSegmentInfo(const unsigned short nPoint1ID, const unsigned short nPoint2ID, const Color& lineColor = Color(255, 255, 255)) :
					bActive(true)
			{
				nPoint1 = nPoint1ID;
				nPoint2 = nPoint2ID;
				color = lineColor;
			}
		};
	 
		struct PointInfo
		{
			Vector2f v2fPoint;
			//false if this point is in a recycled state
			bool bActive;
	 
			PointInfo(const Vector2f& v2fLinePoint = Vector2f(0.0f, 0.0f)) :
				bActive(true)
			{
				v2fPoint = v2fLinePoint;
			}
		};
	 
		//METHODS
	public:
	 
		//Constructur
		DrawableLineShape();
	 
		//Assignment operator
		DrawableLineShape& operator = (const DrawableLineShape& that);
	 
		//Destructor
		~DrawableLineShape();
	 
		//adds a point to the line shape
		//returns an ID for that point
		unsigned short AddPoint(const Vector2f& v2fPointPosition);
	 
		//adds a point to the line shape
		//returns an ID for that point
		unsigned short AddPoint(const float fXPosition, const float fYPosition);
	 
		//removes the passed point.
		//Also removes any line segments that depended on that point
		void RemovePoint(const unsigned short nPointID);
	 
		//Moves the position of the point (Attached line segments automatically move appropriately)
		void MovePoint(const unsigned short nPointID, const Vector2f& v2fNewPosition);
	 
		//Moves the position of the point (Attached line segments automatically move appropriately)
		void MovePoint(const unsigned short nPointID, const float fNewPositionX, const float fNewPositionY);
	 
		//Adds a line segment that references 2 point IDs
		//Does nothing if both point IDs don't exist and returns -1
		//Object won't render if there isn't at least 1 line segment
		//Returns an ID to the line segment
		unsigned short AddLineSegment(const unsigned short nPoint1ID, const unsigned short nPoint2ID, const Color& color = sf::Color(255, 255, 255));
	 
		//removes the passed line segment.
		//Also removes any no-longer used points if bRemoveOrphanPoints is true
		void RemoveLineSegment(const unsigned short nLineSegmentID, bool bRemoveOrphanedPoints = true);
	 
		//sets the thickness of the passed line segment
		void SetLineThickness(const float fLineThickness);
	 
		//sets the color of the passed line segment
		void SetLineSegmentColor(const unsigned short nLineSegmentID, const Color& color);
	 
		//returns the color of the passed line segment ID
		const Color& GetLineSegmentColor(const unsigned short nLineSegmentID) const;
	 
		//gets the number of current line segments
		int GetNumLineSegments();
	 
		//removes all points and line segments
		void Clear();
	 
	protected:
	 
		//See Drawable::Render
		virtual void Render(const RenderWindow& Window);
	 
		//MEMBERS
	protected:
	 
		//All the points
		vector<PointInfo> m_vPoints;
		//All the line segments
		vector<LineSegmentInfo> m_vLineSegments;
		//the line thickness
		float m_fLineThickness;
		//Stores if the display list needs to be recompiled or not
		bool m_bIsDisplayListCurrent;
		//the ID for the OpenGL display list
		int m_nDisplayListID;
	};
	 
	#endif	
```

## DrawableLineShape.cpp
```cpp
    #include "DrawableLineShape.h"
     
    //Constructur
    DrawableLineShape::DrawableLineShape() :
    m_fLineThickness(1.0f),
    m_bIsDisplayListCurrent(false)
    {
    	m_nDisplayListID = glGenLists(1);
    }
     
    //Destructor
    DrawableLineShape::~DrawableLineShape()
    {
    	glDeleteLists(m_nDisplayListID, 1);
    }
     
    //Assignment operator
    DrawableLineShape& DrawableLineShape::operator = (const DrawableLineShape& that)
    {
    	if (this != &that)
    	{
    		m_vPoints = that.m_vPoints;
    		m_vLineSegments = that.m_vLineSegments;
    		m_bIsDisplayListCurrent = that.m_bIsDisplayListCurrent;
    	}
     
    	return *this;
    }
     
    //adds a point to the line shape
    //returns an ID for that point
    unsigned short DrawableLineShape::AddPoint(const sf::Vector2f& v2fPointPosition)
    {
    	//search for a recycled point to use
    	for (unsigned short i = 0; i < m_vPoints.size(); ++i)
    	{
    		if (!m_vPoints[i].bActive)
    		{
    			m_vPoints[i].v2fPoint = v2fPointPosition;
    			m_vPoints[i].bActive = true;
    			return i;
    		}
    	}
     
    	//if no recycled point, add a new point
    	m_vPoints.push_back(PointInfo(v2fPointPosition));
    	return (unsigned short)(m_vPoints.size() - 1);
     
    	//display commands are out-of-date now
    	m_bIsDisplayListCurrent = false;
    }
     
    //adds a point to the line shape
    //returns an ID for that point
    unsigned short DrawableLineShape::AddPoint(const float fXPosition, const float fYPosition)
    {
    	return AddPoint(sf::Vector2f(fXPosition, fYPosition));
    }
     
    //removes the passed point.
    //Also removes any line segments that depended on that point
    void DrawableLineShape::RemovePoint(const unsigned short nPointID)
    {
    	//remove the point
    	m_vPoints[nPointID].bActive = false;
     
    	//remove all line segments referencing that point
    	for (unsigned int i = 0; i < m_vLineSegments.size(); ++i)
    	{
    		if (m_vLineSegments[i].bActive && (m_vLineSegments[i].nPoint1 == nPointID || m_vLineSegments[i].nPoint2 == nPointID))
    			RemoveLineSegment(i);
    	}
     
    	//display commands are out-of-date now
    	m_bIsDisplayListCurrent = false;
    }
     
    //Moves the position of the point (Attached line segments automatically move appropriately)
    void DrawableLineShape::MovePoint(const unsigned short nPointID, const Vector2f& v2fNewPosition)
    {
    	m_vPoints[nPointID].v2fPoint = v2fNewPosition;
     
    	//display commands are out-of-date now
    	m_bIsDisplayListCurrent = false;
    }
     
    //Moves the position of the point (Attached line segments automatically move appropriately)
    void DrawableLineShape::MovePoint(const unsigned short nPointID, const float fNewPositionX, const float fNewPositionY)
    {
    	m_vPoints[nPointID].v2fPoint.x = fNewPositionX;
    	m_vPoints[nPointID].v2fPoint.y = fNewPositionY;
     
    	//display commands are out-of-date now
    	m_bIsDisplayListCurrent = false;
    }
     
    //Adds a line segment that references 2 point IDs
    //Does nothing if both point IDs don't exist and returns -1
    //Object won't render if there isn't at least 1 line segment
    //Returns an ID to the line segment
    unsigned short DrawableLineShape::AddLineSegment(const unsigned short nPoint1ID, const unsigned short nPoint2ID, const sf::Color& color)
    {
    	//search for a recycled line segment to use
    	for (unsigned short = 0; i < m_vLineSegments.size(); ++i)
    	{
    		if (!m_vLineSegments[i].bActive)
    		{
    			m_vLineSegments[i].nPoint1 = nPoint1ID;
    			m_vLineSegments[i].nPoint2 = nPoint2ID;
    			m_vLineSegments[i].color = color;
    			m_vLineSegments[i].bActive = true;
    			return (unsigned short)i;
    		}
    	}
     
    	//if no recycled line segment, add a new one
    	m_vLineSegments.push_back(LineSegmentInfo(nPoint1ID, nPoint2ID, color));
    	return (unsigned short)(m_vLineSegments.size() - 1);
     
    	//display commands are out-of-date now
    	m_bIsDisplayListCurrent = false;
    }
     
    //removes the passed line segment.
    //Also removes any no-longer used points if bRemoveOrphanPoints is true
    void DrawableLineShape::RemoveLineSegment(const unsigned short nLineSegmentID, bool bRemoveOrphanedPoints)
    {
    	//remove the line segment
    	m_vLineSegments[nLineSegmentID].bActive = false;
     
    	//remove all newly-orphaned points, if requested
    	if (bRemoveOrphanedPoints)
    	{
    		//get the point IDs (we deactivated the segment above, but not change/delete values)
    		unsigned short nPointID1 = m_vLineSegments[nLineSegmentID].nPoint1;
    		unsigned short nPointID2 = m_vLineSegments[nLineSegmentID].nPoint2;
     
    		//assume no reference until found
    		bool bPoint1Referenced = false;
    		bool bPoint2Referenced = false;
     
    		//iterate through lines searching for references to both points
    		for (unsigned int nLineIndex = 0; nLineIndex < m_vLineSegments.size(); ++nLineIndex)
    		{
    			if (!m_vLineSegments[nLineIndex].bActive)
    				continue;
     
    			if (m_vLineSegments[nLineIndex].nPoint1 == nPointID1 || m_vLineSegments[nLineIndex].nPoint2 == nPointID1)
    				bPoint1Referenced = true;
    			if (m_vLineSegments[nLineIndex].nPoint1 == nPointID2 || m_vLineSegments[nLineIndex].nPoint2 == nPointID2)
    				bPoint2Referenced = true;
     
    			//early out
    			if (bPoint1Referenced && bPoint2Referenced)
    				break;
    		}
     
    		//remove unreferenced points
    		if (!bPoint1Referenced)
    			m_vPoints[nPointID1].bActive = false;
    		if (!bPoint2Referenced)
    			m_vPoints[nPointID2].bActive = false;
    	}
     
    	//display commands are out-of-date now
    	m_bIsDisplayListCurrent = false;
    }
     
    //sets the thickness of the passed line segment
    void DrawableLineShape::SetLineThickness(const float fLineThickness)
    {
    	m_fLineThickness = fLineThickness;
     
    	//display commands are out-of-date now
    	m_bIsDisplayListCurrent = false;
    }
     
    //sets the color of the passed line segment
    void DrawableLineShape::SetLineSegmentColor(const unsigned short nLineSegmentID, const sf::Color& color)
    {
    	m_vLineSegments[nLineSegmentID].color = color;
     
    	//display commands are out-of-date now
    	m_bIsDisplayListCurrent = false;
    }
     
    //returns the color of the passed line segment ID
    const Color& DrawableLineShape::GetLineSegmentColor(const unsigned short nLineSegmentID) const
    {
    	return m_vLineSegments[nLineSegmentID].color;
    }
     
     
    //gets the number of current line segments
    int DrawableLineShape::GetNumLineSegments()
    {
    	int nNumLineSegments = 0;
     
    	//count each active line segment
    	for (unsigned int i = 0; i < m_vLineSegments.size(); ++i)
    	{
    		if (m_vLineSegments[i].bActive)
    			++nNumLineSegments;
    	}
     
    	return nNumLineSegments;
    }
     
    //removes all points and line segments
    void DrawableLineShape::Clear()
    {
    	unsigned int i;
    	//deactivate points
    	for (i = 0; i < m_vPoints.size(); ++i)
    		m_vPoints[i].bActive = false;
    	//deactivate line segments
    	for (i = 0; i < m_vLineSegments.size(); ++i)
    		m_vLineSegments[i].bActive = false;
     
    	//display commands are out-of-date now
    	m_bIsDisplayListCurrent = false;
    }
     
     
    //See Drawable::Render
    void DrawableLineShape::Render(const RenderWindow& Window)
    {
    	//if the display list is current, just run it
    	if (m_bIsDisplayListCurrent)
    	{
    		glCallList(m_nDisplayListID);
    		return;
    	}
     
    	// Make sure the shape has at least 1 line segment
    	if (m_vLineSegments.size() < 1)
    		return;
     
    	//store all these generated commands in the display list
    	glNewList(m_nDisplayListID, GL_COMPILE_AND_EXECUTE);
     
    	// Shapes only use color, no texture
    	glDisable(GL_TEXTURE_2D);
     
    	// Draw the shape
    	Color lineColor;
    	Vector2f v2fLinePoint1, v2fLinePoint2;
     
    	//set line thickness
    	glLineWidth(m_fLineThickness);
    	glBegin(GL_LINES);
    	for (unsigned int i = 0; i < m_vLineSegments.size(); ++i)
    	{
    		if (!m_vLineSegments[i].bActive)
    			continue;
     
    		//set color
    		lineColor = m_vLineSegments[i].color;
    		glColor4ub(lineColor.r, lineColor.g, lineColor.b, lineColor.a);
    		//pass line segment end-points
    		v2fLinePoint1 = m_vPoints[m_vLineSegments[i].nPoint1].v2fPoint;
    		v2fLinePoint2 = m_vPoints[m_vLineSegments[i].nPoint2].v2fPoint;
    		glVertex2f(v2fLinePoint1.x, v2fLinePoint1.y);
    		glVertex2f(v2fLinePoint2.x, v2fLinePoint2.y);
    	}
    	glEnd();
     
    	glEndList();
     
    	m_bIsDisplayListCurrent = true;
    }
```