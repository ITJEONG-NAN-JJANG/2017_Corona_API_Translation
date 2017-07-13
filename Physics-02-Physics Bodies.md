# [Developer Guides] Physics Bodies

<https://docs.coronalabs.com/guide/physics/physicsBodies/index.html>

###### Documentation > Developer Guides > Physics > Physics Setup 를 번역한 글입니다. 가끔씩 오타/번역오류 등이 있을 수 있으며, 어떠한 이득을 목적으로도 하지 않습니다.

# Physics Bodies

 이 가이드는 코로나를 이용하여 multi-element body들을 포함한 Box2D의 physics body들을 만드는 방법을 설명하고 있습니다.

## Overview

 이 물리 세계는 physical body들의 상호작용을 기반으로 하고 있습니다. 코로나는 이러한 physicsl body들을 *display object들*의 확대로 간주합니다. 이 physics body는 1:1 대응 관계의 코로나의 display object에 바인딩 될 수 있으며, 코로나는 모든 위치 업데이트 및 기타 동기화 작업을 자동으로 처리합니다.

 `x`,`y` 및 `rotate`처럼 표준 개체 읽기/쓰기 특성은 physical body들에 계속해서 적용되어야 합니다. 그러나, 만약 개체의 `bodyType`이 'dynamic'일 경우, 중력이나 다른 힘의 지속적인 영향을 받는 한 물리 엔진은 물체를 움직이려는 명령을 무시할 수도 있습니다.

 physics body가 적용된 display object들은 *object:removeSelf()*나 *display.remove()*를 사용하여 정상적으로 삭제될 수 있습니다. 이 경우에, 스크린**과** 물리 시뮬레이션(physics body 데이터가 파괴될 것입니다.) 모두에서 삭제될 것입니다. 또한, 스크린에는 display object가 유지되는 반면, 단지 physical body만 삭제하도록 *physics.removeBody()*를 사용할 수 있습니다. *destroying object*에 대한 자세한 정보와 충돌 무시에 대한 영향력이 큰 기록을 보세요.

## Creating Bodies

 코로나에서의 모든 physical body들은 *physics.addBody()* 함수를 사용해서 만들어집니다. 이 기능은 어떠한 Corona display object도 물리적 속성이 할당된 코드를 통해 physical object로 변환할 수 있게 해줍니다.

~~~
physics.addBody( 대상 개체, { 속성들 } )
~~~

### Physical Properties

 모든 physical body들은 3개의 핵심 속성을 가지고 있으며 각각의 속성들은 속성 테이블에서 key-value 쌍으로 정의됩니다. 이 속성들은 선택적이며 명시되지 않았다면, 기본값이 적용될 것입니다.

#### Density

 `밀도(density)`는 질량을 결정하는 물체의 영역에 의해 변합니다. 이 인자값은 물의 밀도인 `1.0`라는 표준값을 기반으로 해서, 나무와 같이 물보다 가벼운 물질은 `1.0`보다 낮은 밀도값을 가지게 되고, 돌과 같은 무거운 물질에 대해서는 `1.0`보다 큰 밀도값을 가지게 됩니다. 하지만 전반적인 개체의 움직임은 중력과 pixels-to-meter-scale 설정( *Physics Setup* 을 보십시오 )에 달려 있기 때문에 시뮬레이션에 적합한 느낌으로 밀도 값을 자유롭게 설정할 수 있습니다. 기본값은 `1.0`입니다.

#### Friction

`friction`은 어떠한 값도 될 수 있습니다. `0`값은 마찰이 없음을 의미하고, `1.0`은 상당히 강한 마찰을 의미합니다. 기본값은 `0.3`입니다.

#### Bounce

 `bounce`는 Box2D에서 "restitution" 설정으로 알려져있고, 충돌 후에 물체의 속도가 얼마나 되는지를 결정합니다. `0.3`보다 큰 값은 꽤 "통통 튀는" 값이고, `1.0`일 경우에 무한히 튀어오를 것입니다. - 예를 들어, 만약에 물체가 땅에 떨어진다면, 이 물체는 떨어졌을 때의 높이와 거의 비슷한 높이로 다시 튀어오릅니다. 바운스는 `1.0`보다 큰 값이 유효하게 적용됩니다. 그리고 충돌이 일어날 때마다 속도가 커질 수 있습니다. 기본값은 약간 탄력이 있는 정도인 `0.2`입니다.

~~~
--예제:

local crate = display.newImage( "crate.png", 100, 200 )
physics.addBody( crate, { density=1.0, friction=0.3, bounce=0.2 } )
 
local balloon = display.newImage( "balloon.png", 200, 200 )
physics.addBody( balloon, { density=0.1, friction=0.1, bounce=0.4 } )
~~~

 물리적 특성을 담고있는 테이블을 미리 선언하면, 여러 차례 사용될 수 있습니다:

~~~
local crate1 = display.newImage( "crate1.png", 100, 200 )
local crate2 = display.newImage( "crate2.png", 180, 280 )
  
local crateMaterial = { density=1.0, friction=0.3, bounce=0.2 }
  
physics.addBody( crate1, crateMaterial )
physics.addBody( crate2, crateMaterial )
~~~

### Body Type

 physical body들은 **dynamic**, **static** 또는 **kinematic**의 세 가지 중 하나의 타입을 가질 수 있습니다.

#### dynamic

 dynamic body들은 완전히 표현됩니다. 코드를 통해 수동적으로 움직일 수 있지만 일반적으로는 중력이나 충돌의 반동에 따라서 움직입니다. Box2D에서 physical object의 기본적인 바디 타입입니다. dynamic body들은 다른 타입의 body들과 충돌할 수 있습니다.

#### static

 static body들은 시뮬레이션 상에서 움직이지 않으며, 마치 무한적인 질량을 가진 것처럼 행동합니다. static body는 수동적으로 움직일 수 있지만 속도에는 영향을 받지 않습니다. static body들은 dynamic body와만 충돌하며, 다른 static body나 kinematic body와는 충돌하지 않습니다.

#### kinematic

 kinematic body들은 속도에 따른 시뮬레이션 아래에서만 움직입니다. kinematic body들은 중력과 같은 힘에 반응하지 않을 것입니다. kinematic body들은 dynamic body와만 충돌하며, kinematic body나 static body들과는 충돌하지 않습니다.

> 중요
> 일부 body type은 다른 body type들과 충돌하거나 충돌하지 않을 수 있습니다. 두 물리적 물체들이 충돌할 때, 다른 body type과 출동하는 유일한 body type이기 때문에 적어도 **하나의** 물체는 dynamic 타입이어야 합니다.

 코로나에서 body type을 선언하는 것은 `physics.addBody()`라는 명령어로 한 번에 적용하거나 *object.bodyType*을 사용해 미리 생성할 수 있습니다.

~~~
--한 번에:
physics.addBody( triangle, "static", { density=1.6, friction=0.5, bounce=0.2 } )
 
--미리 생성:
physics.addBody( triangle, { density=1.6, friction=0.5, bounce=0.2 } )
triangle.bodyType = "static"
~~~

### Rectangular Bodies

 일반적으로, `physics.addBody()`는 이미지나 벡터 오브젝트를 둘러싼 가장자리로 이루어진 사각형 모양의 범위로 적용됩니다. 이것은 플랫폼, 큰 너비의 물체들 그리고 다른 간결한 사각형 물체들을 다룰 때 유용합니다. 이 사각형 범위에는 이미지 주위의 투명한 픽셀도 포함됩니다.

~~~
local platform = display.newImage( "platform.png", 600, 200 )
physics.addBody( platform, { density=1.0, friction=0.3, bounce=0.2 } )
~~~

 만약 physical body를 사각형 범위와 연결하고 싶지 않을 경우, `반지름` 속성을 포함하는 특별한 모양 데이터나 다각형의 좌표 테이블을 정의해야 합니다.

 의심할 바 없이, 물리 엔진이 실제로 물체들을 어떻게 간주하는지를 체크하기 위해 *physics.setDrawMode()*를 사용하면 됩니다.

### Circular Bodies

 circular body는 속성 테이블에서 `radius` 인자값에 의해 생성됩니다. 이는 계산할 때 대략적으로 고려할 수 있는 공, 바위 그리고 다른 물체들에 적합합니다.

~~~
local ball = display.newImage( "ball.png", 100, 100 )
physics.addBody( ball, { radius=50, density=1.0, friction=0.3, bounce=0.2 } )
~~~

circular body는 특별한 경우로 존재하기 때문에 oval body는 Box2D 충돌하는 형태로 존재할 수 없습니다. oval body를 만들기 위해서는 아래에서 볼 수 있는 다각형 좌표 테이블을 고려하는 것이 좋습니다.

### Polygonal Bodies

 polygonal body는 `shape` 값 사용으로 만들어질 수 있습니다. 루아 테이블의  **x**와 **y** 좌표쌍이 있는데, 이 좌표쌍은 어떠한 도형의 꼭짓점으로 정의되어 있습니다. 이 좌표들은 display object의 중심과 특별히 관련이 있습니다. 그러므로 `(0,0)`은 물체의 정 중앙에 일치되고, 좌표 `(-20,-10)`는 중심에서 왼 쪽으로 20픽셀, 위 쪽으로 10픽셀 이동한 점을 의미합니다.

~~~
--삼각형:
local triangle = display.newImage( "triangle.png" )
triangle.x = 200
triangle.y = 150
 
--모양 테이블을 정의합니다 ( 한 번 만들면, 여러 번 사용할 수 있습니다 )
local triangleShape = { 0,-35, 37,30, -37,30 }
 
physics.addBody( triangle, { shape=triangleShape, density=1.6, friction=0.5, bounce=0.2 } )
 
 
--오각형:
local pentagon = display.newImage( "pentagon.png" )
pentagon.x = 200
pentagon.y = 50
 
local pentagonShape = { 0,-37, 37,-10, 23,34, -23,34, -37,-10 }
 
physics.addBody( pentagon, { shape=pentagonShape, density=3.0, friction=0.8, bounce=0.3 } )
~~~

> **중요**
> * 점의 좌표는 반드시 **시계 방향**으로 선언되어야 합니다. 시계 반대 방향으로 점들의 좌표가 선언되었더라도 `hybrid`나 `debug` 모드에서 올바르게 보일지도 모르나, 충돌 기능이 제대로 수행되지 않을 것입니다.
> * 다각형은 **볼록**한 모양이여야 합니다. 사발이나 컵같은 오목한 모양의 도형은 만들 수 없습니다. 오목한 모양의 도형을 만들기 위해서는, 아래의 Multi-Element Bodies 설명처럼 여러 다각형들로 이루어진 body를 구성해야 합니다.
> * 다각형은 **최대 8개의 꼭짓점**과 변을 가질 수 있습니다. 더 많은 점이 필요하다면, 여러 개의 다각형을 합쳐서 body를 구성해야 합니다.

### Multi-Element Bodies

 위의 예시들은 직사각형, 원형, 볼록한 다각형처럼 하나의 요소로만 이루어진 body를 가정하고 있습니다. 그러나, 조금 더 다양한 상황에서 보다 정확한 충돌처리를 위한 `여러 개의` 다각형에서 body를 구성해야 할 것입니다. 또한 Box2D의 다각형은 반드시 볼록해야 하며, 오목한 형태의 물체는 여러 개의 body들로 구성되어야 합니다.

 multi-element body들을 만들 때에는, 각각의 요소는 분리된 *다각형 모양*으로 지정되어야 합니다. 이 모양들이 각각 지정되었을지라도, 전체 body의 일부분으로 간주될 것이고, 물리적 힘의 적용하에 독립적으로 움직이지 않을 것입니다.

 multi-element body의 구성은 필수적으로 간단한 다각형으로 구성되어야 합니다. - 첫 번째 요소 이후에 추가적으로 선언하면 됩니다.

~~~
physics.addBody( object, "static",
    { bodyElement1 },
    { bodyElement2 },
    --계속해서 추가할 수 있습니다.
)
~~~

 각각의 body 요소가 모양을 정의하면서 충돌이나 바운스 등의 고유한 물리적 특성을 가질 수 있다. 아래는 예시입니다:

~~~
local car = display.newImage( "car.png" )
local roofShape = { -20,-10, 20,-10, 20,10, -20,10 }
local hoodShape = { 0,-35, 37,30, -37,30 }
local trunkShape = { 0,-37, 37,-10, 23,34, -23,34, -37,-10 }
 
physics.addBody( car, "dynamic",
    { density=3.0, friction=0.5, bounce=0.2, shape=roofShape },
    { density=6.0, friction=0.6, bounce=0.4, shape=hoodShape },
    { density=4.0, friction=0.5, bounce=0.4, shape=trunkShape }
)
~~~

### Offset/Angled Rectangular Bodies

 만약 

### Edge Shape (Chain) Body

### Outline Bodies

## Destroying Bodies



###### © 2017 Corona Labs Inc. All Rights Reserved. (Last updated: 17-May-2017)