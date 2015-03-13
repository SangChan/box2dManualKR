# Chapter 10 World Class

## About

b2World 클래스는 강체와 joint를 포함합니다. 해당 클래스는 물리연산과 관련된 모든 부분을 책임지며, AABB관련 질의 및 레이캐스트 같은 비동기질의를 허용합니다. 아마도 Box2D와 사용자간의 상호작용은 해당 객체를 통해서 이루어질 것입니다.

The b2World class contains the bodies and joints. It manages all aspects of the simulation and allows for asynchronous queries (like AABB queries and ray-casts). Much of your interactions with Box2D will be with a b2World object.

## Creating and Destroying a World

world를 생성하는것은 매우 간단합니다. 중력에 대한 벡터값과 강체가 잠자기 모드로 들어갈 수 있는지를 정하는 플래그값만 아규먼트로 넘겨주면 됩니다. 일반적으로 new를 통해 생성하고 delete를 통해 소멸할 수 있습니다.

Creating a world is fairly simple. You just need to provide a gravity vector and a Boolean indicating if bodies can sleep. Usually you will create and destroy a world using new and delete.

	b2World* myWorld = new b2World(gravity, doSleep);
	... do stuff ...
	delete myWorld;

## Using a World

world 클래스는 강체 및 joint를 생성, 소멸하기 위한 팩토리 메소드를 가지고 있습니다. 해당 부분에 대한 자세한 얘기는 강체 및 joint 관련 챕터를 참조하기 바랍니다. 여기서는 b2World와 상호작용하는 부분에 대해서 기술합니다.

The world class contains factories for creating and destroying bodies and joints. These factories are discussed later in the sections on bodies and joints. There are some other interactions with b2World that I will cover now.

## Simulation

world 클래스는 물리엔진속 물리연산을 이끄는 존재입니다. time step 및 속도, 위치에 대한 반복 횟수를 설정합니다. 다음과 같습니다:

The world class is used to drive the simulation. You specify a time step and a velocity and position iteration count. For example:

	float32 timeStep = 1.0f / 60.f;
	int32 velocityIterations = 10;
	int32 positionIterations = 8;
	myWorld->Step(timeStep, velocityIterations, positionIterations);

매 time step 이후, 강체 및 joint 에 대한 정보를 검사할 수 있습니다. 대체로 위치값 및 강체에 데이터를 가져다 직접 actor의 정보를 업데이트하고 그리는 일을 할 것 입니다. time step은 게임이 진행되는 동안 언제든 실행 가능하지만, 일이 돌아가는 순서에 대해서는 주의하길 바랍니다. 일례로, 새로 생긴 강체가 해당 프레임 내에서 충돌하는 지를 알고 싶다면 time step 이전에 강체를 생성해야만 합니다.

After the time step you can examine your bodies and joints for information. Most likely you will grab the position off the bodies so that you can update your actors and render them. You can perform the time step anywhere in your game loop, but you should be aware of the order of things. For example, you must create bodies before the time step if you want to get collision results for the new bodies in that frame.

HelloWorld 예제에서 기술하였는데, 반드시 time step 값은 고정할 필요가 있습니다. time step 값을 크게 잡으면 초당 프레임이 낮아져 성능향상에 도움이 됩니다. 하지만 일반적으로 1/30초 보다 크게 잡지는 말길 바랍니다. 1/60초로 time step을 잡았을 때가 물리연산에 대한 품질을 기대할만 합니다.

As I discussed above in the HelloWorld tutorial, you should use a fixed time step. By using a larger time step you can improve performance in low frame rate scenarios. But generally you should use a time step no larger than 1/30 seconds. A time step of 1/60 seconds will usually deliver a high quality simulation.

반복 횟수는 제한관련 solver가 world에 존재하는 contact와 joint를 훑어보는지에 대한 횟수를 정하는 것이다. 횟수가 많은 것이 더 나은 물리연산을 제공하겠지만 반복 횟수를 늘리는 것과 time step을 적게 만드는 것을 저울질 하지는 마시기 바랍니다. 60Hz에서 10회의 반복 하는 것이 30Hz에서 20회 반복하는 것보다 훨씬 낫습니다.

The iteration count controls how many times the constraint solver sweeps over all the contacts and joints in the world. More iteration always yields a better simulation. But don't trade a small time step for a large iteration count. 60Hz and 10 iterations is far better than 30Hz and 20 iterations.

time step 이후, 강체에 가해진 힘을 제거해 줘야 합니다. 이는 b2World:ClearForces 메소드를 불러서 해결할 수 있습니다. 이는 같은 역장내에 있는 여러 하위 time step에 대해서도 적용할 수 있습니다.

After stepping, you should clear any forces you have applied to your bodies. This is done with the command b2World::ClearForces. This lets you take multiple sub-steps with the same force field.

	myWorld->ClearForces();

## Exploring the World

world 클래스는 강체, contact, joint의 저장소입니다. 사용자는 반복문을 통해서 상기한 개체를 가져오는 것이 가능합니다. 일례로, 다음과 같이 하면 world내의 모든 강체를 깨우는 결과를 얻을 수 있습니다:

The world is a container for bodies, contacts, and joints. You can grab the body, contact, and joint lists off the world and iterate over them. For example, this code wakes up all the bodies in the world:

	for (b2Body* b = myWorld->GetBodyList(); b; b = b->GetNext())
	{
    	b->SetAwake(true);
	}

불행히도 실제 구현코드는 약간 더 복잡합니다. 다음과 같이 작성하면 문제가 발생합니다:

Unfortunately real programs can be more complicated. For example, the following code is broken:

	for (b2Body* b = myWorld->GetBodyList(); b; b = b->GetNext())
	{
    	GameActor* myActor = (GameActor*)b->GetUserData();
    	if (myActor->IsDead())
    	{
        	myWorld->DestroyBody(b); // ERROR: now GetNext returns garbage.
    	}
	}
	
강체가 소멸하기 전까지는 모두 정상인 상태이지만, 일단 강체가 소멸한다면 그 다음을 가르키는 포인터가 가르키는 곳에서 아무 것도 가져올 수 없는 상태가 됩니다. 그러므로 b2Body:GetNext()를 하면 사용할 수 없는 쓰레기값이 넘어옵니다. 해결책은 강체를 소멸시키기 전에 그 다음에 있는 포인터를 복사해두는 것입니다.

Everything goes ok until a body is destroyed. Once a body is destroyed, its next pointer becomes invalid. So the call to b2Body::GetNext() will return garbage. The solution to this is to copy the next pointer before destroying the body.

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

This safely destroys the current body. However, you may want to call a game function that may destroy multiple bodies. In this case you need to be very careful. The solution is application specific, but for convenience I'll show one method of solving the problem.

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

Obviously to make this work, GameCrazyBodyDestroyer must be honest about what it has destroyed.

## AABB Queries

때때로 특정 범위내의 모든 shape을 결정해야 할 수 있습니다. b2World 클래스는 광범위한 데이터 구조체를 사용하는데 있어 접근 횟수가 log(N)정도 되는 빠른 함수를 제공합니다. b2QueryCallback을 구현하여 world 좌표계에 대한 AABB를 제공할 수 있습니다. world는 해당 구현체를 불러 AABB 질의에 겹치는 AABB를 가진 fixture를 가져옵니다. 이때, 결과값을 true로 리턴하면 질의를 이어갑니다. 하기의 코드는 지정된 AABB에 대해 교차할 가능성이 있는 fixture를 모두 찾아 해당 강체를 깨우는 것입니다.

Sometimes you want to determine all the shapes in a region. The b2World class has a fast log(N) method for this using the broad-phase data structure. You provide an AABB in world coordinates and an implementation of b2QueryCallback. The world calls your class with each fixture whose AABB overlaps the query AABB. Return true to continue the query, otherwise return false. For example, the following code finds all the fixtures that potentially intersect a specified AABB and wakes up all of the associated bodies.

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

You cannot make any assumptions about the order of the callbacks.

## Ray Casts

시선, 조준선, 사격에 대한 기능을 구현하기 위해 ray cast 기능을 사용할 수 있습니다. 콜백 클래스를 구현하고 시작과 끝 점을 지정하는 것으로 ray cast를 쓸 수 있습니다. world 클래스는 이렇게 구현된 클래스를 불러 각각의 fixture에 대해 빛이 닿는지를 체크합니다. 해당 콜백 클래스에는 교차점, 법선벡터 단위, 빛의 단편적인 길이를 제공해야 합니다. 또한 콜백의 순서에 대해 어떠한 추정도 할 수 없습니다.

You can use ray casts to do line-of-sight checks, fire guns, etc. You perform a ray cast by implementing a callback class and providing the start and end points. The world class calls your class with each fixture hit by the ray. Your callback is provided with the fixture, the point of intersection, the unit normal vector, and the fractional distance along the ray. You cannot make any assumptions about the order of the callbacks.

You control the continuation of the ray cast by returning a fraction. Returning a fraction of zero indicates the ray cast should be terminated. A fraction of one indicates the ray cast should continue as if no hit occurred. If you return the fraction from the argument list, the ray will be clipped to the current intersection point. So you can ray cast any shape, ray cast all shapes, or ray cast the closest shape by returning the appropriate fraction.

You may also return of fraction of -1 to filter the fixture. Then the ray cast will proceed as if the fixture does not exist.

Here is an example:

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

> Caution <br>
> Due to round-off errors, ray casts can sneak through small cracks between polygons in your static environment. If this is not acceptable in your application, please enlarge your polygons slightly.

	void SetLinearVelocity(const b2Vec2& v);
	b2Vec2 GetLinearVelocity() const;
	void SetAngularVelocity(float32 omega);
	float32 GetAngularVelocity() const;

## Forces and Impulses
You can apply forces, torques, and impulses to a body. When you apply a force or an impulse, you provide a world point where the load is applied. This often results in a torque about the center of mass.

	void ApplyForce(const b2Vec2& force, const b2Vec2& point);
	void ApplyTorque(float32 torque);
	void ApplyLinearImpulse(const b2Vec2& impulse, const b2Vec2& point);
	void ApplyAngularImpulse(float32 impulse);
	
Applying a force, torque, or impulse wakes the body. Sometimes this is undesirable. For example, you may be applying a steady force and want to allow the body to sleep to improve performance. In this case you can use the following code.

	if (myBody->IsAwake() == true)
	{
    	myBody->ApplyForce(myForce, myPoint);
	}
	
## Coordinate Transformations
The body class has some utility functions to help you transform points and vectors between local and world space. If you don't understand these concepts, please read "Essential Mathematics for Games and Interactive Applications" by Jim Van Verth and Lars Bishop. These functions are efficient (when inlined).

	b2Vec2 GetWorldPoint(const b2Vec2& localPoint);
	b2Vec2 GetWorldVector(const b2Vec2& localVector);
	b2Vec2 GetLocalPoint(const b2Vec2& worldPoint);
	b2Vec2 GetLocalVector(const b2Vec2& worldVector);
	
## Lists

You can iterate over a body's fixtures. This is mainly useful if you need to access the fixture's user data.

	for (b2Fixture* f = body->GetFixtureList(); f; f = f->GetNext())
	{
    	MyFixtureData* data = (MyFixtureData*)f->GetUserData();
    	... do something with data ...
	}
	
You can similarly iterate over the body's joint list.

The body also provides a list of associated contacts. You can use this to get information about the current contacts. Be careful, because the contact list may not contain all the contacts that existed during the previous time step.