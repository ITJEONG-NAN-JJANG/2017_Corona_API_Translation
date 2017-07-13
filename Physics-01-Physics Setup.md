# [Developer Guides] Physics Setup
<https://docs.coronalabs.com/guide/physics/physicsSetup/index.html>

###### Documentation > Developer Guides > Physics > Physics Setup 를 번역한 글입니다. 가끔씩 오타/번역오류 등이 있을 수 있으며, 어떠한 이득을 목적으로도 하지 않습니다.

# Physics Setup

 이 가이드는 코로나 앱에서 어떻게 Box2D 물리 엔진을 설정할 수 있는지 설명합니다. physics는 중력과 같은 다양한 물리적 힘 아래에서 이동하고, 충돌하고, 상호 작용하는 물체의 시뮬레이션을 포함하는 어플리케이션에서 흔히 사용됩니다.
 
## Overview

 코로나는 당신이 물리 엔진을 사용한 적이 없었더라도, 앱에 physics를 쉽게 추가할 수 있게 해 줍니다. 코로나는 대중적인 Box2D 엔진으로 구축되어 있지만, 과거에 필요했던 많은 코딩을 줄이는 다른 설계 접근법을 사용했습니다.
 
 코로나의 물리 엔진을 사용하면, 친근한 *display object*들로 시작할 수 있습니다. 코로나는 display object의 확장으로서 physical body를 인식합니다. 이미지, 벡터 요소 또는 스프라이트 등의 어떤 display object라도 physical body가 될 수 있고, 이것들은 자동적으로 다른 object들과 상호작용할 것입니다.

 코로나가 픽셀 단위와 코로나 시뮬레이션의 미터법을 자동적으로 변환한다는 점을 유의하세요. 모든 위치 값들은 30pixels-per-meter의 기본 비율을 통해 변환되어 픽셀로 표현될 것입니다. 일관성을 위해 모든 각도 값은 라디안이 아니라 각도로 표현되며, `(0,0)` 지점은 화면의 왼쪽 위 부분입니다.
 
## Physics Setup

 physics의 사용을 위해, physics 라이브러리를 요청해야 합니다.
 다음 줄은 `physics`라는 변수를 통해 물리 엔진을 사용할 수 있게 해줍니다. 

~~~
local physics = require "physics
~~~
 
### Start / Pause / Stop
 
 다음의 기능들은 physics 시뮬레이션을 각각 시작, 일시 중지, 그리고 중지합니다.

* *physics.start()* - 시뮬레이션을 다시 시작하거나 재시작합니다(일시 중지 상태였을 경우) 다른 물리적 기능들을 사용하기 전에 **반드시** 호출되어야 합니다.

* *physics.pause()* - physics 시뮬레이션을 일시 중지합니다.

* *physics.stop()* - physics 시뮬레이션 세계를 '파괴'합니다. 일시 정지를 원할 경우, *physics.pause()*를 사용하십시오.

~~~
physics.start() - 어떠한 물리 시뮬레이션이 시작되기 전에 호출되어야 합니다.

physics.pause()  
physics.stop()
~~~

### Sleeping Bodies

 기본적으로 충돌에 관여하지 않는 'physics body'는 몇 초 후에 'sleep' 상태로 전환됩니다. 이 기능은 효율적이지만, 일부 상황에서는 이러한 동작이 불필요할 수 있습니다. 특히 가속도계를 사용하여 중력 변화에 영향을 미치는 앱이 이에 해당됩니다 - 이 경우 'sleep' 상태로 전환된 물체들은 중력의 방향 변화에 반응하지 않습니다.
 
 당신은 `body.isSleepAllowed = false` 명령어를 사용하여 하나의 물체를 재정의하거나, `physics.start()`에 boolean 형태의 파라미터 값을 넣어줌으로써 **모든** 물체들을 재정의할 수 있습니다.

~~~
physics.start( true ) --

physics.start( false ) --
~~~
 
## Simulation Options

 이 엔진에는 여러 가지 글로벌 시뮬레이션 옵션이 있습니다. 이를 통해 기본 동작 이외의 physics 시뮬레이션을 조정할 수 있습니다.

### physics.setGravity()

 *physics.setGravity()*는 m/s2 단위로, 글로벌 중력 벡터의 **x**,**y** 성분을 설정합니다. 기본 값은 지구의 중력과 같은 `( 0, 9.8 )`이며, 이 경우 **y** 축으로만 중력이 작용합니다.

~~~
physics.setGravity( 0, 6 );
~~~

### physics.setScale()

 *physics.setScale()*은 화면의 코로나 좌표와 시뮬레이션 된 물리 자표 간이 변환에 사용되는 내부 pixels-per-meter 비율을 설정합니다. 이 과정은 물리 객체가 활성화 되기 전 한 번만 수행되어야 합니다.

 이 값을 변경하면 물리적 정확성에만 영향을 끼칠 뿐, 시각적인 변화가 나타나지는 않습니다. Box2D 엔진은 0.1m와 10m 크기 사이에서 물체를 변환합니다. 그래서 게임 내의 물체들이 범위 내의 physics 객체들로 맵핑될 때 사용하는 것이 가장 적합합니다.

기본값은 `30`이며, 이는 0.1m와 10m가 각각 3px과 300px에 맵핑됨을 의미합니다. 더 큰 물체들을 다룰 경우, 이 값을 `60`이나 그 이상으로 바꿀 수 있습니다.

~~~
physics.setScale(60)
~~~

 또한 시뮬레이션하려는 객체가 너무 느리거나, 느린 반응을 보이는 경우에도 이 값을 늘릴 수 있습니다. 이 경우, 물체의 사이즈가 커지거나, 밀도가 커지기 때문에 크고 무거울 수 있습니다.

### physics.setDrawMode()

 *physics.setDrawMode()*는 물리 엔진이 세 가지 '렌더링 모드'를 설정합니다.  이 기능이 기기에서 동작하는 동안 예상치 못한 물리 엔진 오류가 생겼을 때, 코로나 시뮬레이터에서 유용하게 사용될 것입니다.

~~~
physics.setDrawMode( "hybrid" ) -- 시각 객체들의 충돌 경계면을 보여줍니다.  
physics.setDrawMode( "normal" ) -- 코로나 렌더러 기본값. 충돌 경계면을 보여주지 않습니다.  
physics.setDrawMode( "debug" ) -- 충돌 엔진의 경계면만을 보여줍니다.  
~~~

 physics data는 다양한 객체 유형과 속성을 반영하는 컬러 벡터 그래픽을 사용하여 표시됩니다. 상세한 내용을 위한 'Physics Bodies' 가이드를 보세요. 

 * orange - 동적 물리 객체 
 * dark blue - kinematic 물리 객체 
 * green - 정적, 움직임이 없는 물리 객체 
 * gray - 활동성이 없어 "sleep" 상태로 전환된 객체 
 * light blue - 물리 조인트 ( Physics Joint 가이드를 참고하세요 )
 
> **중요**  
> 코로나 시각 객체들과 Box2D를 사용하고자 할 때, Box2D는 모든 물리 객체가 **글로벌 좌표계**를 공유한다는 것을 명심해야 합니다. 모든 시각 객체들은 그 그룹의 내부 좌표를 공유하기 때문에 잘 작동할 것입니다. 그러나, 물리 객체들이 그룹에서 움직이거나, 스케일이 조정되거나, 회전하기 위해 물리 객체가 추가된다면, 예상치 못한 결과를 가져올 수도 있습니다. 일반적인 경우에는 물리 객체가 포함된 그룹의 위치, 크기 또는 회전률을 변경하지 **마세요**.
 
### physics.setPositionIterations()

 physics.setPositionIterations()는 엔진 위치 계산의 정확도를 설정합니다.

~~~
physics.setPositionIterations( 6 )  
~~~

 기본값은 `3`이며, 엔진이 각 물체에 대해 매 프레임 당 8개의 위치 근사치를 통과할 수 있음을 의미합니다. 이 값을 늘리면 중복 물체와 같은 일시적인 오류가 최소화되지만, 컴퓨터 오버헤드가 증가할 수 있습니다. 보통의 경우에는 기본값을 사용합니다.

### physics.setVelocityIterations()

 physics.setVelocityIterations()는 엔진 속도 계산의 정확도를 설정합니다.

~~~
physics.setVelocityIterations( 16 )
~~~

 기본값은 `8`이며, 엔진이 각 물체에 대해 매 프레임 당 3개의 속도 근사치를 통해 엔진이 감속됨을 의미합니다. 이 값을 증가시키면, 중복 물체와 같은 일시적인 오류가 최소화되지만, 컴퓨터 오버헤드가 증가할 수 있습니다. 보통의 경우에는 기본값을 사용합니다.

### physics.setContinuous()

 physics.setContinous()는 연속적인 physics의 활성화 여부를 제어합니다 일반적으로, Box2D는 연속적인 충돌 처리를 수행하는데, 이는 물체가 '터널링'되는 것을 방지할 수 있습니다. 만약 이 값이 false라면, 충분히 빠르게 움직이는 물체는 얇은 벽을 통과할 수도 있습니다.
 예를 들어서 피벗 조인트(ragdolls)에 의해 연결되는 특정한 복잡한 상황에서는 불안정성이 발생할 수 있습니다. 이를 해결하기 위해 physics.setVelocityIteratoins()의 값을 증가시켜, 속도 반복 횟수를 늘릴 수 있습니다. 그러나 이 방법은 비용이 많이 들고, FPS를 낮출 수 있습니다. 그래서 또 다른 해결책으로 physics를 **비활성화** 할 수 있습니다.

~~~
physics.setContinuous( false )
~~~

 이 값을 false로 전환하면 반복 작업을 늘리지 않고도 불안정성을 방지할 수 있지만, 터널링 효과의 방지를 위해 정적 물체를 더욱 두껍게 할 필요가 있습니다.

### physics.setAverageCollisionPositions()

 physics.setAverageCollisionPositions()은 물리 객체들 사이의 모든 충돌 지점의 평균값을 산출합니다. Box2D에서 단일 프레임 반복 중의 다중 충돌 지점을 보고하는 것이 일반적이기 때문에, 이 설정은 충돌지점들을 하나의 평균 포인트로 지정할 때에 유리합니다.

~~~
physics.setAverageCollisionPositions( true )
~~~

### physics.setReportCollisionInContentCoordinates()

 physics.setReportCollisionsInContentCoordinates()는 'collision', 'preCollision', 'postCollision' 등을 물리 이벤트의 기본 포인트로 설정해 줍니다.

~~~
physics.setReportCollisionInContentCoordinates( true )
~~~

### physics.setDebugErrorsEnabled()

physics.setDebugErrorsEnabled()는 Box2D에 의해 발견된 기타 물리 오류들을 허용합니다. 기본값은 `true` 입니다.

### physics.setTimeStep()

physics.setTimeStep()은 특정한 프레임 기반의 물리 시뮬레이션이나 시간 기반의 시뮬레이션을 지정합니다.

~~~
physics.setTimeStep(0)
~~~

### physics.setMKS()

 physics.setMKS()는 특정 키에 대한 물리 시뮬레이션의 MKS(Meters, Kilograms, and Seconds) 값을 설정합니다. 이 설정은 고급 설정으로, 일반적인 개발자나 프로젝트에서 이 기능을 필요로 하지는 않을 것입니다.
 
 
 
###### © 2017 Corona Labs Inc. All Rights Reserved. (Last updated: 17-May-2017)
