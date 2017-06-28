

How to Integrate SFML 2.3.2 to a Qt 5.5.1 interface, based on [this discussion](http://en.sfml-dev.org/forums/index.php?topic=19163.msg138575#msg138575).

How it worked in SFML 1.6
-------------------------

[A description of this.](http://www.sfml-dev.org/tutorials/1.6/graphics-qt.php)

Making it work in SFML 2.3.2
----------------------------

### Changed methods

The methods `Create` and `Display` were changed to lowercase in newer versions of SFML. Hence these changes must be applied to the code in the tutorial linked above. However, it seems like QWidget does also provide the method `create`, thus we have to be more precise and write `sf::Window::create`. According to [this post](http://en.sfml-dev.org/forums/index.php?topic=13851.msg97124#msg97124), the argument in the *create*-statement should be casted to `sf::WindowHandle`, so the line with the *create*-statement ultimately becomes

    sf::RenderWindow::create((sf::WindowHandle) winId());

### Virtual methods
 
The derived class `QSFMLCanvas` inherits some virtual methods which have to be implemented. Adding

    QSFMLCanvas::~QSFMLCanvas() {}
    void QSFMLCanvas::OnInit() {}
    void QSFMLCanvas::OnUpdate() {}

to the class definition should do it. (I think `QSFMLCanvas` is supposed to be virtual, so it might be better to declare those as virtual instead of dummy-implementing them.)

Everything should be working now.

### Resizing

When `QWidget::resize()` is called, the RenderWindow does not match the QWidget anymore. A solution to this (but maybe not the best one) is to call `sf::RenderWindow::create((sf::WindowHandle) winId());` once again subsequently.

### A better method for resizing

When `QWidget::resize()` is called, it sends a `QResizeEvent` event. This event can be captured using `QWidget::resizeEvent()`. Whenever QWidget resizes, the size of the RenderWindow can be set using `setSize()` method of the `RenderWindow` class and passing `QWidget::width()` and `QWidget::height()` as the arguments . Thus the RenderWindow can now resize with QWidget.

    void QSFMLCanvas::resizeEvent(QResizeEvent* event)
    {
        setSize(sf::Vector2u(QWidget::width(), QWidget::height()));
    }