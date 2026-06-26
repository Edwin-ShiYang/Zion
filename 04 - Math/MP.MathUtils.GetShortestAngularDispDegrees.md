![[MP.MathUtils.GetShortestAngularDispDegrees_01.png| center | 600]]

![[MP.MathUtils.GetShortestAngularDispDegrees_02.png| center | 700]]


```cpp
float GetShortestAngularDispDegrees( float startDegrees, float endDegrees )
{
	float displacement = endDegrees - startDegrees;

	while ( displacement > 180 )  { 
		displacement -= 360;
	}

	while ( displacement < -180 ) {
		displacement += 360;
	}

	return displacement;
}
```