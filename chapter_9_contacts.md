# Chapter 9 Contacts

# 9.1 About

contact는 Box2D에서 두 개의 fixture간의 충돌을 관리하기 위해 만드는 객체입니다. fixture가 shape끼리 연결된 자식을 가지고 있다면, contact는 각 자식들에 대해 존재합니다. b2Contact를 contact클래스는 다른 종류의 fixture 간 접촉을 관리 하기 위해 존재합니다. 일례로, 다각형과 다각형의 충돌을 관리해주는 contact 클래스가 있고 원과 원의 충돌을 관리해주는 클래스가 따로 있습니다.

Contacts are objects created by Box2D to manage collision between two fixtures. If the fixture has children, such as a chain shape, then a contact exists for each relevant child. There are different kinds of contacts, derived from b2Contact, for managing contact between different kinds of fixtures. For example there is a contact class for managing polygon-polygon collision and another contact class for managing circle-circle collision.

다음은 contact와 관련된 용어입니다.

Here is some terminology associated with contacts.

## contact point

contact point는 두 shape이 닿는 지점을 말합니다. Box2D는 아주 작은 단위의 지점에 근사치로 접촉합니다.

A contact point is a point where two shapes touch. Box2D approximates contact with a small number of points.

## contact normal

contact normal은 한 개의 shape에서 다른 것을 가르키는 단위벡터입니다. contact normal은 fixtureA 에서 fixtureB로 향하는 규칙을 따릅니다.

A contact normal is a unit vector that points from one shape to another. By convention, the normal points from fixtureA to fixtureB.

## contact separation

분리는 침투의 반대이며. 분리는 shape가 겹치는 것에 부정적입니다. 향후의 Box2D에서는 contact point가 겹칠수 있도록 하는 것도 가능하니 contact point가 선언될 때, 해당 값을 체크해두는 것을 권장합니다.

Separation is the opposite of penetration. Separation is negative when shapes overlap. It is possible that future versions of Box2D will create contact points with positive separation, so you may want to check the sign when contact points are reported.

## contact manifold

두 개의 볼록다각형에는 2개 이상의 접점이 있을 수 있습니다. 각각의 접점이 동일한 contact normal을 사용한다면  contact mainfold 내부에 그룹 지어집니다. 해당 값은 접촉이 일어날 수 있는 연속적인 구획상의 근사치입니다.

Contact between two convex polygons may generate up to 2 contact points. Both of these points use the same normal, so they are grouped into a contact manifold, which is an approximation of a continuous region of contact.

## normal impulse

수직력은 물체가 접점에서 충돌할 때 shape 끼리 침투되는 것을 방지하기 위해 작용하는 힘입니다. 편의상, Box2D는 impulse 형태로 작용하는데, 이는 수직충격량이 수직력에 world의 time step값을 곱한 값이기 때문입니다.

The normal force is the force applied at a contact point to prevent the shapes from penetrating. For convenience, Box2D works with impulses. The normal impulse is just the normal force multiplied by the time step.

## tangent impulse

접선력은 접점에서 마찰을 연산하기 위해 생성됩니다. 편의상 impulse 형태로 저장됩니다.

The tangent force is generated at a contact point to simulate friction. For convenience, this is stored as an impulse.

## contact ids

Box2D는 특정 time step에서 바로 다음 time step을 추측하고자 접촉력을 다시 사용하는 것을 시도합니다. 이를 위해 Box2D는 contact ids를 사용해 time step에 대해 접점을 맞춰줍니다. contact ids는 다른 강체로 부터 생기는 한 접점을 구별하는데 도움이 되는 기하학적 속성을 포함하고 있습니다.

Box2D tries to re-use the contact force results from a time step as the initial guess for the next time step. Box2D uses contact ids to match contact points across time steps. The ids contain geometric features indices that help to distinguish one contact point from another.

contact는 두 fixture의 AABB(Axis-aligned bounding box)가 겹칠 때 생성됩니다. 때때로 충돌 필터링 기능이 contact가 생성되는 것을 방지하기도 합니다. contact는 AABB가 겹치는 기능을 중단할 때 소멸됩니다.

Contacts are created when two fixture’s AABBs overlap. Sometimes collision filtering will prevent the creation of contacts. Contacts are destroyed with the AABBs cease to overlap.

그러므로 AABB에서 자체적으로 해당 상자를 딱히 건드리지 않더라도 contact가 생성이 될 수 있으며, 해당 정보를 수집할 수도 있습니다. 당연하지만, 이건 마치 "닭이 먼저냐 달걀이 먼저냐" 하는 문제라고 볼 수 있습니다. 사용자 입장에서는 contact 객체가 충돌을 분석하기 위해 생성될 필요가 있다는 것을 알 수가 없습니다. 사용자가 shape를 따로 건드리지 않는다면 contact를 제거하는게 옳겠으나, AABB가 겹치는 기능을 중단하는 것을 기다리는 것도 방법입니다. Box2D는 시스템 정보를 캐쉬하도록 하여 성능을 향상할 수 있으므로 후자를 선택하였습니다.

So you might gather that there may be contacts created for fixtures that are not touching (just their AABBs). Well, this is correct. It's a "chicken or egg" problem. We don't know if we need a contact object until one is created to analyze the collision. We could delete the contact right away if the shapes are not touching, or we can just wait until the AABBs stop overlapping. Box2D takes the latter approach because it lets the system cache information to improve performance.

# 9.2 Contact Class

앞서 서술하였다시피, contact 클래스는 Box2D가 생성하고 소멸합니다. contact 객체는 사용자가 생성하는 것이 아닙니다만, 해당 클래스에 접근해 상호작용할 수 있습니다.

As mentioned before, the contact class is created and destroyed by Box2D. Contact objects are not created by the user. However, you are able to access the contact class and interact with it.

contact mainfold 의 raw 데이터에 접근할 수도 있습니다:

You can access the raw contact manifold:

	b2Manifold* GetManifold();
	const b2Manifold* GetManifold() const;

잠재적으로 mainfold를 수정하는 것도 가능하지만, 고급사용법을 위해서 일반적 용도로는 사용할 수 없도록 하였습니다.

You can potentially modify the manifold, but this is generally not supported and is for advanced usage.

b2WorldManifold를 얻어올 수 있는 편의성 함수도 있습니다.

There is a helper function to get the b2WorldManifold:

	void GetWorldManifold(b2WorldManifold* worldManifold) const;

이는 강체의 현재위치를 접점의 world 상 위치에 투영하여 연산하는데 사용됩니다. sensor는 manifold를 생성하지 않으므로 다음과 같이 사용하십시요:

This uses the current positions of the bodies to compute world positions of the contact points.
Sensors do not create manifolds, so for them use:

	bool touching = sensorContact->IsTouching();

해당 함수는 sensor가 아닌 경우에도 작동합니다.

This function also works for non-sensors.

contact로부터 fixture를 얻을 수도 있습니다. 이를 통해 강체를 얻어올 수 있습니다.

You can get the fixtures from a contact. From those you can get the bodies.

	b2Fixture* fixtureA = myContact->GetFixtureA();
	b2Body* bodyA = fixtureA->GetBody();
	MyActor* actorA = (MyActor*)bodyA->GetUserData();

contact를 쓰지 못하게 할 수 있으며, 이는 b2ContactListener::PreSolve 이벤트 안에서만 가능합니다. 하단에 기술하겠습니다.

You can disable a contact. This only works inside the b2ContactListener::PreSolve event, discussed below.

# 9.3 Accessing Contacts

contact에 접근할 수 있는 방법은 여러가지 있습니다. world 나 body 구조체를 통해 직접 접근할 수 있습니다. contact 리스너를 만들어 접근할 수도 있습니다.

You can get access to contacts in several ways. You can access the contacts directly on the world and body structures. You can also implement a contact listener.

반복문을 통해 world상에 존재하는 모든 contact에 접근할 수도 있습니다.

You can iterate over all contacts in the world:

	for (b2Contact* c = myWorld->GetContactList(); c; c = c->GetNext())
	{
		// process c
	}
	
또한 반복문을 통해 강체에 존재하는 모든 contact에 접근하는 것도 가능합니다. contact edge 구조체에 그래프로 저장되어 있습니다.

You can also iterate over all the contacts on a body. These are stored in a graph using a contact edge structure.

	for (b2ContactEdge* ce = myBody->GetContactList(); ce; ce = ce->next)
	{
		b2Contact* c = ce->contact;
		// process c
	}

contact 리스너를 통해 contact에 접근하는 것은 하단에 기술하겠습니다.

You can also access contacts using the contact listener that is described below.

> 주의<br>
> b2World와 b2Body에 접근하는 동안, time step 중간에 몇몇 contact를 놓칠 가능성이 있습니다. 그러므로 b2ContactListener를 사용하는 것이 가장 정확한 결과를 얻을 수 있는 방법입니다.

> Caution<br>
> Accessing contacts off b2World and b2Body may miss some transient contacts that occur in the middle of the time step. Use b2ContactListener to get the most accurate results.

# 9.4 Contact Listener

b2ContactListener를 구현하여 contact에 대한 정보를 수신할 수 있습니다. contact 리스너는 begin, end, pre-solve, post-solve 같은 이벤트를 지원합니다.

You can receive contact data by implementing b2ContactListener. The contact listener supports several events: begin, end, pre-solve, and post-solve.

	class MyContactListener : public b2ContactListener
	{
	public:
		void BeginContact(b2Contact* contact)
		{ /* handle begin event */ }
		
		void EndContact(b2Contact* contact) 
		{ /* handle end event */ }
		
		void PreSolve(b2Contact* contact, const b2Manifold* oldManifold)
		{ /* handle pre-solve event */ }
		
		void PostSolve(b2Contact* contact, const b2ContactImpulse* impulse)  
		{ /* handle post-solve event */ }
	};

> 주의<br>
> b2ContactListener로 보내는 포인터의 참조값을 저장해두지 마십시요. 대신 contact point에 대한 정보를 사용자의 버퍼에 복사본을 만드십시요. 하단에 예제가 있습니다.

> Caution<br>
> Do not keep a reference to the pointers sent to b2ContactListener. Instead make a deep copy of the contact point data into your own buffer. The example below shows one way of doing this.

런타임 상에서 리스너의 인스턴스는 b2World::SetContactListener를 이용해 만들고 등록하는 것이 가능합니다. world 객체가 존재하는 동안 리스너도 존재해야 합니다. 

At run-time you can create an instance of the listener and register it with b2World::SetContactListener. Be sure your listener remains in scope while the world object exists.

## Begin Contact Event

두 개의 fixture가 겹치기 시작할 때 불립니다. sensor와 sensor가 아닌 것을 위해 불립니다. time step 도중에만 발생합니다.

This is called when two fixtures begin to overlap. This is called for sensors and non-sensors. This event can only occur inside the time step.
 
## End Contact Event

두 개의 fixture가 겹치는 것을 중단할 때 불립니다. sensor와 sensor가 아닌 것을 위해 불립니다. 강체가 소멸하거나, time step을 벗어났을 때 불릴 수 있습니다.

This is called when two fixtures cease to overlap. This is called for sensors and non-sensors. This may be called when a body is destroyed, so this event can occur outside the time step.

## Pre-Solve Event

충돌이 감지가 되고 난 후이지만, 충돌이 완료되기는 전인 시점에 불립니다. 현재 상태에 따라서 contact를 사용하지 않도록 할 수 있는 기회가 주어지는 것입니다. 예를 들어, b2Contact::SetEnabled(false)가 부르고 해당 콜백을 이용해 한 쪽 방향으로만 작동하는 발판을 만들 수 있습니다. contact는 매 time step에 충돌관련 연산을 다시 사용가능하게 하므로, 매번 사용 못하도록 해줄 필요가 있습니다. pre-solve 이벤트는 연속적인 충돌 검색을 하는 동안 contact 별로 time step 당 여러 번 발생할 수 있습니다.

This is called after collision detection, but before collision resolution. This gives you a chance to disable the contact based on the current configuration. For example, you can implement a one-sided platform using this callback and calling b2Contact::SetEnabled(false). The contact will be re-enabled each time through collision processing, so you will need to disable the contact every time-step. The pre-solve event may be fired multiple times per time step per contact due to continuous collision detection.

	void PreSolve(b2Contact* contact, const b2Manifold* oldManifold)
	{
		b2WorldManifold worldManifold;
		contact->GetWorldManifold(&worldManifold);
		if (worldManifold.normal.y < -0.5f){
			contact->SetEnabled(false);
		}
	}
	
pre-solve 이벤트는 충돌에 대한 접근 속도 및 위치 상태를 특정하기에 좋은 곳 입니다.

The pre-solve event is also a good place to determine the point state and the approach velocity of collisions.

	void PreSolve(b2Contact* contact, const b2Manifold* oldManifold
	{
		b2WorldManifold worldManifold;
		contact->GetWorldManifold(&worldManifold);
		b2PointState state1[2], state2[2];
		b2GetPointStates(state1, state2, oldManifold, contact->GetManifold());
		if (state2[0] == b2_addState) 
		{
			const b2Body* bodyA = contact->GetFixtureA()->GetBody();
			const b2Body* bodyB = contact->GetFixtureB()->GetBody();
			b2Vec2 point = worldManifold.points[0];
			b2Vec2 vA = bodyA->GetLinearVelocityFromWorldPoint(point);
			b2Vec2 vB = bodyB->GetLinearVelocityFromWorldPoint(point);
			float32 approachVelocity = b2Dot(vB – vA, worldManifold.normal);
			if (approachVelocity > 1.0f)
			{
				MyPlayCollisionSound();
			}
		}
	}

## Post-Solve Event

post-solve 이벤트는 충돌 충격 결과를 수집할 수 있는 곳 입니다. 충격에 대한 부분을 신경쓰지 않을 것이라면 pre-solve 이벤트를 구현하는 것만으로도 충분할 것 입니다.

The post solve event is where you can gather collision impulse results. If you don’t care about the impulses, you should probably just implement the pre-solve event.

contact 콜백 안에서 world상의 물리법칙을 게임로직으로 대체하려 할 때 사용하고자 할 수도 있습니다. 예를 들어, 충돌을 통해 데미지를 주고 관련된 액터 및 강체를 소멸하려고 한다고 칩시다. Box2D는 콜백 내부에서 물리법칙을 대체하는 것을 허용하지 않습니다. 왜냐하면 Box2D에서 연산 중에 객체를 소멸하는 것은 포인터가 가르키는 위치에 원하는 데이터가 존재하지 않도록 할 수 있기 때문입니다.

It is tempting to implement game logic that alters the physics world inside a contact callback. For example, you may have a collision that applies damage and try to destroy the associated actor and its rigid body. However, Box2D does not allow you to alter the physics world inside a callback because you might destroy objects that Box2D is currently processing, leading to orphaned pointers.

접점을 연산하기 위해 추천하는 방법은 버퍼에 모든 contact 정보를 넣어놓고 time step 이후에 해당 정보를 연산하는 것을 살펴보는 것입니다. 접점에 대해 time step 이후에 즉시 연산해야만 합니다. 아닐 경우, 다른 코드에서 world상의 물리연산으로 대체하여 해당 버퍼가 무의미하게 될 수 있습니다. world 상의 물리연산을 대체하며 버퍼에 있는 contact를 연산할 수 있으나, 포인터가 가르키는 곳에 contact가 존재하지 않는 고아 포인터가 저장되지 않도록 조심해야 합니다. 테스트 케이스에 고아 포인터가 생기는 것에 대한 방어코드가 있는 예제가 있습니다.

The recommended practice for processing contact points is to buffer all contact data that you care about and process it after the time step. You should always process the contact points immediately after the time step; otherwise some other client code might alter the physics world, invalidating the contact buffer. When you process the contact buffer you can alter the physics world, but you still need to be careful that you don't orphan pointers stored in the contact point buffer. The testbed has example contact point processing that is safe from orphaned pointers.

CollisionProcessing에 있는 본 코드는 contact 버퍼를 연산할 때 부모가 없는 강체를 어떻게 처리해야 하는지 보여주고 있습니다. 하단은 해당 내용을 발췌한 것입니다. 주석을 잘 읽어주시기 바랍니다. 본 코드는 contact point를 모두 모아 b2ContactPoint로 된 배열인 m_points 버퍼에 저장합니다.

This code from the CollisionProcessing test shows how to handle orphaned bodies when processing the contact buffer. Here is an excerpt. Be sure to read the comments in the listing. This code assumes that all contact points have been buffered in the b2ContactPoint array m_points.

	// We are going to destroy some bodies according to contact
	// points. We must buffer the bodies that should be destroyed
	// because they may belong to multiple contact points.
	const int32 k_maxNuke = 6;
	b2Body* nuke[k_maxNuke];
	int32 nukeCount = 0;

	// Traverse the contact buffer. Destroy bodies that
	// are touching heavier bodies.
	for (int32 i = 0; i < m_pointCount; ++i)
	{
    	ContactPoint* point = m_points + i;

    	b2Body* body1 = point->fixtureA->GetBody();
    	b2Body* body2 = point->FixtureB->GetBody();
    	float32 mass1 = body1->GetMass();
    	float32 mass2 = body2->GetMass();

    	if (mass1 > 0.0f && mass2 > 0.0f)
    	{
        	if (mass2 > mass1)
        	{
            	nuke[nukeCount++] = body1;
        	}
        	else
        	{
            	nuke[nukeCount++] = body2;
        	}

        	if (nukeCount == k_maxNuke)
        	{
            	break;
        	}
    	}
	}

	// Sort the nuke array to group duplicates.
	std::sort(nuke, nuke + nukeCount);

	// Destroy the bodies, skipping duplicates.
	int32 i = 0;
	while (i < nukeCount)
	{
    	b2Body* b = nuke[i++];
    	while (i < nukeCount && nuke[i] == b)
    	{
        	++i;
    	}

    	m_world->DestroyBody(b);
	}

# 9.5 Contact Filtering

게임내에서는 모든 객체가 충돌하지 않도록 해야할 수 있습니다. 예를 들어, 특정 캐릭터는 그냥 통과하는 문을 만들고 싶을 수 있습니다. 이를 contact filtering 이라고 부릅니다. 왜냐하면 상호작용이 필터링 되기 때문입니다. 

Often in a game you don't want all objects to collide. For example, you may want to create a door that only certain characters can pass through. This is called contact filtering, because some interactions are filtered out.

Box2D는 b2ContactFilter 클래스를 상속받은 커스텀 필터링을 생성할 수 있도록 하고 있습니다. 해당 클래스는 두 개의 b2Shape 포인터를 파라미터로 가지는 ShoulCollide 함수를 구현해야할 필요가 있습니다. 충돌해야할 경우, 리턴값은 true입니다.

Box2D allows you to achieve custom contact filtering by implementing a b2ContactFilter class. This class requires you to implement a ShouldCollide function that receives two b2Shape pointers. Your function returns true if the shapes should collide.

챕터6에서 b2FilterData 에서 ShouldCollide를 다음과 같이 구현한 바 있습니다.

The default implementation of ShouldCollide uses the b2FilterData defined in Chapter 6, Fixtures.

	bool b2ContactFilter::ShouldCollide(b2Fixture* fixtureA, b2Fixture* fixtureB)
	{
    	const b2Filter& filterA = fixtureA->GetFilterData();
     	const b2Filter& filterB = fixtureB->GetFilterData();

     	if (filterA.groupIndex == filterB.groupIndex && filterA.groupIndex != 0)
     	{
          	return filterA.groupIndex > 0;
     	}

     	bool collide = (filterA.maskBits & filterB.categoryBits) != 0 &&
                  (filterA.categoryBits & filterB.maskBits) != 0;
     	return collide; 	
	}
	

런타임상에서 b2World::SetContactFilter 에서 contact filter의 인스턴스를 생성하고 선언할 수 있습니다. world가 존재하는 한 해당 filter도 존재해야 합니다.

At run-time you can create an instance of your contact filter and register it with b2World::SetContactFilter. Make sure your filter stays in scope while the world exists.

	MyContactFilter filter;
	world->SetContactFilter(&filter);
	// filter remains in scope …