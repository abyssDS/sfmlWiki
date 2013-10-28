#2D Vector Manipulation and Transformation

###Overview

This is a vector class that includes more advanced functions for manipulating and transforming two-dimensional vectors.

####vector2.h

Note: the modulo operator (`%`) has been repurposed for rotating the vector. 

```cpp
#pragma once

/* Includes - STL */
#include <string>

/* Includes - SFML */
#include <SFML/System/Vector2.hpp>

/* Defines */
#define PI 3.14159265358979323846F

class VECTOR2
{
public:
    /* The x and y values of this vector */
    float x, y;

    VECTOR2(void);
    VECTOR2(float, float);
    ~VECTOR2(void);

    /*
     * Add and assign operator
     * @VECTOR2		Vector to add against
     * ->VECTOR2	this
     */
    VECTOR2& operator+=(const VECTOR2);

    /*
     * Add operator
     * @VECTOR2		Vector for adding against
     * ->VECTOR2	Result
     */
    VECTOR2 operator+(const VECTOR2) const;

    /*
     * Subtract and assign operator
     * @VECTOR2		Vector to subtract against
     * ->VECTOR2	this
     */
    VECTOR2& operator-=(const VECTOR2);

    /*
     * Subtract operator
     * @VECTOR2		Vector for subtracting against
     * ->VECTOR2	Result
     */
    VECTOR2 operator-(const VECTOR2) const;

    /* 
     * Multiply and assign operator
     * @float		Scalar for multiplying against
     * ->VECTOR2	this
     */
    VECTOR2& operator*=(float);

    /*
     * Multiply operator
     * @float		Scalar for multiplying against
     * ->VECTOR2	Result
     */
    VECTOR2 operator*(float) const;

    /*
     * Divide and assign operator
     * @float		Scalar for dividing against
     * ->VECTOR2	this
     */
    VECTOR2& operator/=(float);

    /*
     * Divide operator
     * @float		Scalar for dividing against
     * ->VECTOR2	Result
     */
    VECTOR2 operator/(float) const;

    /*
     * Rotate and assign operator
     * @float		Theta for rotating against
     * ->VECTOR2	this
     */
    VECTOR2& operator%=(float);

    /*
     * Rotate operator
     * @float		Theta for rotating against
     * ->VECTOR2	Result
     */
    VECTOR2 operator%(float) const;

    /*
     * Equals operator
     * @VECTOR2		Vector for comparing against
     * ->bool		Result
     */
    bool operator==(const VECTOR2);

    /*
     * Not equals operator
     * @VECTOR2		Vector for comparing against
     * ->bool		Result
     */
    bool operator!=(const VECTOR2);
	
    /*
     * The dot product of this and another vector
     * @VECTOR2		Vector for comparing against
     * ->float		The dot product
     */
    float dot(const VECTOR2);

    /*
     * The normalization of this vector
     * ->VECTOR2	A new vector that equals this vector normalized
     */
    VECTOR2 norm();

    /*
     * Truncates this vector
     * @float		Value to truncate by 
     * ->VECTOR2		this
     */
    VECTOR2& trunc(float);

    /*
     * The angle of this vector
     * ->float		This vector's angle in radians
     */
    float ang();

    /*
     * The magnitude of this vector
     * ->float		This vector's magnitude
     */
    float mag();
	
    /*
     * Reassigns this vector's values to the specified values
     * @float		The new x value
     * @float		The new y value
     */
    void reassign(float, float);

    /*
     * Reassigns this vector's values to the vector
     * @VECTOR2		The new vector
     */
    void reassign(VECTOR2);

    /*
     * Converts this vector to a SFML vector
     * -> sf::Vector2f	The new vector
     */
    const sf::Vector2f sf() { return sf::Vector2f(this->x, this->y); }

    /*
     * A string representation of this vector
     * ->std::string	This vector to a string
     */
    const std::string to_str();
};
```

####vector2.cpp

```cpp
#include "vector2.h"

/* Includes - STL */
#include <cstdio>
#include <cmath>

VECTOR2::VECTOR2(void): x(0), y(0)
{
}

VECTOR2::VECTOR2(float x, float y): x(x), y(y)
{
}

VECTOR2::~VECTOR2(void)
{
}

void VECTOR2::reassign(float x, float y)
{
    this->x = x;
    this->y = y;
}

void VECTOR2::reassign(VECTOR2 otr)
{
    reassign(otr.x, otr.y);
}

VECTOR2& VECTOR2::operator+=(const VECTOR2 otr)
{
    this->x += otr.x;
    this->y += otr.y;
    return *this;
}

VECTOR2& VECTOR2::operator-=(const VECTOR2 otr)
{
    this->x -= otr.x;
    this->y -= otr.y;
    return *this;
}

VECTOR2& VECTOR2::operator*=(float scl)
{
    this->x *= scl;
    this->y *= scl;
    return *this;
}

VECTOR2& VECTOR2::operator/=(float scl)
{
    this->x /= scl;
    this->y /= scl;
    return *this;
}

VECTOR2& VECTOR2::operator%=(float theta)
{
    float cs = std::cos(theta);
    float sn = std::sin(theta);

    float px = x * cs - y * sn; 
    float py = x * sn + y * cs;

    this->x = px;
    this->y = py;
    return *this;
}

VECTOR2 VECTOR2::operator+(const VECTOR2 otr) const
{
    return VECTOR2(this->x + otr.x, this->y + otr.y);
}

VECTOR2 VECTOR2::operator-(const VECTOR2 otr) const
{
    return VECTOR2(this->x - otr.x, this->y - otr.y);
}

VECTOR2 VECTOR2::operator*(float scl) const
{
    return VECTOR2(this->x * scl, this->y * scl);
}

VECTOR2 VECTOR2::operator/(float scl) const
{
    return VECTOR2(this->x / scl, this->y / scl);
}

VECTOR2 VECTOR2::operator%(float theta) const
{
    float cs = std::cos(theta);
    float sn = std::sin(theta);

    float px = x * cs - y * sn; 
    float py = x * sn + y * cs;

    return VECTOR2(px, py);
}

bool VECTOR2::operator==(const VECTOR2 otr)
{
    return this->x == otr.x && this->y == otr.y;
}

bool VECTOR2::operator!=(const VECTOR2 otr)
{
    return !(*this == otr);
}

float VECTOR2::dot(const VECTOR2 otr)
{
    return this->x * otr.x + this->y * otr.y; 
}

VECTOR2 VECTOR2::norm()
{
    return *this / this->mag();
}

VECTOR2& VECTOR2::trunc(float max) 
{
    if (this->mag() > max)
        this->reassign(this->norm() * max);
    return *this;
}

float VECTOR2::ang()
{
    return std::atan2(this->x, this->y);
}

float VECTOR2::mag()
{
    return std::sqrt(std::pow(this->x, 2) + std::pow(this->y, 2));
}

const std::string VECTOR2::to_str() 
{
    char tmpbuf[256];
    sprintf(tmpbuf, "[%f, %f] \n", x, y);
    return tmpbuf;
}
```