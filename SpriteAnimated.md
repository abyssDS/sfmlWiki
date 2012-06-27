This is a fairly simple C# class for handling animated sprite in SFML.Net 2.0. It's based upon docflabby's AniSprite code from [here](http://www.sfml-dev.org/wiki/en/sources/anisprite).

### How this works
In this class, all sprite animations are stored in single texture, consisting of frames of similar size.![Sprite example](http://s13.postimage.org/ekgxwqgvr/test.png)

This picture shows an example of frame sequence, stored in one image, where each 64x64 square is one frame. You can also see in which order frames are numered. When your sprite is updated in game loop, it will switch between frames at given rate, using TextureRect property.

### Using this class
First, to make this work, you have to pass to this code time since last frame (delta time). In method Update() find this line: 

`// _clock += Game.Delta; - Here should be your relative time implementation.`

This line assumes that you have static _Game_ class, property of wich is _int Delta_ (time since last frame). Then you add delta to your internal clock, which allows method to decide, should it switch to next frame. You can either implement this in your program, or devise another way to get delta time (for example, pass it as argument to _Update()_ method).

Here is usage example (for usage details see docs):


    Texture SomeTexture = new Texture("path/to/sprite.tga");
    SpriteAnimated SomeSprite = new SpriteAnimated(SomeTexture, 64, 64, 30, (RenderTarget)_YourRenderWindow, RenderStates.Default, 0, 11, true, true);

    // ...
    // Game loop
    while()
    {
        Game.Delta = NewDeltaTime;
        SomeSprite.Update;
    }

### Actual code
    using System;
    using SFML.Window;
    using SFML.Graphics;
    using SFML.Audio;

    namespace SFMLproject
    {
    class SpriteAnimated : Sprite
    {
        RenderTarget _renderTarget;
        RenderStates _renderStates;
        int _fps, _frameWidth, _frameHeight, _frameCount, _currentFrame, _firstFrame, _lastFrame, _interval, _clock;
        bool _isAnimated = false;
        bool _isLooped = true;

        /// <summary>
        /// Create new SpriteAnimated.
        /// </summary>
        /// <param name="Text">Texture to use in your sprite.</param>
        /// <param name="FrameWidth">Width of one frame in pixels.</param>
        /// <param name="FrameHeight">Height of one frame in pixels.</param>
        /// <param name="FramesPerSecond">Your sprite's FPS.</param>
        /// <param name="RTarget">RenderTarget reference.</param>
        /// <param name="RStates">RenderStates object.</param>
        /// <param name="FirstFrame">First frame of animation sequence.</param>
        /// <param name="LastFrame">Last frame of animation sequence.</param>
        /// <param name="IsAnimated">Should sequence be played immediately after creation? If false, first frame will be paused.</param>
        /// <param name="IsLooped">Should sequence be looped? If false, animation will stop after one full sequence.</param>
        public SpriteAnimated(Texture Text, int FrameWidth, int FrameHeight, int FramesPerSecond, RenderTarget RTarget, RenderStates RStates, int FirstFrame = 0,  int LastFrame = 0, bool IsAnimated = false, bool IsLooped = true) : base(Text)
        {
            _renderTarget = RTarget;
            _renderStates = RStates;
            _fps = FramesPerSecond;
            _interval = 1000 / _fps;
            _frameWidth = FrameWidth;
            _frameHeight = FrameHeight;
            _frameCount = LastFrame - FirstFrame;
            _firstFrame = FirstFrame;
            _currentFrame = FirstFrame;
            _lastFrame = LastFrame;
            _isAnimated = IsAnimated;
            _isLooped = IsLooped;
            _clock = 0;
            this.TextureRect = this.GetFramePosition(_currentFrame);
        }

        /// <summary>
        /// This method calculates TextureRect coordinates for a certain frame.
        /// </summary>
        /// <param name="frame">Frame which coordinates you need.</param>
        /// <returns>Returns frame coordinates as IntRect.</returns>
        public IntRect GetFramePosition(int frame)
        {
            int WCount = (int)this.Texture.Size.X / _frameWidth;
            int XPos = frame % WCount;
            int YPos = frame / WCount;

            IntRect Position = new IntRect(_frameWidth * XPos, _frameHeight * YPos, _frameWidth, _frameHeight);
            return Position;
        }
 
        /// <summary>
        /// This method is used to update animation state and draw sprite.
        /// You should avoid using Draw() method if you use this.
        /// </summary>
        public void Update()
        {
            _clock += Game.Delta;
            if (_isAnimated & _clock >= _interval)
            {
                this.TextureRect = this.GetFramePosition(_currentFrame);
                if (_currentFrame < _lastFrame)
                    _currentFrame++;
                else
                    _currentFrame = _firstFrame;
                _clock = 0;
            }

            //if (!_isLooped & (_currentFrame == _lastFrame))
            //{
            //     _isAnimated = false;
            //}

            this.Draw(_renderTarget, _renderStates);
        }

        /// <summary>
        /// Get or set current framerate.
        /// </summary>
        public int FPS
        {
            set
            {
                _fps = value;
                _interval = _fps / 1000;
            }
            get
            {
                return _fps;
            }
        }

        /// <summary>
        /// Resume animation with current settings.
        /// </summary>
        public void Play()
        {
            _isAnimated = true;
        }

        /// <summary>
        /// Pause current animation.
        /// </summary>
        public void Pause()
        {
            _isAnimated = false;
        }

        /// <summary>
        /// Stop animation and return to frame zero.
        /// </summary>
        public void Reset()
        {
            _isAnimated = false;
            _currentFrame = 0;
            this.TextureRect = new IntRect(0, 0, _frameWidth, _frameHeight);
        }

        /// <summary>
        /// Set new animation sequence.
        /// </summary>
        /// <param name="FirstFrame">First frame of new sequence.</param>
        /// <param name="LastFrame">Last frame of new sequence.</param>
        /// <param name="IsAnimated">Should sequence be played immediately? If false, first sequence frame will be paused.</param>
        /// <param name="IsLooped">Should sequence be looped? If false, animation will stop after one full sequence.</param>
        public void SetAnimation(int FirstFrame, int LastFrame, bool IsAnimated = true, bool IsLooped = true)
        {
            _firstFrame = FirstFrame;
            _lastFrame = LastFrame;
            _isAnimated = IsAnimated;
            _isLooped = IsLooped;

            _frameCount = (LastFrame + 1) - FirstFrame;
            if (!IsAnimated)
            {
                this.TextureRect = this.GetFramePosition(FirstFrame);
            }
        }
    }
    }