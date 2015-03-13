# Chapter 10 World Class

## About

b2World 클래스는 강체와 joint를 포함합니다. 해당 클래스는 물리연산과 관련된 모든 부분을 책임지며, AABB관련 질의 및 레이캐스트 같은 비동기질의를 허용합니다. 아마도 Box2D와 사용자간의 상호작용은 해당 객체를 통해서 이루어질 것입니다.

## Creating and Destroying a World

world를 생성하는것은 매우 간단합니다. 중력에 대한 벡터값과 강체가 잠자기 모드로 들어갈 수 있는지를 정하는 플래그값만 아규먼트로 넘겨주면 됩니다. 일반적으로 new를 통해 생성하고 delete를 통해 소멸할 수 있습니다.

	b2World* myWorld = new b2World(gravity, doSleep);
	... do stuff ...
	delete myWorld;

## Using a World

world 클래스는 강체 및 joint를 생성, 소멸하기 위한 팩토리 메소드를 가지고 있습니다. 해당 부분에 대한 자세한 얘기는 강체 및 joint 관련 챕터를 참조하기 바랍니다. 여기서는 b2World와 상호작용하는 부분에 대해서 기술합니다.

## Simulation

world 클래스는 물리엔진속 물리연산을 이끄는 존재입니다. time step 및 속도, 위치에 대한 반복 횟수를 설정합니다. 다음과 같습니다:

	float32 timeStep = 1.0f / 60.f;
	int32 velocityIterations = 10;
	int32 positionIterations = 8;
	myWorld->Step(timeStep, velocityIterations, positionIterations);

매 time step 이후, 강체 및 joint 에 대한 정보를 검사할 수 있습니다. 대체로 위치값 및 강체에 데이터를 가져다 직접 actor의 정보를 업데이트하고 그리는 일을 할 것 입니다. time step은 게임이 진행되는 동안 언제든 실행 가능하지만, 일이 돌아가는 순서에 대해서는 주의하길 바랍니다. 일례로, 새로 생긴 강체가 해당 프레임 내에서 충돌하는 지를 알고 싶다면 time step 이전에 강체를 생성해야만 합니다.

HelloWorld 예제에서 기술하였는데, 반드시 time step 값은 고정할 필요가 있습니다. time step 값을 크게 잡으면 초당 프레임이 낮아져 성능향상에 도움이 됩니다. 하지만 일반적으로 1/30초 보다 크게 잡지는 말길 바랍니다. 1/60초로 time step을 잡았을 때가 물리연산에 대한 품질을 기대할만 합니다.

반복 횟수는 제한관련 solver가 world에 존재하는 contact와 joint를 훑어보는지에 대한 횟수를 정하는 것이다. 횟수가 많은 것이 더 나은 물리연산을 제공하겠지만 반복 횟수를 늘리는 것과 time step을 적게 만드는 것을 저울질 하지는 마시기 바랍니다. 60Hz에서 10회의 반복 하는 것이 30Hz에서 20회 반복하는 것보다 훨씬 낫습니다.

time step 이후, 강체에 가해진 힘을 제거해 줘야 합니다. 이는 b2World:ClearForces 메소드를 불러서 해결할 수 있습니다. 이는 같은 역장내에 있는 여러 하위 time step에 대해서도 적용할 수 있습니다.

	myWorld->ClearForces();

## Exploring the World

world 클래스는 강체, contact, joint의 저장소입니다. 사용자는 반복문을 통해서 상기한 개체를 가져오는 것이 가능합니다. 일례로, 다음과 같이 하면 world내의 모든 강체를 깨우는 결과를 얻을 수 있습니다:

	for (b2Body* b = myWorld->GetBodyList(); b; b = b->GetNext())
	{
    	b->SetAwake(true);
	}

불행히도 실제 구현코드는 약간 더 복잡합니다. 다음과 같이 작성하면 문제가 발생합니다:

	for (b2Body* b = myWorld->GetBodyList(); b; b = b->GetNext())
	{
    	GameActor* myActor = (GameActor*)b->GetUserData();
    	if (myActor->IsDead())
    	{
        	myWorld->DestroyBody(b); // ERROR: now GetNext returns garbage.
    	}
	}
	
강체가 소멸하기 전까지는 모두 정상인 상태이지만, 일단 강체가 소멸한다면 그 다음을 가르키는 포인터가 가르키는 곳에서 아무 것도 가져올 수 없는 상태가 됩니다. 그러므로 b2Body:GetNext()를 하면 사용할 수 없는 쓰레기값이 넘어옵니다. 해결책은 강체를 소멸시키기 전에 그 다음에 있는 포인터를 복사해두는 것입니다.

	b2Body* node = myWorld->GetBodyList();
	while (node)
	{
    	b2Body* b = node;
    	node = node->GetNext();
    	
    	GameActor* myActor = (GameActor*)b->GetUserData();
    	if (myActor->IsDead())
    	{
        	myWorld->DestroyBody(b);
    	}
	}
	
안전하게 현재 강체를 제거할 수 있게 해줍니다. 하지만 여러 개의 강체를 한 번에 소멸시키는 함수를 불러야 할 필요가 있을 수도 있습니다. 이런 경우, 매우 조심해야 합니다. 해결책은 구현하는 사람마다 다르겠지만, 편의상 한 가지 해법을 제시합니다.

	b2Body* node = myWorld->GetBodyList();
	while (node)
	{
    	b2Body* b = node;
    	node = node->GetNext();
   
    	GameActor* myActor = (GameActor*)b->GetUserData();
    	if (myActor->IsDead())
    	{
        	bool otherBodiesDestroyed = GameCrazyBodyDestroyer(b);
        	if (otherBodiesDestroyed)
        	{
            	node = myWorld->GetBodyList();
        	}
    	}
	}
	
이 작업을 위한 전제조건은, GameCrazyBodyDestroyer가 강체의 소멸에 대해 항상 정확한 결과를 주어야 한다는 것입니다.

## AABB Queries

때때로 특정 범위내의 모든 shape을 결정해야 할 수 있습니다. b2World 클래스는 광범위한 데이터 구조체를 사용하는데 있어 접근 횟수가 log(N)정도 되는 빠른 함수를 제공합니다. b2QueryCallback을 구현하여 world 좌표계에 대한 AABB를 제공할 수 있습니다. world는 해당 구현체를 불러 AABB 질의에 겹치는 AABB를 가진 fixture를 가져옵니다. 이때, 결과값을 true로 리턴하면 질의를 이어갑니다. 하기의 코드는 지정된 AABB에 대해 교차할 가능성이 있는 fixture를 모두 찾아 해당 강체를 깨우는 것입니다.

	class MyQueryCallback : public b2QueryCallback
	{
	public:
    	bool ReportFixture(b2Fixture* fixture)
    	{
        	b2Body* body = fixture->GetBody();
        	body->SetAwake(true);
       
       		// Return true to continue the query.
        	return true;
    	}
	};

	...

	MyQueryCallback callback;
	b2AABB aabb;
	aabb.lowerBound.Set(-1.0f, -1.0f);
	aabb.upperBound.Set(1.0f, 1.0f);
	myWorld->Query(&callback, aabb);
	
콜백의 순서에 대해 어떠한 추정도 할 수 없습니다.

## Ray Casts

시선, 조준선, 사격에 대한 기능을 구현하기 위해 ray cast 기능을 사용할 수 있습니다. 콜백 클래스를 구현하고 시작과 끝 점을 지정하는 것으로 ray cast를 쓸 수 있습니다. world 클래스는 이렇게 구현된 클래스를 불러 각각의 fixture에 대해 빛이 닿는지를 체크합니다. 해당 콜백 클래스에는 교차점, 법선벡터 단위, 빛의 단편적인 길이를 제공해야 합니다. 또한 콜백의 순서에 대해 어떠한 추정도 할 수 없습니다.

ray cast의 지속 여부는 fraction 값을 반환함으로 결정할 수 있습니다. 해당 값을 0으로 반환하면 ray cast가 종료되어야 함을 의미합니다. 1이 반환되는 것은 충돌이 일어나지 않는다면 ray cast가 계속 되어야 함을 의미합니다. 아규먼트로 들어오는 fraction값을 반환한다면, 빛은 해당 충돌점에서 딱 잘려버릴 것 입니다. 그러므로 모든 shape 에 대해 ray cast를 하거나 적절한 fraction 값을 리턴해 가까운 객체에만 ray cast를 할 수 있습니다.

fraction값을 -1로 리턴한다면 해당 fixture를 필터링 하여 존재하지 않는 것처럼 처리합니다. 

다음은 해당 내용의 예제입니다:

	// This class captures the closest hit shape.
	class MyRayCastCallback : public b2RayCastCallback
	{
	public:
    	MyRayCastCallback()
    	{
        	m_fixture = NULL;
    	}
   
    	float32 ReportFixture(b2Fixture* fixture, const b2Vec2& point,
        	                      const b2Vec2& normal, float32 fraction)
    	{
        	m_fixture = fixture;
        	m_point = point;
        	m_normal = normal;
        	m_fraction = fraction;
        	return fraction;
    	}
   
    	b2Fixture* m_fixture;
    	b2Vec2 m_point;
    	b2Vec2 m_normal;
    	float32 m_fraction;
	};

	MyRayCastCallback callback;
	b2Vec2 point1(-1.0f, 0.0f);
	b2Vec2 point2(3.0f, 1.0f);
	myWorld->RayCast(&callback, point1, point2);


> 주의 <br>
> 반올림 오류로 인해 ray cast가 정적 환경하의 다각형 사이의 작은 틈으로 스며들 수 있습니다. 해당 프로그램에서 이러한 상황을 막고자 한다면, 해당 다각형의 크기를 키워주시기 바랍니다.

	void SetLinearVelocity(const b2Vec2& v);
	b2Vec2 GetLinearVelocity() const;
	void SetAngularVelocity(float32 omega);
	float32 GetAngularVelocity() const;

## Forces and Impulses

강체에 힘, 회전력, 충격을 가할 수 있습니다. 힘과 충격을 가할 때는 world 좌표계상에서 어느 위치에서 가하는지를 제공해야 합니다. 회전력은 질량중심점 기준으로 발생하는 결과를 낳을 수 있습니다.

	void ApplyForce(const b2Vec2& force, const b2Vec2& point);
	void ApplyTorque(float32 torque);
	void ApplyLinearImpulse(const b2Vec2& impulse, const b2Vec2& point);
	void ApplyAngularImpulse(float32 impulse);

힘, 회전력, 충격을 가하는 것은 강체를 깨우는 역할도 합니다. 지속적으로 힘을 가하면서도 강체는 계속 잠자기 모드에 두며 성능을 향상시키는 것처럼 강체를 깨우는 것을 원치 않는 경우도 있을 수 있습니다. 이런 경우, 다음과 같이 하면 됩니다.

	if (myBody->IsAwake() == true)
	{
    	myBody->ApplyForce(myForce, myPoint);
	}
	
## Coordinate Transformations

강체 클래스는 지역좌표계와 world좌표계 사이에서 위치와 벡터를 변경하는데 유용한 함수를 가지고 있습니다. 해당 기능에 대해 잘 이해가 가지 않는다면, 짐 밴 버스와 라스 비숍의 "Essential Mathematics for Games and Interactive Applications"을 읽어보길 바랍니다. 이 기능은 확실히 유용합니다.(역주: 지앤선에서 "게임&인터랙티브 애플리케이션을 위한 수학"이라는 제목으로 2008년에 출간되었습니다.)

	b2Vec2 GetWorldPoint(const b2Vec2& localPoint);
	b2Vec2 GetWorldVector(const b2Vec2& localVector);
	b2Vec2 GetLocalPoint(const b2Vec2& worldPoint);
	b2Vec2 GetLocalVector(const b2Vec2& worldVector);
	
## Lists

강체의 fixture에 대해서도 반복문을 돌리는 것이 가능합니다. fixture 안에 존재하는 user data에 접근하고자 할때 매우 유용합니다.

	for (b2Fixture* f = body->GetFixtureList(); f; f = f->GetNext())
	{
    	MyFixtureData* data = (MyFixtureData*)f->GetUserData();
    	... do something with data ...
	}
	

강체가 가진 여러 개의 joint에 대해서도 유사한 방법으로 반복문을 사용할 수 있습니다.

또한, 강체는 contact집합에 대한 정보도 제공합니다. 현재 contact의 정보를 얻기 위해 사용할 수 있습니다. 지난 time step에 존재하던 모든 contact에 대한 정보를 모두 제공하는 것은 아니므로 주의를 할 필요가 있습니다.