## Links

- [Forum Thread](https://en.sfml-dev.org/forums/index.php?topic=23578.0) (by Foaly)
- Longer discuss on [GitHub issue #744](https://github.com/SFML/SFML/issues/744)
- [Reference implementation](https://github.com/Foaly/SFML/tree/feature/window_state) (macOS & Windows) by Foaly

## Current Situation

Currently SFML (2.5) doesn't strictly differentiate between window states and window styles. The following APIs exist for manipulating window's styles and states:

- Constructor / `create(...)` function that takes a bit field of `sf::Style`
  - `sf::Style::None` - Borderless window
  - `sf::Style::Titlebar`- Window with titlebar
  - `sf::Style::Resize` - Resizable window
  - `sf::Style::Close` - Window with close button
  - `sf::Style::Fullscreen` - Fullscreen mode
  - `sf::Style::Default` - Combination of Titlebar, Resize and Close
- `setVisible(bool)` - Hide or show window

The limitations of this design are:

- Mixing of window styling (borderless, titlebar, close button, etc.) and window state (windowed, fullscreen, hidden, etc.)
- Window has to be recreated for changing a style or switching between window and fullscreen mode
- No support to programmatically maximize or minimize a window
- The constructor parameter list is relatively long, because the window is instantly shown at creation, instead of letting the user configure the window with dedicated functions before showing it

## Target Situation

### API

- Constructor / `create(...)` function that takes a bit field of `sf::Style`
  - `sf::Style::None` - Borderless window
  - `sf::Style::Titlebar`- Window with titlebar
  - `sf::Style::Resize` - Resizable window
  - `sf::Style::Close` - Window with close button
  - `sf::Style::Fullscreen` - **(deprecated)** Fullscreen mode
  - `sf::Style::Default` - Combination of Titlebar, Resize and Close
  - `sf::Style::Hidden` - **new** Create window hidden initially
- `setState(sf::State)` - **new** Change window state
  - `sf::State::Minimized` - Minimize the window
  - `sf::State::Maximized` - Maximize the window
  - `sf::State::Floating` (or `sf::State::Windowed`) - "Normal" window mode
  - `sf::State::Fullscreen` - Fullscreen mode
- `setVisible(bool)` - Hide or show window

### OS Support

| Feature                           | Windows | Unix | macOS |
|-----------------------------------|---------|------|-------|
| Initially hidden window           |         |      |       |
| Minimize                          |         |      |       |
| Maximize                          |         |      |       |
| Mode switching without recreation |         |      |       |
| Titlebar                          | x       | x    | x     |
| Borderless                        | x       | x    | x     |
| Resizable                         | x       | x    | x     |
| Close button                      | x       | x    | x     |
| Minimize button                   |         |      |       |
| Maximize button                   |         |      |       |