# Chapter 2 Hello Box2D

Hello World 프로젝트를 통해 Box2D 사용법을 알아 보겠습니다. 본 프로그램은 거대한 바탕 상자와 작은 동적 상자를 만들지만 그래픽 관련 코드는 포함하지 않습니다. 그리고 사용자는 콘솔을 통해 박스의 위치를 바로 바로 알 수 있습니다.
Box2D 를 어떻게 사용하는지에 대한 좋은 예제입니다.

# 2.1 Creating a World

모든 Box2D 프로그램은 b2World 객체를 생성하는것으로 시작합니다. 해당 객체는 물리적 연산에 관련된 메모리, 객체, 시뮬레이션을 관리하는 허브 역할을 합니다. 사용자는 스택, 힙, 데이터에 해당 물리엔진을 생성할 수 있습니다.
 
Box2D의 world 를 생성하는 것은 쉽습니다. 우선, 중력관련 벡터를 설정합니다. 그리고 world 객체에게 강체들이 쉬고자 할때 잘 수 있도록 합니다.잠자기에 들어간 강체는 더이상 연산대상이 아닙니다.

	b2Vec2 gravity(0.0f, -10.0f);
	bool doSleep = true;
	
이제 world 객체를 생성합니다. 스택에 해당 객체를 생성하면, 해당 스택내에서 항상 world 객체가 release 되지 않도록 해야 합니다.
	
	b2World world(gravity);

이제 우리는 물리연산이 가능하게 되었습니다. 이제 뭔가를 추가해보도록 합니다.


# 2.2 Creating a Ground Box

강체의 생성은 다음과 같은 과정을 거칩니다.

1. 강체의 위치, 저항등을 설정함.
1. world 객체가 강체를 생성하도록 함.
1. fixture로 마찰, 비중, 모양등을 설정함.
1. 강체에 fixture를 생성함.

첫째로, 바닥 강체를 생성합니다. 이를 위해 강체를 정의해야 하며, 여기에서는 위치를 설정합니다.

	b2BodyDef groundBodyDef;
	groundBodyDef.position.Set(0.0f, -10.0f);

둘째로, 강체의 정의를 world 객체로 보내 바닥 강체를 생성하도록 합니다. world 객체는 강체 정의에 대한 참조를 저장해두지 않는다는 것을 명심하세요. 강체는 따로 설정하지 않으면 static 으로 생성됩니다. static 강체는 움직이지 않으며, 다른 static 강체와 충돌하지도 않습니다.

	b2Body* groundBody = world.CreateBody(&groundBodyDef);

셋째로, 바닥의 모양을 생성하도록 하겠습니다. 부모 강체의 중심에 맞추어 SetAsBox 함수를 통해 간단하게 박스 모양 바닥을 생성합니다.

	b2PolygonShape groundBox;
	groundBox.SetAsBox(50.0f, 10.0f);

SetAsBox 함수는 아규먼트로 외연이 되는 너비와 높이의 절반 값을 받습니다. 그러므로 상단의 경우, x 축으로 100단위 너비, y 축으로 20단위 높이가 됩니다. Box2D 는 미터, 킬로그램, 초 단위에 맞도록 튜닝되어 있습니다. 그러므로 외연이 미터가 되도록 해야 합니다. Box2D는 일반적으로 오브젝트가 실제 세상과 비슷할 때 정상적으로 작동합니다. 예를 들어, 드럼통은 1미터정도의 높이 입니다. 부동소수점 연산의 한계점을 고려할 때, 빙하나 먼지의 움직임을 Box2D 로 구현하고자 하는 것은 좋은 생각이 아닙니다.

마지막으로, 모양 관련 fixture 를 생성함으로써 바닥 강체 생성을 마치도록 하겠습니다. 이 단계에서는 fixture 객체의 생성 없이 직접 강체에 모양 관련 내용을 아규먼트로 줌으로써, fixture 에 다른 설정값을 대체하지 않도록 하겠습니다. fixture 정의에 대한 내용은 차후에 알아보도록 하겠습니다. 두번째 아규먼트는 kg/m^2 으로 된 밀도 값을 줍니다. static 강체는 질량이 0으로 정의되어 있으므로 밀도는 사용되지 않습니다.

	groundBody->CreateFixture(&groundBox, 0.0f);

Box2D는 shape 객체에 대한 참조값을 저장해두지 않으며, 해당 값은 새 b2Shape 객체로 복사됩니다.
fixture는 static 속성이라도 모든 fixture 객체는 반드시 부모 강체가 있어야 합니다. 물론, 한 개의 static 강체에 여러개의 static 속성 fixture를 붙일 수도 있습니다.

fixture 를 통해 강체에 특정 모양을 지정한다면, 모양의 좌표가 강체에 적용됩니다. 강체가 움직일때, 모양의 좌표도 마찬가지로 움직이며, fixture가 world 에서 움직이는 것은 부모 강체로 부터 상속된 속성입니다. fixture 는 강체로부터 독립적으로 움직일 수 없으므로, 강체 주변으로 모양 속성이 움직이도록 할 수는 없습니다. 강체에 귀속된 모양 속성을 함부로 움직이는 것은 지원되지 않습니다. 왜냐하면, 모양을 갖춘 강체의 형태를 바꾸는 것이 가능할 때, 이를 강체라고 정의할 수 없고, Box2D 는 강체 엔진이기 때문입니다. 

# 2.3 Creating a Dynamic Body

자, 이제 바닥 강체를 생성했습니다. 같은 식으로 dynamic 속성의 강체를 생성해보겠습니다. 제일 중요한 다른 점은 dynamic 속성의 강체에는 질량 속성을 주어야 한다는 것입니다.

먼저, CreateBody 를 사용해 강체를 생성합니다. 기본적으로 강체는 static 속성을 가지므로 b2BodyType 을 생성해 강체가 dynamic 속성을 가질 수 있도록 설정합니다.

	b2BodyDef bodyDef;
	bodyDef.type = b2_dynamicBody;
	bodyDef.position.Set(0.0f, 4.0f);
	b2Body* body = world.CreateBody(&bodyDef);

> ### 주의 <br>
> b2_dynamicBody 로 설정해야만 해당 강체가 물리법칙을 따릅니다.

다음으로, fixture 정의에 사용할 Polygon을 박스모양으로 생성합니다. 

	b2PolygonShape dynamicBox;
	dynamicBox.SetAsBox(1.0f, 1.0f);

fixture 정의 사항에 상단의 박스를 사용합니다. 밀도는 1로 설정합니다. 기본적인 생성값은 0입니다. 마찰은 0.3 으로 설정합니다.

	b2FixtureDef fixtureDef;
	fixtureDef.shape = &dynamicBox;
	fixtureDef.density = 1.0f;
	fixtureDef.friction = 0.3f;

> ### 주의 <br>
> fixture 를 생성할 때는 fixture definition을 사용합니다.<br>
> 이를 통해 강체의 질량을 자동적으로 적용할 수 있습니다. <br>
> 또한 이를 통해 여러 fixture 를 강체에 적용할 수 있습니다. <br>
> 각각의 것들은 총 질량에 반영됩니다.

	body->CreateFixture(&fixtureDef);

생성과정은 이것으로 완료되었습니다. 이제 시뮬레이션이 가능합니다. 바닥 강체와 dynamic 속성의 강체가 존재합니다. 이제 힘이 어떻게 작용하는지 볼 차례입니다만 몇 가지 이슈가 존재합니다.

That's it for initialization. We are now ready to begin simulating.o we have initialized the ground box and a dynamic box. Now we are ready to set Newton loose to do his thing. We just have a couple more issues to consider.

Box2D는 적분기라는 계산용 알고리즘을 사용합니다. 적분기는 시간의 이산 지점에 대해 물리법칙을 시연합니다. 이는 플립북 넘기기를 스크린으로 옮긴 것과 같은 전통적인 게임루프와 궤를 같이 합니다. 그래서 우리는 시간 단계를 설정할 필요가 있습니다. 일반적인 물리엔진은 1/60 초 내지는 60Hz 를 최소한의 요구사항으로 삼습니다. 이보다 큰 값을 설정할 수도 있습니다만, 좀더 신중해야할 필요가 있습니다. 또한, 너무 많이 변경하는 것도 추천하지 않습니다. 여러 가지 시간 단계는 여러 가지 결과물을 낳을 수 있고, 이는 디버깅의 어려움을 초래합니다. 그러므로 시간 단계가 프레임 레이트와 묶이도록 설정하지 마십시요. (애지간하면 말이죠.)
각설하고 아래가 시간 단계의 설정입니다. 

	float32 timeStep = 1.0f / 60.0f;

적분기에 대해 한마디 덧붙이자면, Box2D는 constraint solver 라는 것에 꽤나 공을 들였습니다. 이는 시뮬레이션 상의 여러 제약사항을 한번에 푸는 장치로 단순한 제약은 완벽히 풀 수 있습니다. 다만, 하나의 제약을 풀 때, 다른 제약 사항을 방해할 수도 있습니다. 이를 위한 해결책으로 단기간에 모든 제약을 반복할 수 있어야 합니다.

constraint solver 에는 속도 와 위치에 대한 두 가지 단계가 있습니다. 속도 단계에서는 강체를 정확히 움직일 수 있는 충격량을 정확히 계산합니다. 위치 단계에서는 강체간의 중첩이나 joint 간의 연결에 대한 부분을 조정합니다. 각각의 단계는 반복하기 위한 카운터를 가지고 있으며, 에러값이 작다면 반복값보다 먼저 해당 단계를 종료할 수 있습니다. 

Box2D는 8번의 속도 단계 반복 및 3번의 위치 단계 반복을 추천합니다. 물론, 사용자 마음대로 조정할 수 있으나 정확도와 성능효율 면에서 충분히 신뢰할 수 있는 횟수입니다. 반복 횟수를 줄이면 성능효율은 올라가지만 정확도 면에서 손해를 봅니다. 반대로 반복 횟수를 늘리면 정확도는 올라가지만 성능효율은 낮아지게 됩니다. 이 간단한 예제에서는 많은 반복이 필요 없으므로, 다음 반복 횟수 정도가 적당합니다. 

	int32 velocityIterations = 6;
	int32 positionIterations = 2;

시간 단계와 반복 횟수는 완전히 관계가 없음을 명심하기 바랍니다. 반복은 단순한 하위 단계가 아닙니다. 한개의 solver 반복은 한개의 시간 단계 내에서 모든 제약 조건을 통과하기 위한 한 단계입니다. 한개의 시간 단계 내에 서 여러번의 반복을 할 수 도 있는 것 입니다. 

이제 시뮬레이션 루프를 시작할 준비가 되었습니다. 이제 시뮬레이션 루프가 본 게임 내 게임 루프에 합쳐질 수 있습니다. 게임 루프 내에서 b2World:Step 를 부르도록 합니다. 일반적으로 프레임 레이트와 적절하게 정한 시간단계에 따라 한 번씩 불러주면 되겠습니다.
Hello World 프로그램은 간단한 구조이므로, 시각적인 출력물은 없습니다. 코드는 오로지 dynamic 속성의 강체의 위치와 회전에 대한 정보를 콘솔창에 출력할 뿐입니다. 1초간 60번으로 시간 단계를 설정하고 시뮬레이션을 실행해 보도록 하겠습니다.
	
	for (int32 i = 0; i < 60; ++i)
	{
    	world.Step(timeStep, velocityIterations, positionIterations);

    	b2Vec2 position = body->GetPosition();

	    	float32 angle = body->GetAngle();
	    	printf("%4.2f %4.2f %4.2f\n", position.x, position.y, angle);
	}

콘솔창에 보이는 결과는 상자가 바닥에 떨어지는 것을 나타냅니다. 다음과 같을 것 입니다.

	0.00 4.00 0.00
	0.00 3.99 0.00
	0.00 3.98 0.00
	...
	0.00 1.25 0.00
	0.00 1.13 0.00
	0.00 1.01 0.00

# 2.5 Cleanup

world가 현재 범위를 벗어나거나 참조하고 있는 포인터를 제거하고자 한다면, 메모리에 상주한 강체, fixture, joint 관련 내용이 모두 해제 됩니다. 매우 쉽고, 성능 향상에도 도움이 됩니다. 해당 포인터는 더이상 사용이 불가하므로 반드시 null 값을 주어야 합니다.


# 2.6 The Testbed

HelloWorld 예제를 이해했다면, Box2D testbed 를 보기 바랍니다. testbed 는 유닛 테스트 프레임웍이며, 다음과 같은 데모 환경을 제공합니다. 

Once you have conquered the HelloWorld example, you should start looking at Box2D's testbed. The testbed is a unit-testing framework and demo environment. Here are some of the features:

* 패닝, 주밍이 되는 카메라 
* 마우스를 통해 dynamic 속성 강체에 모양 설정가능 
* 테스트를 위한 확장 환경
* 테스트 선택을 위한 GUI, 파라미터 설정, 디버그 관련 그리기 옵션
* 정지 및 단독 시뮬레이션
* 문자 렌더링

testbed 는 Box2D 에서 사용되는 여러 테스트 케이스 및 예제를 가지고 있습니다. Box2D 를 익히고자 한다면 반드시 해당 프로젝트를 연구해보길 바랍니다.
 
test는 freeglut 및 GLUI로 작성되었습니다. testbed 는 Box2D의 라이브러리가 아니며, Box2D 는 렌더링용이 아닙니다. HelloWorld 에서 보았듯이 Box2D 를 사용하는데 렌더러는 필요하지 않습니다.
