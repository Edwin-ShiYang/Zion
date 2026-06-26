
![[MP.AABB2.IsPointInside_01.png| center | 800]]


```cpp
bool AABB2::IsPointInside( Vec2 const& point ) const
{
	bool isInsideX = ( m_mins.x < point.x ) && ( point.x < m_maxs.x );
	bool isInsideY = ( m_mins.y < point.y ) && ( point.y < m_maxs.y );

	return isInsideX && isInsideY;
}	
```