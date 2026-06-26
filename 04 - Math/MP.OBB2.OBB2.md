## Oriented Bounding Box

> [!NOTE]
> Unlike an Axis-Aligned Bounding Box (AABB), which is always aligned with the world's X and Y axes, an OBB can be **freely rotated**.


![[MP.OBB2.OBB2_01.png| center | 500]]

```c++
struct OBB2 
{
	Vec2 center; Vec2 iBasisNormal; 
	Vec2 jBasisNormal; 
	Vec2 halfDimensions;
};
```

