
> [!NOTE]
> Returns the width and height of the AABB2

![[MP.AABB2.GetDimensions_01.png| center | 800]]

```cpp
//-----------------------------------------------------------------------------------------------
Vec2 const AABB2::GetDimensions() const
{
	float width  = m_maxs.x - m_mins.x;
	float height = m_maxs.y - m_mins.y;

	return Vec2( width, height );
}
```

