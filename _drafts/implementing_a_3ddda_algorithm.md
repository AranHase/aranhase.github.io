---
layout: post
title: Implementing a 3DDDA Algorithm for Raytracing
tags: algorithm
---

Recently I needed to implement a 3DDDA (3D Digital Differential Analyzer)
([wikipedia][1]) and found a really good paper by John Amanatides and Andrew
Woo called *A Fast Voxel Traversal Algorithm for Ray Tracing* (1987)([link][2])
describing the algorithm.  Unfortunately, I somehow got stuck for two days with
really basic edge cases, so I decided to write down my solution so you can have
a better time.


The algorithm I'm describing here is actually the 2D version, but, its a simple
step to add another dimension because all that is needed is to copy what the
others two dimensions are doing (\*＾▽＾)／. Also, note that my implementation
is not for a traditional DDA algorithm. It assumes the ray may be generated
both inside and outside a grid. So, before traversing the cells of the grid
with a 3DDDA algorithm, this implementation will find the intersection of the
ray to the grid to find the entry point, and only then do the 3DDDA.


I recommend reading the paper before looking at this post as I'm not going to
discuss much about it. Also, familiarity with Python (more precisely *ipython
notebook* with *pylab*) may help.


Before we get to the actual algorithm, we need a few
utility classes. Note that the following code are not the complete solution, it
is more an educational post. For the complete and working solution, please
check my [gist][3].

Utility classes
===============

The first is a simple Ray class. Its function is to hold the origin and
direction of a ray and prove a normalization function (it transforms the ray
into a unit ray).

{% highlight python %}
    class Ray:
        def __init__(self):
            self.origin = np.zeros((2))

        def norm(self):
            self.direction = self.direction / np.linalg.norm(self.direction)
{% endhighlight %}

The next is an axis-aligned bounding box implementation because we need it for the intersection test:

{% highlight python %}
    class AABB:
        def __init__(self, low, high):
            self.low = low
            self.high = high
    
        def intersects(self, ray):
            r_t_min = 0
            r_t_max = 0
            t_min = 0
            t_max = 0
            ty_min = 0
            ty_max = 0
    
            size = self.high - self.low;
    
    
            bounds_x1 = self.low[0];
            bounds_x2 = self.low[0];
            bounds_y1 = self.low[1];
            bounds_y2 = self.low[1];
            irayd = 1./ray.direction
            if (irayd[0] >= 0):
                bounds_x2 += size[0];
            else:
                bounds_x1 += size[0];
            if (irayd[1] >= 0):
                bounds_y2 += size[1];
            else:
                bounds_y1 += size[1];
    
            t_min =  (bounds_x1 - ray.origin[0]) * irayd[0];
            t_max =  (bounds_x2 - ray.origin[0]) * irayd[0];
            ty_min = (bounds_y1 - ray.origin[1]) * irayd[1];
            ty_max = (bounds_y2 - ray.origin[1]) * irayd[1];
    
            t_min = fmax(t_min, ty_min);
            t_max = fmin(t_max, ty_max);
    
            if (t_min < t_max) and (t_max > 0):
              r_t_min = t_min;
              r_t_max = t_max;
              return (True,r_t_min,r_t_max)
            return (False,-1,-1)
{% endhighlight %}


The `Grid` is a very simple class that hold voxels, just note that it has
`(width+1)*(height+1)` elements in it. The idea is that the grid holds `width`
quadrants in the `x` axis and `height` in the `y` axis, where each quadrant has
4 values.

{% highlight python %}
    class Grid:
        def __init__(self, aabb):
            self.width = aabb.high[0]-aabb.low[0]
            self.height = aabb.high[1]-aabb.low[1]
            self.grid = np.zeros((self.width+1,self.height+1))
            self.aabb = aabb
{% endhighlight %}


This `diff_distance` function will be very useful soon. For now, just imagine
that it takes the fraction part of the value `s` and calculates how much until
it reaches `1` when incremented by `ds`.

{% highlight python %}
    # Find the distance between "frac(s)" and "1" if ds > 0, or "0" if ds < 0.
    def diff_distance(s,ds):
        if s < 0:
            s = s - int(-1+s)
        else:
            s = s - int(s)
        if ds > 0:
            return (1.-s)/ds
        else:
            ds = -ds
            return s/ds
{% endhighlight %}

The Traversal Algorithm
=======================

Let's jump straight to the implementation. We will have 4 main methods: the
initialization, the step, the voxel the algorithm is at and its interval (along
the ray). The constructor is simple. We just copy the ray and grid to the
traversal class and check if the direction of the ray is zero because we will
need to divide by it later.


{% highlight python %}
    class AmanatidesTraversal:
        def __init__(self, grid, ray):
            self.grid = grid
            self.ray = Ray()
            self.ray.origin = ray.origin.copy()
            self.ray.direction = ray.direction.copy()
            if self.ray.direction[0] == 0:
                self.ray.direction[0] = sys.float_info.min
            if self.ray.direction[1] == 0:
                self.ray.direction[1] = sys.float_info.min
            self.t_min = 0
{% endhighlight %}


The `initialize()` is the method the paper did not explains well. The method
sets all the important variables that will be crucial for the `step()` method.
It should be called just once, then should return `True` if the ray hits the
grid, or `False` if the ray misses the grid. Basically we want the following
variables:


* `self.voxel`: The first voxel the ray enters.
* `self.step_[x|y]`: The direction of the ray.
* `self.delta_[x|y]`: The distance a ray travels when crossing a single voxel.
* `self.t_max_[x|y]`: The distance until the next voxel. We are counting from the point where the ray enters the grid cube.


{% highlight python %}
class AmanatidesTraversal:
  def initialize(self):
     (cube_result, cube_hit_t_min, cube_hit_t_max) = self.grid.aabb.intersects(self.ray)
     if cube_result:
         cube_hit_point = self.ray.origin + (cube_hit_t_min) * self.ray.direction
         self.t_min = cube_hit_t_min
         self.cube_hit_t_min = cube_hit_t_min
  
         print "DDA: Cube Hit Point:", cube_hit_point
  
         self.step_x = copysign(1., self.ray.direction[0])
         self.step_y = copysign(1., self.ray.direction[1])
  
         self.t_delta_x = (self.step_x / self.ray.direction[0])
         self.t_delta_y = (self.step_y / self.ray.direction[1])
  
         self.t_max_x = diff_distance(cube_hit_point[0], self.ray.direction[0])
         self.t_max_y = diff_distance(cube_hit_point[1], self.ray.direction[1])
  
         if cube_hit_point[0] < 0:
             cube_hit_point[0] -= 1
         if cube_hit_point[1] < 0:
             cube_hit_point[1] -= 1
         self.voxel = np.array(cube_hit_point, dtype=int)
         print "DDA: Initial Voxel:", self.voxel
         '''
         this conditional solves the problem where the "cube_hit_point" is just
         outside the grid because of floating point imprecision.
         '''
         while self.voxel[0] < self.grid.aabb.low[0] or self.voxel[1] < self.grid.aabb.low[1]\
         or self.voxel[0] >= self.grid.aabb.high[0] or self.voxel[1] >= self.grid.aabb.high[1]:
             print "DDA: Skyping:", self.voxel
             if not self.step():
                 return False
  
         return True
     else:
         return False
{% endhighlight %}


The conditional verifying if the `self.voxel` is valid or not was necessary
because sometimes the point where the ray hits the grid cube may end being
outside the cube.


The next method is the `step()`. This method is called every time we want to
move to the next voxel crossed by the ray. The `self.t_min` gets set to the
previous `self.t_max_[x|y]`, then the next `self.t_max_[x|y]` is set.


{% highlight python %}
    class AmanatidesTraversal:
        def step(self):
            self.t_min = (min(self.t_max_x,self.t_max_y) + self.cube_hit_t_min)
            if (self.t_max_x < self.t_max_y):
                self.t_max_x += self.t_delta_x
                self.voxel[0] += self.step_x
                if self.voxel[0] >= self.grid.aabb.high[0] or self.voxel[0] < self.grid.aabb.low[0]:
                    return False
            else:
                self.t_max_y += self.t_delta_y
                self.voxel[1] += self.step_y
                if self.voxel[1] >= self.grid.aabb.high[1] or self.voxel[1] < self.grid.aabb.low[1]:
                    return False
            return True
{% endhighlight %}

The outside world just need to know two things: the voxel index and its interval. It is easily computed with:

{% highlight python %}
    class AmanatidesTraversal:
        def get_voxel(self):
            return self.voxel.copy()

        def get_t_interval(self):
            return (self.t_min,(min(self.t_max_x,self.t_max_y) + self.cube_hit_t_min))
{% endhighlight %}

And that's it! The 3D version can be found in the paper, but it's as easy as
the 2D once you have a working `initialize()` method.

Using the implementation
========================

Let's throw a ray into our grid and see what happens:

{% highlight python %}
    ray = Ray()
    ray.origin = np.array([-7.65,3.27])
    ray.direction = np.array([+0.8001,-0.29])
    ray.norm()
    
    gridaabb = AABB(np.array([-3,-2]), np.array([6,3]))
    grid = Grid(gridaabb)
    print "Grid N. Quadrants: (", grid.width, ",", grid.height, ")"
    grid.plot(axes)
    
        traversal = AmanatidesTraversal(grid, ray)
    
    if traversal.initialize():
        # this ray should be plotted outside the grid
        ray.plot(axes, 0, traversal.get_t_interval()[0])
        cube_hit = ray.origin + traversal.get_t_interval()[0]*ray.direction
        axes.plot(cube_hit[0],cube_hit[1], 'og', ms=10)
    
        while(True):
            # Ignore while t_max is negative.
            if traversal.get_t_interval()[1] < 0:
                if not traversal.step():
                    break
                continue
            ray.plot(axes, traversal.get_t_interval()[0], traversal.get_t_interval()[1], color=random.random(3))
    
            ray_tmax = ray.origin + traversal.get_t_interval()[1]*ray.direction
            axes.plot(ray_tmax[0],ray_tmax[1], 'or', ms=10);
    
            print "voxel:", traversal.get_voxel()
            if not traversal.step():
                break
{% endhighlight %}

The usage should be straightforward. The code above results in (<3 python matplotlib): ![amanatides][4]

Additional notes
================

* The algorithm should work fine on negative grids.
* For rays that starts inside the grid, I found easier to just step the ray
  until the "t_max" is positive.
* The above algorithm works for a grid with uniform spaced values, where each
  quadrant is axis-aligned and has a size of 1 on each axis. Applying
transformation on the grid is not very wise, my tip is to apply the inverse
transformation to your ray instead.
*  Found any bug? Hit me on comments or on GitHub on my Gist, thank you!

 [1]: http://en.wikipedia.org/wiki/Digital_differential_analyzer_(graphics_algorithm)
 [2]: http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.42.3443
 [3]: https://gist.github.com/AranHase/9d73beda100a360b3008
 [4]: /assets/images/amanatides2.png
