This class works as a "bridge" between GTK and SFML: It acts both as a GTK Widget, i.e. it can be put into TopLevelWindows or other containers, and as a SFML RenderTarget, that is, you can draw to it. I find it easiest to derive a class from this for each game scene. Use SFML's event system to handle clicks and key presses. The code below includes a sample mouse press event.

Note that there are [other ways](Source:-GTK-SFMLWidget) to use SFML with GTK (e.g passing a DrawingArea's Handle to the SFML Window constructor), but this is the best, IMO, and the only one that I have found that works with AA under Linux. (Since SFML 2.1)

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

namespace Pax.Client
{
	public abstract class SFMLWidget : RenderWindow
	{
		protected abstract void Redraw();

		private Gtk.Socket widget;
		private System.Timers.Timer timer;
		private float scale;

		
		public SFMLWidget(float scale): this(new ContextSettings(8, 8, 8), scale)
		{
		}

		public SFMLWidget(): this(new ContextSettings(8, 8, 8))
		{
		}

		public SFMLWidget(ContextSettings settings,
		                  float scale = 1000f, uint minWidth = 640, uint minHeight = 480) :
			base(new VideoMode(minWidth, minHeight), "SFML Window", Styles.None, settings)
		{
			this.scale = scale;

			timer = new System.Timers.Timer(25);
			timer.Elapsed += (o, e) => OnTimerElapsed();

			widget = new Gtk.Socket();
			widget.SetSizeRequest((int)minWidth, (int)minHeight);

			Gtk.Application.Invoke((o, e) => ShowSFMLWindow());
		}

		/// <summary>
		/// This must be called in the main thread. Creates the SFML window and "steals" it.
		/// </summary>
		private void ShowSFMLWindow()
		{
			widget.AddId((uint)SystemHandle);
			Resized += (o, e) => OnResized(e);

			MouseButtonPressed += (o, e) => OnButtonPressed(o, e);

			// use a timer to periodically redraw the window
			timer.Enabled = true;
			widget.Show();
		}

		public void SetSize(float w, float h)
		{
			float sW = (w > h ? w / h : 1f) * scale;
			float sH = (h > w ? h / w : 1f) * scale;
			SetView(new View(new Vector2f(), new Vector2f(sW, sH)));
		}

		/// <summary>
		/// Timer handler. Since GTK controlls the main loop we need this to poll SFML events.
		/// </summary>
		private void OnTimerElapsed()
		{
			Gtk.Application.Invoke(
				delegate
			{
				DispatchEvents();
				Redraw();
				Display();
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

		public static implicit operator Gtk.Widget(SFMLWidget w)
		{
			return w.widget;
		}
	}
}
```