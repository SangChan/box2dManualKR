# Chapter 9 Contacts

# 9.1 About
Contacts are objects created by Box2D to manage collision between two fixtures. If the fixture has children, such as a chain shape, then a contact exists for each relevant child. There are different kinds of contacts, derived from b2Contact, for managing contact between different kinds of fixtures. For example there is a contact class for managing polygon-polygon collision and another contact class for managing circle-circle collision.

Here is some terminology associated with contacts.

## contact point

A contact point is a point where two shapes touch. Box2D approximates contact with a small number of points.

## contact normal

A contact normal is a unit vector that points from one shape to another. By convention, the normal points from fixtureA to fixtureB.

## contact separation
Separation is the opposite of penetration. Separation is negative when shapes overlap. It is possible that future versions of Box2D will create contact points with positive separation, so you may want to check the sign when contact points are reported.

## contact manifold
Contact between two convex polygons may generate up to 2 contact points. Both of these points use the same normal, so they are grouped into a contact manifold, which is an approximation of a continuous region of contact.

## normal impulse
The normal force is the force applied at a contact point to prevent the shapes from penetrating. For convenience, Box2D works with impulses. The normal impulse is just the normal force multiplied by the time step.

## tangent impulse
The tangent force is generated at a contact point to simulate friction. For convenience, this is stored as an impulse.

## contact ids
Box2D tries to re-use the contact force results from a time step as the initial guess for the next time step. Box2D uses contact ids to match contact points across time steps. The ids contain geometric features indices that help to distinguish one contact point from another.

Contacts are created when two fixture’s AABBs overlap. Sometimes collision filtering will prevent the creation of contacts. Contacts are destroyed with the AABBs cease to overlap.

So you might gather that there may be contacts created for fixtures that are not touching (just their AABBs). Well, this is correct. It's a "chicken or egg" problem. We don't know if we need a contact object until one is created to analyze the collision. We could delete the contact right away if the shapes are not touching, or we can just wait until the AABBs stop overlapping. Box2D takes the latter approach because it lets the system cache information to improve performance.

# 9.2 Contact Class
As mentioned before, the contact class is created and destroyed by Box2D. Contact objects are not created by the user. However, you are able to access the contact class and interact with it.

You can access the raw contact manifold:

	b2Manifold* GetManifold();
	const b2Manifold* GetManifold() const;

You can potentially modify the manifold, but this is generally not supported and is for advanced usage.

There is a helper function to get the b2WorldManifold:

	void GetWorldManifold(b2WorldManifold* worldManifold) const;

This uses the current positions of the bodies to compute world positions of the contact points.
Sensors do not create manifolds, so for them use:

	bool touching = sensorContact->IsTouching();

This function also works for non-sensors.

You can get the fixtures from a contact. From those you can get the bodies.

	b2Fixture* fixtureA = myContact->GetFixtureA();
	b2Body* bodyA = fixtureA->GetBody();
	MyActor* actorA = (MyActor*)bodyA->GetUserData();
	
You can disable a contact. This only works inside the b2ContactListener::PreSolve event, discussed below.

# 9.3 Accessing Contacts

You can get access to contacts in several ways. You can access the contacts directly on the world and body structures. You can also implement a contact listener.

You can iterate over all contacts in the world:

	for (b2Contact* c = myWorld->GetContactList(); c; c = c->GetNext())
	{
		// process c
	}
	
You can also iterate over all the contacts on a body. These are stored in a graph using a contact edge structure.

	for (b2ContactEdge* ce = myBody->GetContactList(); ce; ce = ce->next)
	{
		b2Contact* c = ce->contact;
		// process c
	}
	
You can also access contacts using the contact listener that is described below.

> Caution<br>
> Accessing contacts off b2World and b2Body may miss some transient contacts that occur in the middle of the time step. Use b2ContactListener to get the most accurate results.

# 9.4 Contact Listener
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

> Caution<br>
> Do not keep a reference to the pointers sent to b2ContactListener. Instead make a deep copy of the contact point data into your own buffer. The example below shows one way of doing this.

At run-time you can create an instance of the listener and register it with b2World::SetContactListener. Be sure your listener remains in scope while the world object exists.

## Begin Contact Event
This is called when two fixtures begin to overlap. This is called for sensors and non-sensors. This event can only occur inside the time step.
 
## End Contact Event
This is called when two fixtures cease to overlap. This is called for sensors and non-sensors. This may be called when a body is destroyed, so this event can occur outside the time step.

## Pre-Solve Event
This is called after collision detection, but before collision resolution. This gives you a chance to disable the contact based on the current configuration. For example, you can implement a one-sided platform using this callback and calling b2Contact::SetEnabled(false). The contact will be re-enabled each time through collision processing, so you will need to disable the contact every time-step. The pre-solve event may be fired multiple times per time step per contact due to continuous collision detection.

	void PreSolve(b2Contact* contact, const b2Manifold* oldManifold)
	{
		b2WorldManifold worldManifold;
		contact->GetWorldManifold(&worldManifold);
		if (worldManifold.normal.y < -0.5f){
			contact->SetEnabled(false);
		}
	}
	
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
The post solve event is where you can gather collision impulse results. If you don’t care about the impulses, you should probably just implement the pre-solve event.

It is tempting to implement game logic that alters the physics world inside a contact callback. For example, you may have a collision that applies damage and try to destroy the associated actor and its rigid body. However, Box2D does not allow you to alter the physics world inside a callback because you might destroy objects that Box2D is currently processing, leading to orphaned pointers.

The recommended practice for processing contact points is to buffer all contact data that you care about and process it after the time step. You should always process the contact points immediately after the time step; otherwise some other client code might alter the physics world, invalidating the contact buffer. When you process the contact buffer you can alter the physics world, but you still need to be careful that you don't orphan pointers stored in the contact point buffer. The testbed has example contact point processing that is safe from orphaned pointers.

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
Often in a game you don't want all objects to collide. For example, you may want to create a door that only certain characters can pass through. This is called contact filtering, because some interactions are filtered out.

Box2D allows you to achieve custom contact filtering by implementing a b2ContactFilter class. This class requires you to implement a ShouldCollide function that receives two b2Shape pointers. Your function returns true if the shapes should collide.

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
	
At run-time you can create an instance of your contact filter and register it with b2World::SetContactFilter. Make sure your filter stays in scope while the world exists.

	MyContactFilter filter;
	world->SetContactFilter(&filter);
	// filter remains in scope …