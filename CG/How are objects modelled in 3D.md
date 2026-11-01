![](../assets/Pasted%20image%2020261101200736.png)
![](../assets/Pasted%20image%2020261101201847.png)
### Polyhedra

- Objects are a set of surface polygons that enclose an object interior.
- Simplest and fastest way to render objects.
- These objects are often referred to as standard graphics objects.
- packages may allow us to define objects as curved surfaces etc. but actually convert these to polygon meshes for display
- To define polyhedra we simply define the vertices of the polygons required.

### Quadric surfaces

- These are 3D surfaces described using quadratic equations
- ![](../assets/Pasted%20image%2020261101201019.png)

Example with spheres:

![](../assets/Pasted%20image%2020261101201059.png)
![](../assets/Pasted%20image%2020261101201106.png)


### Sweep Representations

- Sweep representations are useful for constructing 3 dimensional objects that possess translational, rotational or other symmetries.
- Objects are specified as a 2 dimensional shape and a sweep that moves that shape through a region of space

![](../assets/Pasted%20image%2020261101201211.png)


### Constructive Solid Geometry Methods

Constructive Solid Geometry (CSG) performs solid modelling by generating a
new object from two three dimensional objects using a set operation.

![](../assets/Pasted%20image%2020261101201347.png)

![](../assets/Pasted%20image%2020261101201403.png)

![](../assets/Pasted%20image%2020261101201423.png)
![](../assets/Pasted%20image%2020261101201434.png)

##### Ray casting

- Ray-casting is typically used to implement CSG operators when objects are described with boundary representations
- Ray casting is applied by determining the objects that are intersected by a set of parallel lines emanating from the xy plane along the z axis
- The xy plane is referred to as the firing plane

![](../assets/Pasted%20image%2020261101201600.png)

- Surface intersections along each ray are calculated and these are sorted according to distance from the firing plane
- The surface limits for the composite object are then determined by the specified set operation

![](../assets/Pasted%20image%2020261101201702.png)


### Octrees

- Octrees are hierarchical tree structures used to represent solid objects
- Octrees are typically used when the interior of objects is important
- Octrees are based on a two-dimensional representation scheme called quadtree encoding
- Quadtree encoding divides a square region of space into four equal areas until homogeneous regions are found. These regions can then be arranged in a tree

![](../assets/Pasted%20image%2020261101202015.png)

- Quadtree encodings provide considerable savings in storage when large colour areas exist in a region of space
- An octree takes the same approach as quadtrees, but divides a cube region of 3D space into octants
- Each region within an octree is referred to as a volume element or voxel. Division is continued until homogeneous regions are discovered.

![](../assets/Pasted%20image%2020261101202224.png)

- In 3 dimensions regions can be considered to be homogeneous in terms of colour, material type, density or any other physical characteristics
- Voxels also have the unique possibility of being empty


### Fractals

Fractal methods use procedures rather than equations to model objects (procedural
modelling). Natural objects can be realistically described using fractal geometry methods.

The model is not based on data, but rather on the implementation of a procedure following a particular set of rules.

![](../assets/Pasted%20image%2020261101202531.png)

![](../assets/Pasted%20image%2020261101202613.png)

- Although fractal objects, by definition have infinite detail, we only apply the transformation a finite number of times
- A procedural representation approaches a true representation as we increase the number of iterations
- The amount of detail is limited by the resolution of the display device, but we can always zoom in for further detail.

![](../assets/Pasted%20image%2020261101202730.png)

#### Fractal dimension

The amount of variation in the structure of a fractal object is described as the fractal
dimension. More jagged looking objects have larger fractal dimensions.

#### Types of fractals

![](../assets/Pasted%20image%2020261101202846.png)

#### Random Midpoint Displacement

One of the most successful uses of fractal techniques in graphics is the generation of landscapes. One efficient method for doing this is random midpoint displacement.

![](../assets/Pasted%20image%2020261101203043.png)
