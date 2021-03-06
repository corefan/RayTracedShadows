# Ray Traced Shadows

This demo implements BVH construction and GPU traversal for rendering hard shadows.

## BVH Construction and Layout

BVH is constructed on CPU. The build process is fairly naive, but results in a high quality hierarchy that's fast to traverse. The tree is constructed using a top-down strategy, using a surface area heuristic (SAH) to find optimal split point at every level.

Nodes are laid out in memory using a depth-first traversal order. Child node with the larger surface area is always on the left. This heuristic aims to find an intersected primitive for a ray in a cache-coherent manner.

Each intermediate BVH node is packed into 32 bytes:

	struct BVHNode
	{
		vec3 bboxMin;
		uint primitiveId;
		vec3 bboxMax;
		uint next;
	};

Leaf BVH nodes are packed into 48 bytes:

	struct BVHNodeLeaf
	{
		vec3 edge0;
		uint padding0;
		vec3 edge1;
		uint next; // node miss pointer
		vec3 vertex;
		uint padding2;
	};

## BVH Traversal

Hard shadows are implemented by using any-hit BVH traversal for a ray on GPU. A stackless algorithm is used, which relies on the depth-first memory layout of the tree.

The bounding box of each visited intermediate node is tested against a ray. On hit, the next node that must be visited is next in memory. On miss, current node's `next` pointer is used from the current node.
Each intermediate node contains a `primitiveId` field. If this field is not `0xFFFFFFFF`, then current node is reinterpreted as `BVHNodeLeaf`. Extra data for leaf nodes is stored deinterleaved (at the end of the BVH buffer).

## Acknowledgements

This demo uses similar ideas to what is described in the following work:

* [The Perfect BVH, Jacco Bikker, 2016](http://www.cs.uu.nl/docs/vakken/magr/2015-2016/slides/lecture%2003%20-%20the%20perfect%20BVH.pdf)
* [Implementing a practical rendering system using GLSL, Toshiya Hachisuka, 2015](http://www.cs.uu.nl/docs/vakken/magr/2015-2016/slides/lecture%2003%20-%20the%20perfect%20BVH.pdf)
* [AMD RadeonRays](https://github.com/GPUOpen-LibrariesAndSDKs/RadeonRays_SDK)
