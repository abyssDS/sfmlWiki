# Ruby light manager

I decided to do a conversion of the C++ Light manager in the old French wiki. So now it's available for Ruby! I've done some modifications to the general structure to apply a more Ruby-like feel to the whole. There is for instance no handle classes to manipulate light sources or walls. 

### Future plans
Well I want to work more on the class as I really enjoy working with lighting. Some of the plans to add is:

* Follow the target view for rendering.
* Support Z-layers for games that want depth or maybe tiled levels/floors.
* Regenerate affected static light when the environment geometry changes.
* Directional light, both dynamic and static.
* Allow queries to the light data to see if a pixel is under any light. (Maybe add entire shapes?)
* Optimizations, this is done entirely in ruby so I'll probably be benchmarking a lot and do everything I can do to slim it down.

### Example
![The current Light manager in action.](http://www.groogy.se/lightmanager.png)

I added more walls to the example and the frame-rate dropped significantly. The static lights though has a high quality number but don't have an impact at all in the FPS. Will have to sort out what walls are relevant by doing a Circle-Vs-Point collision detection.
```ruby
require 'sfml/all'
require './LightManager.rb'

fps_text = SFML::Text.new
fps_text.position = [0, 0]

image = SFML::Image.new( "background.jpg" )
background = SFML::Sprite.new( image )

window = SFML::RenderWindow.new( [800 , 600], "Performance Test" )

light_manager = LightManager.new( window )
light_manager.debug = false
light_manager.add_wall( SFML::Vector2.new( 100.0, 100.0 ), SFML::Vector2.new( 200.0, 200.0 ) )
light_manager.add_wall( SFML::Vector2.new( 400.0, 400.0 ), SFML::Vector2.new( 300.0, 500.0 ) )
light_manager.add_wall( SFML::Vector2.new( 400.0, 100.0 ), SFML::Vector2.new( 500.0, 0.0 ) )
walls = []
wall = SFML::Shape.line( 0.0, 0.0, 100.0, 100.0, 5, SFML::Color::White )
wall.position = [ 100.0, 100.0 ]
walls << wall

wall = SFML::Shape.line( 0.0, 0.0, -100.0, 100.0, 5, SFML::Color::White )
wall.position = [ 400.0, 400.0 ]
walls << wall

wall = SFML::Shape.line( 0.0, 0.0, 100.0, -100.0, 5, SFML::Color::White )
wall.position = [ 400.0, 100.0 ]
walls << wall

mouse_light = light_manager.add_dynamic( SFML::Vector2.new( 0.0, 0.0 ), 1, 256, 32, SFML::Color::White )
light_manager.add_static( SFML::Vector2.new( 450, 450 ), 1, 150, 128, SFML::Color::Red )
light_manager.add_static( SFML::Vector2.new( 400, 0 ), 1, 180, 128, SFML::Color::Blue )

frame = 0
elapsed_time = 0.0
fps = 0

while true
  while event = window.get_event
    if event.type == SFML::Event::Closed
      exit
    end
  end
  
  mouse_light.position = SFML::Vector2.new( window.input.mouse_x.to_f, window.input.mouse_y.to_f )
  
  light_manager.generate()
  
  window.clear()
  window.draw( background )
  for wall in walls
    window.draw( wall )
  end
  light_manager.render_on( window )
  window.draw( fps_text )
  window.display()
  if elapsed_time >= 1.0
    fps = frame
    frame = 0
    elapsed_time = 0
  end
  elapsed_time += window.frame_time
  frame += 1
  fps_text.string = fps.to_s + " FPS"
end
```

### The code
```ruby
class LightManager

  attr_accessor :debug

  def add_dynamic( position, intensity, radius, quality, color )
    light = LightManager::Source.new( position, intensity, radius, quality, color )
    @dynamic_lights.push( light )
    return light
  end

  def remove_dynamic( light )
    @dynamic_lights.delete( light )
  end

  def remove_all_dynamics()
    @dynamic_lights.clear()
  end

  def add_static( position, intensity, radius, quality, color )
    light = LightManager::Source.new( position, intensity, radius, quality, color )
    @static_lights.push( light )
    light.generate( @walls, @debug )
    return @static_lights.size - 1
  end

  def remove_static( light_handle )
    @static_lights.delete_at( light_handle )
    nil
  end
  
  def regenerate_static( light_handle )
    @static_lights[ light_handle ].generate( @walls, @debug )
    nil
  end
  
  def regenerate_statics()
    for light in @static_lights
      light.generate( @walls, @debug )
    end
  end

  def remove_all_statics()
    @static_lights.clear()
  end

  def remove_all_lights()
    remove_all_dynamics()
    remove_all_statics()
  end

  def add_wall( point1, point2 )
    add_wall = true
    for wall in @walls
      point3 = wall.point1
      point4 = wall.point2

      if( ( point1.y - point2.y ) / ( point1.x - point2.y ) == ( point3.y - point4.y ) / ( point3.x - point4.y ) )
        if( point1 == point3 or point2 == point3 or point1 == point4 || point2 == point4 )
          min = point1.clone
          max = point2.clone

          min.x = point2.x if point2.x < min.x
          max.x = point1.x if point1.x > max.x
          min.x = point3.x if point3.x < min.x
          min.x = point4.x if point4.x < min.x
          max.x = point3.x if point3.x > max.x
          max.x = point4.x if point4.x > max.x

          min.y = point2.y if point2.y < min.y
          max.y = point1.y if point1.y > max.y
          min.y = point3.y if point3.y < min.y
          min.y = point4.y if point4.y < min.y
          max.y = point3.y if point3.y > max.y
          max.y = point4.y if point4.y > max.y

          wall.point1 = min
          wall.point2 = max
          return wall
        end
      end
    end
    wall = LightManager::Wall.new( point1, point2 )
    @walls.push( wall )
    return wall
  end

  def delete_wall( wall )
    @wall.delete( wall )
  end

  def delete_all_walls()
    @wall.clear
  end

  def generate()
    @buffer.clear( @ambience )
    for light in @dynamic_lights
      light.generate( @walls, @debug )
      light.render_on( @buffer )
    end

    for light in @static_lights
      light.render_on( @buffer )
    end

    @buffer.display()
  end

  def render_on( target )
    sprite = SFML::Sprite.new( @buffer.image )
    sprite.blend_mode = SFML::Blend::Multiply
    target.draw( sprite )
  end
  
private
  def initialize( target_window )
    @static_lights = []
    @dynamic_lights = []
    @walls = []
    @light_smooth = 0
    @buffer = SFML::RenderImage.new()
    @buffer.create( target_window.width, target_window.height )
    @ambience = SFML::Color::Black
    @debug = false
  end
  
  class Wall
    attr_accessor :point1
    attr_accessor :point2

    def position=( position )
      buffer = @point1
      @point1 = position.clone
      @point2.x = @point2.x + ( position.x - buffer.x )
      @point2.y = @point2.y + ( position.y - buffer.y )
      @point1
    end
  private
    def initialize( point1, point2 )
      @point1 = point1.clone
      @point2 = point2.clone
    end
  end

  class Source
    attr_accessor :position
    attr_accessor :intensity
    attr_accessor :radius
    attr_accessor :quality
    attr_accessor :color
    
    def render_on( target )
      for shape in @shapes
        target.draw( shape )
      end
    end

    def add_triangle( point1, point2, minimum_wall, walls, debug )
      for index in minimum_wall...(walls.size)
        wall = walls[ index ]
        l1 = SFML::Vector2.new( wall.point1.x - @position.x, wall.point1.y - @position.y )
        l2 = SFML::Vector2.new( wall.point2.x - @position.x, wall.point2.y - @position.y )

        if( l1.x * l1.x + l1.y * l1.y < @radius * @radius )
          i = Collision.intersects( point1, point2, SFML::Vector2.new( 0, 0 ), l1 )
          if( ( ( point1.x > i.x && point2.x < i.x ) || ( point1.x < i.x && point2.x > i.x ) ) &&
              ( ( point1.y > i.y && point2.y < i.y ) || ( point1.y < i.y && point2.y > i.y ) ) &&
              ( l1.y > 0 && i.y > 0 || l1.y < 0 && i.y < 0 ) &&
              ( l1.x > 0 && i.x > 0 || l1.x < 0 && i.x < 0 ) )
            add_triangle( i, point2, index, walls, debug )
            point2 = i
          end
        end

        if( l2.x * l2.x + l2.y * l2.y < @radius * @radius )
          i = Collision.intersects( point1, point2, SFML::Vector2.new( 0, 0 ), l2 )
          if( ( ( point1.x > i.x && point2.x < i.x ) || ( point1.x < i.x && point2.x > i.x ) ) &&
              ( ( point1.y > i.y && point2.y < i.y ) || ( point1.y < i.y && point2.y > i.y ) ) &&
              ( l2.y > 0 && i.y > 0 || l2.y < 0 && i.y < 0 ) &&
              ( l2.x > 0 && i.x > 0 || l2.x < 0 && i.x < 0 ) )
            add_triangle( point1, i, index, walls, debug )
            point1 = i
          end
        end

        m = Collision.collision( l1, l2, SFML::Vector2.new( 0, 0 ), point1 )
        n = Collision.collision( l1, l2, SFML::Vector2.new( 0, 0 ), point2 )
        o = Collision.collision( l1, l2, point1, point2 )

        if( ( m.x != 0 || m.y != 0 ) && ( n.x != 0 || n.y != 0 ) )
          point1 = m
          point2 = n
        else
          if( ( m.x != 0 || m.y != 0 ) && ( o.x != 0 || o.y != 0 ) )
            add_triangle( m, o, index, walls, debug )
            point1 = o
          end
          if( ( n.x != 0 || n.y != 0 ) && ( o.x != 0 || o.y != 0 ) )
            add_triangle( o, n, index, walls, debug )
            point2 = o
          end
        end
      end
      
      shape = SFML::Shape.new()
      if debug == true
        shape.outline = true
        shape.outline_thickness = 1
      end
      color = SFML::Color.new( ( @intensity * @color.r ).to_i,
                               ( @intensity * @color.g ).to_i,
                               ( @intensity * @color.b ).to_i )
      shape.add_point( 0.0, 0.0, color, SFML::Color::Green )
      
      dist = Math.sqrt( point1.x * point1.x + point1.y * point1.y )
      intensity = ( @radius - dist ) / @radius * @intensity
      intensity = @intensity - Math.sqrt( point1.x * point1.x + point1.y * point1.y ) * @intensity / @radius
      color = SFML::Color.new( ( intensity * @color.r ).to_i,
                               ( intensity * @color.g ).to_i,
                               ( intensity * @color.b ).to_i )
      shape.add_point( point1, color, SFML::Color::Green )

      dist = Math.sqrt( point2.x * point2.x + point2.y * point2.y )
      intensity = ( @radius - dist ) / @radius * @intensity
      intensity = @intensity - Math.sqrt( point2.x * point2.x + point2.y * point2.y ) * @intensity / @radius
      color = SFML::Color.new( ( intensity * @color.r ).to_i,
                               ( intensity * @color.g ).to_i,
                               ( intensity * @color.b ).to_i )
      shape.add_point( point2, color, SFML::Color::Green )
      shape.blend_mode = SFML::Blend::Add
      shape.position = @position.clone
      @shapes.push( shape )
    end

    def generate( walls, debug = false )
      @shapes.clear()
      buf = (Math::PI * 2) / @quality.to_f
      for i in 0...(@quality.to_i)
        point1 = SFML::Vector2.new( @radius * Math.cos( i * buf ),
                                    @radius * Math.sin( i * buf ) )
        point2 = SFML::Vector2.new( @radius * Math.cos( ( i + 1 ) * buf ),
                                    @radius * Math.sin( ( i + 1 ) * buf ) )
        add_triangle( point1, point2, 0, walls, debug )
      end
    end
    
  private
    def initialize( position, intensity, radius, quality, color )
      @position = position.clone
      @intensity = intensity.to_f
      @radius = radius.to_f
      @quality = quality.to_i
      @color = color.clone
      @shapes = []
    end
  end
end

module Collision
  def self.intersects( point1, point2, point3, point4 )
    i = SFML::Vector2.new( 0.0, 0.0 )
    if( point2.x - point1.x == 0 and point4.x - point3.x == 0 )
      i.x = 0
      i.y = 0
    elsif( point2.x - point1.x == 0 )
      i.x = point1.x

      c = ( point4.y - point3.y ) / ( point4.x - point3.x )
      d = point3.y - point3.x * c

      i.y = c * i.x + d
    elsif( point4.x - point3.x == 0 )
      i.x = point3.x

      a = ( point2.y - point1.y ) / ( point2.x - point1.x )
      b = point1.y - point1.x * a

      i.y = a * i.x + b;
    else
      a = ( point2.y - point1.y ) / ( point2.x - point1.x )
      b = point1.y - point1.x * a
      c = ( point4.y - point3.y ) / ( point4.x - point3.x )
      d = point3.y - point3.x * c

      i.x = ( d - b )/( a - c )
      i.y = a * i.x + b
    end

    return i
  end

  def self.collision( point1, point2, point3, point4 )
    i = self.intersects( point1, point2, point3, point4 )
    if( ( ( i.x >= point1.x - 0.1 && i.x <= point2.x + 0.1 ) or
          ( i.x >= point2.x - 0.1 && i.x <= point1.x + 0.1 ) ) and
        ( ( i.x >= point3.x - 0.1 && i.x <= point4.x + 0.1 ) or
          ( i.x >= point4.x - 0.1 && i.x <= point3.x + 0.1 ) ) and
        ( ( i.y >= point1.y - 0.1 && i.y <= point2.y + 0.1 ) or
          ( i.y >= point2.y - 0.1 && i.y <= point1.y + 0.1 ) ) and
        ( ( i.y >= point3.y - 0.1 && i.y <= point4.y + 0.1 ) or
          ( i.y >= point4.y - 0.1 && i.y <= point3.y + 0.1 ) ) )
      return i
    else
      return SFML::Vector2.new( 0.0, 0.0 )
    end
  end
end
```