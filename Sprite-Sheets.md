Hey, so these are tested, but still under construction. Use the third constructor in MovableSprite. Just input your sprite sheet and frames per second. Next if you wish add an animation. Just a name and range and then set animation with just the name.

**AutoSprites.cpp**
`
#include <SFML/Graphics.hpp>
#include <sstream>
#include <iostream>



using namespace std;

sf::Clock fpsTimer;
float timeElapsed = 0;
namespace AutoSprites{
    
    class SpriteAnimation{
        public:
        unsigned short int startX = 0;
        unsigned short int startY = 0;
        unsigned short int endX = 0;
        unsigned short int endY = 0;
        unsigned short int fps = 10;
    
        SpriteAnimation();
        SpriteAnimation(unsigned short int x1, unsigned short int y1, unsigned short int x2, unsigned short int y2, unsigned short int frames );
    };
    
    SpriteAnimation::SpriteAnimation(){
        
    }
    
    SpriteAnimation::SpriteAnimation(unsigned short int x1, unsigned short int y1, unsigned short int x2, unsigned short int y2, unsigned short int frames){
        
        startX = x1;
        startY = y1;
        endX = x2;
        endY = y2;
        fps = frames;
        
    }
  
    class MovingSprite{
    private:
        sf::Sprite* AutoSprite;
        unsigned short int totalX = 0;
        unsigned short int totalY = 0;
        
        unsigned short int currentX = 1;
        unsigned short int currentY = 1;
        
        unsigned short int mode = 0;
        
        unsigned short int width = 0;
        unsigned short int height = 0;
        
        unsigned short int tileWidth = 0;
        unsigned short int tileHeight = 0;
        unsigned short int fps = 5;
        sf::IntRect subRect;
        map<string, SpriteAnimation> animations;
        SpriteAnimation currentAnimation;
        
    public:
       
         
        MovingSprite();
        MovingSprite(string path, char switchMode, unsigned short int x, unsigned short int y);
        MovingSprite(sf::Sprite sprt, char switchMode, unsigned short int x, unsigned short int y);
        
        MovingSprite(string path, unsigned short int x, unsigned short int y);
        MovingSprite(sf::Sprite sprt, unsigned short int x, unsigned short int y);
        
        void DrawNext(sf::RenderWindow* win);
        void AddAnimation(std::string name, unsigned short int startX, unsigned short int startY, unsigned short int endX, unsigned short int endY, unsigned short int fps );
        bool SetAnimation(std::string name);
        
    };
    
    bool MovingSprite::SetAnimation(std::string name){
        currentAnimation = animations.find(name)->second;
        currentX = currentAnimation.startX;
        currentY = currentAnimation.startY;
        
        subRect.left = (width / totalX) * (currentX - 1);
        subRect.top = (height / totalY) * (currentY - 1);
        AutoSprite->setTextureRect(subRect);
        std::cout << "LOLX: " << subRect.left << "  LOLY: " << subRect.top;
              return true;
    }
    
    void MovingSprite::AddAnimation(std::string name, unsigned short int startX, unsigned short int startY, unsigned short int endX, unsigned short int endY, unsigned short int fps ){
        
        animations[name] = *new SpriteAnimation(startX, startY, endX, endY, fps);
        
       
    }
   
    
    void MovingSprite::DrawNext(sf::RenderWindow* win){
     
        
        fpsTimer.restart();
        
       
     //   printf("TimeElapsed: %f \n", timeElapsed);
        if (timeElapsed > (1.0 / (currentAnimation.fps))){
            timeElapsed = 0;
           // std::cout << "NEW";
            
        currentX++;
            if (currentX > currentAnimation.endX){
                if (currentY < currentAnimation.endY){
                    currentX = 1;
                    currentY++;
                    subRect.top += height / totalY;
                    AutoSprite->setTextureRect(subRect);
                }
                else {
                    std::cout << "LOL I ENfffDED IN " << currentAnimation.endX << std::endl;
                    currentX = currentAnimation.startX;
                    currentY = currentAnimation.startY;
                    
                    subRect.left = (width / totalX) * (currentX-1);
                    subRect.top = (height / totalY) * (currentY-1);
                    
                    AutoSprite->setTextureRect(subRect);
                    
                    
                }
            }
        
        subRect.left += width / totalX;
         //   printf("LOl: %d", &subRect.left);
        if (subRect.left == width || subRect.left > width){
            subRect.left = 0;
        }
        
       // printf("Subrect left : %d , Subrect top: %d \n", subRect.left, subRect.top);
          
                        
            AutoSprite->setTextureRect(subRect);
            
            
        }
       
      /*  if (currentX > currentAnimation.endX || currentY > currentAnimation.endY){
            std::cout << "ENDED";
            currentX = currentAnimation.startX;
            currentY = currentAnimation.startY;
            
            subRect.left = (width / totalX) * (currentX - 1);
            subRect.top = (height / totalY) * (currentY - 1);
            AutoSprite->setTextureRect(subRect);
        }
*/
        std::cout << "CURRENTX: " << currentX << "  CURRENTY: " << currentY << std::endl;
        win->draw(*AutoSprite);
            timeElapsed += fpsTimer.getElapsedTime().asSeconds();
            
                   
        
       
       
       
    }
    
    MovingSprite::MovingSprite(string path, char switchMode, unsigned short int x, unsigned short int y){
        totalX = x;
        totalY = y;
        
        sf::Texture tempImg;
        tempImg.loadFromFile(path);
        
        //AutoSprite.setTexture(tempImg);
        
        width = tempImg.getSize().x;
        height = tempImg.getSize().y;
        
        subRect.left = 0;
        subRect.top = 0;
        subRect.width = tempImg.getSize().x;
        subRect.height = tempImg.getSize().y;
        
        switch (switchMode) {
            case 'x':
                mode = 0;
                break;
                
            case 'y':
                mode = 1;
                break;
                
            default:
                mode = 0;
                break;
        }
        
        tileWidth = width / totalX;
        tileHeight = height / totalY;
        
        subRect.width = tileWidth;
        subRect.height = tileHeight;
         AutoSprite->setTextureRect(subRect);
        fpsTimer.restart();
        
        currentAnimation = *new SpriteAnimation(1, 1, totalX, totalY, 5);
    }
    
    MovingSprite::MovingSprite(sf::Sprite sprt, char switchMode, unsigned short int x, unsigned short int y){
        totalX = x;
        totalY = y;
        
       // AutoSprite = sprt;
        
       width = sprt.getTexture()->getSize().x;
        height = sprt.getTexture()->getSize().y;
        
        subRect.left = 0;
        subRect.top = 0;
        subRect.width = sprt.getTexture()->getSize().x;
        subRect.height = sprt.getTexture()->getSize().y;
        
        switch (switchMode) {
            case 'x':
                mode = 0;
                break;
                
            case 'y':
                mode = 1;
                break;
                
            default:
                mode = 0;
                break;
        }

        tileWidth = width / totalX;
        tileHeight = height / totalY;
        
        subRect.width = tileWidth;
        subRect.height = tileHeight;
         //AutoSprite.setTextureRect(subRect);
         fpsTimer.restart();
        
        currentAnimation = *new SpriteAnimation(1, 1, totalX, totalY, 5);
    }
    
    
    
    MovingSprite::MovingSprite(string path, unsigned short int x, unsigned short int y){
        totalX = x;
        totalY = y;
        
        sf::Texture tempImg;
        tempImg.loadFromFile(path);
        
        //AutoSprite.setTexture(tempImg);
        
        width = tempImg.getSize().x;
        height = tempImg.getSize().y;
        
        mode = 0;
        
        subRect.left = 0;
        subRect.top = 0;
        subRect.width = tempImg.getSize().x;
        subRect.height = tempImg.getSize().y;
        
        tileWidth = width / totalX;
        tileHeight = height / totalY;
        
        subRect.width = tileWidth;
        subRect.height = tileHeight;
         //AutoSprite.setTextureRect(subRect);
        fpsTimer.restart();
        currentAnimation = *new SpriteAnimation(1, 1, totalX, totalY, 5);
    }
    
    MovingSprite::MovingSprite(sf::Sprite sprt, unsigned short int x, unsigned short int y){
        totalX = x;
        totalY = y;
        
        AutoSprite = &sprt;
        
        width = sprt.getTexture()->getSize().x;
        height = sprt.getTexture()->getSize().y;
        mode = 0;
        
        
        subRect.left = 0;
        subRect.top = 0;
        
        std::stringstream out;
        out << width;
        std::string w = out.str();
        out <<height;
        std::string h = out.str();
        
       
        
        tileWidth = width / totalX;
        tileHeight = height / totalY;
        
        subRect.width = tileWidth;
        subRect.height = tileHeight;
         AutoSprite->setTextureRect(subRect);
         fpsTimer.restart();
        currentAnimation = *new SpriteAnimation(1, 1, totalX, totalY, 5);
    }
    
    
    
    
    
}



`


**AutoSprites.h**
`//
//  AutoSprites.h
//  Baconator
//
//  Created by Thomas Paniagua on 10/13/12.
//  Copyright (c) 2012 LIA. All rights reserved.
//

#ifndef __Baconator__AutoSprites__
#define __Baconator__AutoSprites__

#include <SFML/Graphics.hpp>

using namespace std;
namespace AutoSprites{
class MovingSprite{
public:
    MovingSprite();
    MovingSprite(string path, char switchMode, unsigned short int x, unsigned short int y);
    MovingSprite(sf::Sprite sprt, char switchMode, unsigned short int x, unsigned short int y);
    
    MovingSprite(string path, unsigned short int x, unsigned short int y);
    MovingSprite(sf::Sprite sprt, unsigned short int x, unsigned short int y);
    
    void DrawNext(sf::RenderWindow* win);
    void AddAnimation(std::string name, unsigned short int startX, unsigned short int startY, unsigned short int endX, unsigned short int endY, unsigned short int fps );
    bool SetAnimation(std::string name);
    
};
}

#endif
`