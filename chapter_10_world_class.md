# Chapter 10 World Class

## About
The b2World class contains the bodies and joints. It manages all aspects of the simulation and allows for asynchronous queries (like AABB queries and ray-casts). Much of your interactions with Box2D will be with a b2World object.

## Creating and Destroying a World

Creating a world is fairly simple. You just need to provide a gravity vector and a Boolean indicating if bodies can sleep. Usually you will create and destroy a world using new and delete.

	b2World* myWorld = new b2World(gravity, doSleep);
	... do stuff ...
	delete myWorld;

## Using a World

The world class contains factories for creating and destroying bodies and joints. These factories are discussed later in the sections on bodies and joints. There are some other interactions with b2World that I will cover now.

## Simulation

The world class is used to drive the simulation. You specify a time step and a velocity and position iteration count. For example:

	float32 timeStep = 1.0f / 60.f;
	int32 velocityIterations = 10;
	int32 positionIterations = 8;
	myWorld->Step(timeStep, velocityIterations, positionIterations);

After the time step you can examine your bodies and joints for information. Most likely you will grab the position off the bodies so that you can update your actors and render them. You can perform the time step anywhere in your game loop, but you should be aware of the order of things. For example, you must create bodies before the time step if you want to get collision results for the new bodies in that frame.

As I discussed above in the HelloWorld tutorial, you should use a fixed time step. By using a larger time step you can improve performance in low frame rate scenarios. But generally you should use a time step no larger than 1/30 seconds. A time step of 1/60 seconds will usually deliver a high quality simulation.

The iteration count controls how many times the constraint solver sweeps over all the contacts and joints in the world. More iteration always yields a better simulation. But don't trade a small time step for a large iteration count. 60Hz and 10 iterations is far better than 30Hz and 20 iterations.

After stepping, you should clear any forces you have applied to your bodies. This is done with the command b2World::ClearForces. This lets you take multiple sub-steps with the same force field.

	myWorld->ClearForces();

## Exploring the World
The world is a container for bodies, contacts, and joints. You can grab the body, contact, and joint lists off the world and iterate over them. For example, this code wakes up all the bodies in the world:

	for (b2Body* b = myWorld->GetBodyList(); b; b = b->GetNext())
	{
    	b->SetAwake(true);
	}
Unfortunately real programs can be more complicated. For example, the following code is broken:

	for (b2Body* b = myWorld->GetBodyList(); b; b = b->GetNext())
	{
    	GameActor* myActor = (GameActor*)b->GetUserData();
    	if (myActor->IsDead())
    	{
        	myWorld->DestroyBody(b); // ERROR: now GetNext returns garbage.
    	}
	}
	
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
	
Obviously to make this work, GameCrazyBodyDestroyer must be honest about what it has destroyed.

## AABB Queries

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
	
You cannot make any assumptions about the order of the callbacks.

## Ray Casts

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