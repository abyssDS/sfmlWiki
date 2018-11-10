# What Are Spritesheets
Spritesheets, or sometimes known as Texture Atlases, are one big image with all the frames used in your animations. Typically, new programmers or prototypists have dozens if not hundreds of individual pixel-perfect frames in their assets folder. For example images labeled "jump_00.png" all the way through "jump_55.png" for a single jump animation for one character.

# Why Spritesheets
This becomes increasingly difficult to manage and this is where spritesheets take the cake. For more convenience, you can have one whole spritesheet for your entire game packed together. 

![klyde frog](https://img.itch.zone/aW1hZ2UvMTI5NzM4LzU5NjExMC5wbmc=/347x500/FhexJa.png)

# The Design
But how do we animate a sprite like this? SFML sprites have a component called the `[https://www.sfml-dev.org/documentation/2.5.1/classsf_1_1Sprite.php#a3492896fe7b63f58ae022c5b8bec5c98](TextureRect)` of type `[https://www.sfml-dev.org/documentation/2.5.1/classsf_1_1Rect.php](sf::IntRect)` that defines the rendering portion of its texture; similar to how the `sf::View` determines what part of your window is drawn.

What we'll do is write an **Animation ** class that takes in a sprite as its target. Every update frame, it will step through each rectangle and apply it to its target. We can still draw and transform the sprite the normal SFML way. Our class will only be responsible for 

* the list of "frames" or sf::IntRect
* moving through the frames when the time is right
* reflecting changes on the target sprite


***

# Implementation

In "Animation.h"

We need a small data structure for keeping the time between rectangles.

`
struct Frame {
   sf::IntRect rect;
   double duration; // in seconds
};
`

We need the animation class to show the correct rectangle at the right time. So we'll keep track of the progress as well as the total duration.

`
class Animation {
   std::vector<Frame> frames;
   double totalLength;
   double progress;
   sf::Sprite &target;

   public:
   Animation(sf::Sprite &target);
   virtual ~Animation();
   void addFrame(Frame&& frame);
   void update(double elapsed);
};
`

***

In "Animation.cpp"

The constructor binds our sprite and sets everything to default values.

`
     Animation::Animation(sf::Sprite &target) : target(target) { 
       progress = totalLength = 0.0;
     }
`

Our add frame _moves_ the new frame into the list and updates the total length of the animation. 

`
     void Animation::addFrame(Frame&& frame) {
       frames.push_back(std::move(frame)); 
       totalLength += frame.duration; 
     }
`

If you're unfamiliar with C++11 move operations, in brief it is an optimized choice to skip the copy step that C++ would do when calling `frames.push_back(frame)`. 

Finally, on the update step, we tack onto our progress counter and see if we have enough progress points to satisfy the frame's duration. We do that my subtracting from `progress` until it's zero or under. If so, we stop iterating through the list of frames and apply the rect on the sprite.

`
     void Animation::update(double elapsed) {
        progress += elapsed;
        double p = progress;
        for(size_t i = 0; i < frames.size(); i++) {
           p -= frames[i].duration;  

          // if we have progressed OR if we're on the last frame, apply and stop.
          if (p <= 0.0 || &(frames[i]) == &frames.back())
          {
               target.setTextureRect(frames[i].rect);  
               break; // we found our frame
          }
     }
`

You may have noticed the extra conditional expression in our update loop. We simply check the address of the last frame `&frames.back()` against the address of the current frame `&(frames[i])`. If the address is the same, we know we're on the last frame. Our animation class doesn't have any replay modes like looping, rewinding, playing x amount of times; it's a simple animation class but you can easily add these features yourself.


***


And that's it! Now you can animate your spritesheet:

First we load the spritesheet and animation before our main loop

`
sf::Sprite myCharacter;
// Load the image...
// myCharacter.setTexture(*mySpritesheet);

Animation animation(myCharacter);
animation.addFrame({sf::IntRect(x,y,w,h), 0.1});
animation.addFrame({sf::IntRect(x,y,w,h), 0.1});
animation.addFrame({sf::IntRect(x,y,w,h), 0.4});
animation.addFrame({sf::IntRect(x,y,w,h), 0.1});
// do this for as many frames as you need
`

Finally in your main loop:

`
animation.update(elapsed);
window.draw(myCharacter);
`

***

# Post Development

Cool but what if we have multiple animations? Our player character can jump, walk, swim, fight, even die. 💀

That's easy. Create an animation object for each behavior for the same sprite. Then choose which one to update at the right time and the animation class will update the sprite's rect.

`
Animation jumpAnim(myCharacter);
Animation walkAnim(myCharacter);
Animation swimAnim(myCharacter);
Animation deadAnim(myCharacter);

// in update loop...

if     (player.isJumping)  { jumpAnim.update(elapsed); }
else if(player.isWalking)  { walkAnim.update(elapsed); }
else if(player.isSwimming) { swimAnim.update(elapsed); }
else if(player.isDead)     { deadAnim.update(elapsed); }

window.draw(myCharacter);
`

# Looping

If you've followed along this far you'll notice the animation plays once and then stops. 
You can add playback modes to your animation class. This is how you would add looping:

`
// We check if p has some time left at the end of the animation...
if (playbackMode == modes::LOOP && p > 0.0 && &(frames[i]) == &(frames.back())) {
    i = 0;    // start over from the beginning
    continue; // break off the loop and start where i is
}
`

