This class works as a "bridge" between GTK and SFML: It acts both as a GTK Widget, i.e. it can be put into TopLevelWindows or other containers, and as a SFML RenderTarget, that is, you can draw to it. I find it easiest to derive a class from this for each game scene. Use SFML's event system to handle clicks and key presses.

Note that there are other ways to use SFML with GTK (e.g passing a DrawingArea's Handle to the SFML Window constructor), but this is the best, IMO, and the only one that I have found that works with AA under Linux. (Since SFML 2.1)

```c#
//
//  SFMLWidget.cs
//
//  Author:
//       Camilo Polymeris <cpolymeris@gmail.com>
//
//  Copyright (c) 2013 Camilo Polymeris
//
//  This program is free software: you can redistribute it and/or modify
//  it under the terms of the GNU General Public License as published by
//  the Free Software Foundation, either version 3 of the License, or
//  (at your option) any later version.
//
//  This program is distributed in the hope that it will be useful,
//  but WITHOUT ANY WARRANTY; without even the implied warranty of
//  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
//  GNU General Public License for more details.
//
//  You should have received a copy of the GNU General Public License
//  along with this program.  If not, see <http://www.gnu.org/licenses/>.

using System;
using SFML.Graphics;
using SFML.Window;

namespace <Your Namespace Here>
{
	public abstract class SFMLWidget : Gtk.Socket, RenderTarget
	{
		protected abstract void Redraw();

		private RenderWindow window;
		private System.Timers.Timer timer;
		private float scale;

		public SFMLWidget(float scale = 1000f, int minWidth = 640, int minHeight = 480)
		{
			this.scale = scale;
			timer = new System.Timers.Timer(25);
			timer.Elapsed += (o, e) => OnTimerElapsed();

			SetSizeRequest(minWidth, minHeight);

			Gtk.Application.Invoke((o, e) => CreateSFMLWindow());
		}

		/// <summary>
		/// This must be called in the main thread. Creates the SFML window and "steals" it.
		/// </summary>
		private void CreateSFMLWindow()
		{
			ContextSettings settings = new ContextSettings();
			settings.AntialiasingLevel = 8;
			window = new RenderWindow(new VideoMode(640, 480), "SFML Window", Styles.None, settings);
			AddId((uint)Window.SystemHandle);
			Window.Resized += (o, e) => OnResized(e);

			Window.MouseButtonPressed += (o, e) => OnButtonPressed(o, e);

			// use a timer to periodically redraw the window
			timer.Enabled = true;
			Show();
		}

		public void SetSize(float w, float h)
		{
			float sW = (w > h ? w / h : 1f) * scale;
			float sH = (h > w ? h / w : 1f) * scale;
			Window.SetView(new View(new Vector2f(), new Vector2f(sW, sH)));
		}

		/// <summary>
		/// Timer handler. Since GTK controlls the main loop we need this to poll SFML events.
		/// </summary>
		private void OnTimerElapsed()
		{
			Gtk.Application.Invoke(
				delegate
				{
				Window.DispatchEvents();
				Redraw();
				window.Display();
			});
		}

		void OnResized(SizeEventArgs e)
		{
			Console.WriteLine("{0}, {1}", e.Width, e.Height);
			SetSize(e.Width, e.Height);
		}

		/// <summary>
		/// Sample event handler.
		/// </summary>
		void OnButtonPressed(object sender, MouseButtonEventArgs args)
		{
			Vector2i pixel = new Vector2i((int)args.X, (int)args.Y);
			Vector2f coords = MapPixelToCoords(pixel);
			Console.Out.WriteLine("Button pressed @ {0}, {1} ({2:0.}, {3:0.})",
			                      pixel.X, pixel.Y, coords.X, coords.Y);
		}


		/// Some convenience casts, the first might be unnecessary.
		public static implicit operator RenderWindow(SFMLWidget w)
		{
			return w.window;
		}

		public RenderWindow Window
		{
			get { return window; }
		}

		///// Trivial RenderTarget implementation below
		public View GetView()
		{
			return Window.GetView();
		}

		public void SetView(View view)
		{
			Window.SetView(view);
		}

		public IntRect GetViewport(View view)
		{
			return Window.GetViewport(view);
		}

		public SFML.Window.Vector2f MapPixelToCoords(SFML.Window.Vector2i point)
		{
			return Window.MapPixelToCoords(point);
		}

		public SFML.Window.Vector2f MapPixelToCoords(SFML.Window.Vector2i point, View view)
		{
			return Window.MapPixelToCoords(point, view);
		}

		public SFML.Window.Vector2i MapCoordsToPixel(SFML.Window.Vector2f point)
		{
			return Window.MapCoordsToPixel(point);
		}

		public SFML.Window.Vector2i MapCoordsToPixel(SFML.Window.Vector2f point, View view)
		{
			return Window.MapCoordsToPixel(point, view);
		}

		public void Clear()
		{
			Window.Clear();
		}

		public void Clear(Color color)
		{
			Window.Clear(color);
		}

		public void Draw(Drawable drawable)
		{
			Window.Draw(drawable);
		}

		public void Draw(Drawable drawable, RenderStates states)
		{
			Window.Draw(drawable, states);
		}

		public void Draw(Vertex[] vertices, PrimitiveType type)
		{
			Window.Draw(vertices, type);
		}

		public void Draw(Vertex[] vertices, PrimitiveType type, RenderStates states)
		{
			Window.Draw(vertices, type, states);
		}

		public void Draw(Vertex[] vertices, uint start, uint count, PrimitiveType type)
		{
			Window.Draw(vertices, start, count, type);
		}

		public void Draw(Vertex[] vertices, uint start, uint count, PrimitiveType type, RenderStates states)
		{
			Window.Draw(vertices, start, count, type, states);
		}

		public void PushGLStates()
		{
			Window.PushGLStates();
		}

		public void PopGLStates()
		{
			Window.PopGLStates();
		}

		public void ResetGLStates()
		{
			Window.ResetGLStates();
		}

		public SFML.Window.Vector2u Size
		{
			get	{ return Window.Size; }
		}

		public View DefaultView
		{
			get	{ return Window.DefaultView; }
		}
	}
}

```