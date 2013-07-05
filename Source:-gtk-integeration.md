```c#
//  Copyright (c) 2013 Camilo Polymeris
//
// This software is provided 'as-is', without any express or implied warranty.
// In no event will the authors be held liable for any damages arising from
// the use of this software.

// Permission is granted to anyone to use this software for any purpose,
// including commercial applications, and to alter it and redistribute it
// freely, subject to the following restrictions:

// 1. The origin of this software must not be misrepresented; you must not claim
//    that you wrote the original software. If you use this software in a product,
//    an acknowledgment in the product documentation would be appreciated but is
//    not required.
// 
// 2. Altered source versions must be plainly marked as such, and must not be
//    misrepresented as being the original software.
// 
// 3. This notice may not be removed or altered from any source distribution.

using System;
using Gtk;

using SFML.Graphics;
using SFML.Window;

namespace Pax.Client
{
	public abstract class DrawingArea : Gtk.DrawingArea, RenderTarget
	{
		protected abstract void Redraw();

		private RenderWindow window;
		private System.Timers.Timer timer;

		public DrawingArea(float viewScale = 100f, int minimumWidth, int minimumHeight)
		{
			DoubleBuffered = false;
			CanFocus = true;
			Events = Gdk.EventMask.AllEventsMask;
			SetSizeRequest(minimumWidth, minimumHeight);
			timer = new System.Timers.Timer(25);
			timer.Elapsed += (o, e) => OnTimerElapsed();
			ButtonPressEvent += OnButtonPressed;

			Application.Invoke(
				delegate
			{
				Show();
				ContextSettings settings = new ContextSettings();
				settings.AntialiasingLevel = 8;
				window = new RenderWindow(GetDrawingAreaHandle(), settings);
				ConfigureEvent += (o, args) => SetSize(args.Event.Width, args.Event.Height);
				Window.SetVerticalSyncEnabled(true);
				timer.Enabled = true;
			});
		}

		public void SetSize(float w, float h)
		{
			float sW = (w > h ? w / h : 1f) * viewScale;
			float sH = (h > w ? h / w : 1f) * viewScale;
			Window.SetView(new View(new Vector2f(), new Vector2f(sW, sH)));
		}

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

		void OnButtonPressed(object sender, ButtonPressEventArgs args)
		{
			Vector2i pixel = new Vector2i((int)args.Event.X, (int)args.Event.Y);
			Vector2f coords = MapPixelToCoords(pixel);
		}

		protected RenderWindow Window
		{
			get { return window; }
		}

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
			get
			{
				return Window.Size;
			}
		}

		public View DefaultView
		{
			get { return Window.DefaultView; }
		}

		protected override bool OnExposeEvent(Gdk.EventExpose ev)
		{
			base.OnExposeEvent(ev);
			Window.Display();
			return true;
		}

		private IntPtr GetDrawingAreaHandle()
		{
			int p = (int)Environment.OSVersion.Platform;
			if ((p == 4) || (p == 6) || (p == 128)) //<Unix
				return gdk_x11_drawable_get_xid(GdkWindow.Handle);
			else //<Windows
				return gdk_win32_drawable_get_handle(GdkWindow.Handle);
		}
		
		[System.Runtime.InteropServices.DllImport("libgdk-win32-2.0-0.dll")]
		private static extern IntPtr gdk_win32_drawable_get_handle(IntPtr window);
		
		[System.Runtime.InteropServices.DllImport("gdk-x11-2.0")]
		private static extern IntPtr gdk_x11_drawable_get_xid(IntPtr window);
	}
}
```