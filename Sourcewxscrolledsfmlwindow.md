# wxScrolledSFMLWindow

This is a class that allows you to integrate SFML into wxWidgets inside a scrolled window.  This code was based off one of the SFML tutorials [here](http://www.sfml-dev.org/tutorials/1.6/graphics-wxwidgets.php).

However the tutorial is outdated (in terms of wxGTK) and did not work on my system so I fixed it.  My code assumes you are using wxWidgets 2.9.x however, but the only change to get it to work on 2.8.x may only be changing wxScrolledCanvas to wxScrolledWindow.

To use it just derive a class from wxScrolledSFMLWindow and override the OnUpdate function to do your drawing. Display() is called automatically at the end of the draw loop. Note that the window does not refresh automatically and if you want it to refresh you will need to make a call to Refresh.  Or you can do as the tutorial suggests and handle an IdleEvent to refresh the window.

wxScrolledSFMLWindow.hpp

    #ifndef wxScrolledSFMLWindow_HPP
    #define wxScrolledSFMLWindow_HPP

    ////////////////////////////////////////////////////////////
    // Headers
    ////////////////////////////////////////////////////////////
    #include <SFML/Graphics.hpp>
    #include <wx/wx.h>
    #include <wx/scrolwin.h>


    ////////////////////////////////////////////////////////////
    /// wxSFMLWindow allows to run SFML in a wxWidgets control
    ////////////////////////////////////////////////////////////
    class wxScrolledSFMLWindow : public wxScrolledCanvas, public sf::RenderWindow
    {
        public :

            ////////////////////////////////////////////////////////////
            /// Construct the wxSFMLWindow
            ///
            /// \param Parent :   Parent of the control (NULL by default)
            /// \param Id :       Identifier of the control (-1 by default)
            /// \param Position : Position of the control (wxDefaultPosition by default)
            /// \param Size :     Size of the control (wxDefaultSize by default)
            /// \param Style :    Style of the control (0 by default)
            ///
            ////////////////////////////////////////////////////////////
            wxScrolledSFMLWindow(wxWindow* Parent = NULL, wxWindowID Id = -1, const wxPoint& Position = wxDefaultPosition, const wxSize& Size = wxDefaultSize, long Style = 0);

            ////////////////////////////////////////////////////////////
            /// Destructor
            ///
            ////////////////////////////////////////////////////////////
            virtual ~wxScrolledSFMLWindow();

            ////////////////////////////////////////////////////////////
            /// Override to stop wxWidgets default scroll behavior
            ///
            ////////////////////////////////////////////////////////////
            virtual void ScrollWindow(int dx, int dy, const wxRect* rect = NULL);

        private :

            DECLARE_EVENT_TABLE()

            ////////////////////////////////////////////////////////////
            /// Notification for the derived class that moment is good
            /// for doing its update and drawing stuff
            ///
            ////////////////////////////////////////////////////////////
            virtual void OnUpdate();

            ////////////////////////////////////////////////////////////
            /// Called when the window is repainted - we can display our
            /// SFML window
            ///
            ////////////////////////////////////////////////////////////
            void OnPaint(wxPaintEvent&);

            ////////////////////////////////////////////////////////////
            /// Called when the window is resized
            ///
            ////////////////////////////////////////////////////////////
            void OnResize(wxSizeEvent&);

            ////////////////////////////////////////////////////////////
            /// Called when the window needs to draw its background
            ///
            ////////////////////////////////////////////////////////////
            void OnEraseBackground(wxEraseEvent&);

            ////////////////////////////////////////////////////////////
            /// Updates the view on our control override to change
            /// behavior
            ///
            ////////////////////////////////////////////////////////////
            virtual void UpdateScroll();

            bool recreate_flag;
    };


    #endif // wxScrolledSFMLWindow_HPP

wxScrolledSFMLWindow.cpp

    ////////////////////////////////////////////////////////////
    // Headers
    ////////////////////////////////////////////////////////////
    #include "wxScrolledSFMLWindow.hpp"

    // Platform-specific includes
    #ifdef __WXGTK__
    #include <gdk/gdkx.h>
    #include <gtk/gtk.h>
    //#include <wx/gtk/win_gtk.h>

    #include <gdk/gdkprivate.h>
    #include <gtk/gtkwidget.h>
    #endif


    ////////////////////////////////////////////////////////////
    // Event table
    ////////////////////////////////////////////////////////////
    BEGIN_EVENT_TABLE(wxScrolledSFMLWindow, wxScrolledCanvas)
        EVT_ERASE_BACKGROUND(wxScrolledSFMLWindow::OnEraseBackground)
        EVT_PAINT(wxScrolledSFMLWindow::OnPaint)
        EVT_SIZE(wxScrolledSFMLWindow::OnResize)
    END_EVENT_TABLE()


    ////////////////////////////////////////////////////////////
    /// Construct the wxScrolledSFMLWindow
    ////////////////////////////////////////////////////////////
    wxScrolledSFMLWindow::wxScrolledSFMLWindow(wxWindow* Parent, wxWindowID Id, const wxPoint& Position, const wxSize& Size, long Style) :
    wxScrolledCanvas(Parent, Id, Position, Size, Style)
    {
        recreate_flag = false;
        #ifdef __WXGTK__
            // GTK implementation requires to go deeper to find the low-level X11 identifier of the widget
            gtk_widget_realize(m_wxwindow);
            gtk_widget_set_double_buffered(m_wxwindow, false);
            GdkWindow* Win = gtk_widget_get_window(m_wxwindow);//GTK_PIZZA(m_wxwindow)->bin_window;
            XFlush(GDK_WINDOW_XDISPLAY(Win));
            sf::RenderWindow::Create(GDK_WINDOW_XWINDOW(Win));
        #else
            // Tested under Windows XP only (should work with X11 and other Windows versions - no idea about MacOS)
            sf::RenderWindow::Create(GetHandle());
        #endif
    }


    ////////////////////////////////////////////////////////////
    /// Destructor
    ////////////////////////////////////////////////////////////
    wxScrolledSFMLWindow::~wxScrolledSFMLWindow()
    {
        // Nothing to do...
    }


    ////////////////////////////////////////////////////////////
    /// Notification for the derived class that moment is good
    /// for doing its update and drawing stuff
    ////////////////////////////////////////////////////////////
    void wxScrolledSFMLWindow::OnUpdate()
    {

    }

    ////////////////////////////////////////////////////////////
    /// Called when the window is repainted - we can display our
    /// SFML window
    ///
    ////////////////////////////////////////////////////////////
    void wxScrolledSFMLWindow::OnPaint(wxPaintEvent& event)
    {
        wxPaintDC dc(this);

    #ifdef __WXGTK__
        if (recreate_flag)
        {
            gtk_widget_realize(m_wxwindow);
            gtk_widget_set_double_buffered(m_wxwindow, false);
            GdkWindow* Win = gtk_widget_get_window(m_wxwindow);
            XFlush(GDK_WINDOW_XDISPLAY(Win));
            sf::RenderWindow::Create(GDK_WINDOW_XWINDOW(Win));
            recreate_flag = false;
        }
    #endif

        // Set Active
        SetActive(true);

        // Update the View
        UpdateScroll();

        // Let the derived class do its specific stuff
        OnUpdate();

        // Display on screen
        Display();

    }

    ////////////////////////////////////////////////////////////
    /// Called when the control needs to draw its background
    ////////////////////////////////////////////////////////////
    void wxScrolledSFMLWindow::OnEraseBackground(wxEraseEvent& event)
    {
        // Don't do anything. We intercept this event in order to prevent the
        // parent class to draw the background before repainting the window,
        // which would cause some flickering
    }

    ////////////////////////////////////////////////////////////
    /// Updates the view on our control override to change
    /// behavior
    ///
    ////////////////////////////////////////////////////////////
    void wxScrolledSFMLWindow::UpdateScroll()
    {
        int sx, sy, sw, sh;
        CalcUnscrolledPosition(0, 0, &sx, &sy);
        GetClientSize(&sw, &sh);
        sf::View view = GetView();
        view.Reset(sf::FloatRect(sx, sy, sw, sh));
        SetView(view);
    }

    ////////////////////////////////////////////////////////////
    /// Overriden since we don't want to physically scroll
    /// the window contents just the contents our SFMLView
    /// is displaying.
    ////////////////////////////////////////////////////////////
    void wxScrolledSFMLWindow::ScrollWindow(int dx, int dy, const wxRect* rect)
    {

    }

    ////////////////////////////////////////////////////////////
    /// For GTK we have to recreate the window...
    ////////////////////////////////////////////////////////////
    void wxScrolledSFMLWindow::OnResize(wxSizeEvent& event)
    {
    #ifdef __WXGTK__
        // If I recreate here GTK gives an assertion failure.
        recreate_flag = true;
    #endif
    }
