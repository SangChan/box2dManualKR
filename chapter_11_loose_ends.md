# Chapter 11 Loose Ends

# 11.1 User Data

b2Fixture, b2Body, j2Joint 클래스는 보이드 포인터 형으로 된 user data를 추가하는 것이 가능합니다. Box2D 데이터 구조체를 검사하거나 게임 엔진상의 객체와 얼마나 연관되어 있는지 관여하고 있는지를 밝히고자 할 때 유용합니다.

The b2Fixture, b2Body, and b2Joint classes allow you to attach user data as a void pointer. This is handy when you are examining Box2D data structures and you want to determine how they relate to the objects in your game engine.

예를 들어, 일반적으로 이 기능은 강체를 사용하고 있는 actor의 포인터를 저장하는데 사용됩니다. 이는 선형 참조가 되므로 actor 가 있다면 이를 통해 강체를 얻을 수 있습니다. 반대로 강체가 있다면 이를 통해 actor를 얻는 것도 가능합니다.

For example, it is typical to attach an actor pointer to the rigid body on that actor. This sets up a circular reference. If you have the actor, you can get the body. If you have the body, you can get the actor.

	GameActor* actor = GameCreateActor();
	b2BodyDef bodyDef;
	bodyDef.userData = actor;
	actor->body = box2Dworld->CreateBody(&bodyDef);
	
user data가 반드시 필요한 곳은 다음과 같습니다.

Here are some examples of cases where you would need the user data:

* 충돌결과에 따라 actor에 데미지를 줘야함.
* 캐릭터가 축따라 정렬된 박스(AAB) 내부에 있을 때, 정해진 이벤트가 일어나야 함.
* Box2D가 joint의 소멸을 알려주고자 게임 구조체에 접근하려 함.

* Applying damage to an actor using a collision result.
* Playing a scripted event if the player is inside an axis-aligned box.
* Accessing a game structure when Box2D notifies you that a joint is going to be destroyed.

user data는 부가적인 정보이므로 무엇이든 넣을 수 있다는 것을 명심하십시요. 하지만 일관성을 유지해야 합니다. 여기서 말하는 일관성이란 강체에 actor의 포인터값을 저장하고자 한다면 모든 강체에 그렇게 해야 한다는 뜻입니다.

Keep in mind that user data is optional and you can put anything in it. However, you should be consistent. For example, if you want to store an actor pointer on one body, you should keep an actor pointer on all bodies. 

한 개의 강체에는 actor 포인터를 저장하고 다른 강체에는 다른 종류의 포인터를 저장하지 마십시요. 타입캐스팅을 하는 순간 프로그램에 오류가 발생할 수 있습니다.

Don't store an actor pointer on one body, and a foo pointer on another body. Casting an actor pointer to a foo pointer may lead to a crash.

user data에는 기본적으로 NULL값이 주어져 있습니다.

User data pointers are NULL by default.

게임의 특정 정보, 그러니까 재료의 종류라던가 반복되는 효과나 반복되느 사운드 같은 것을 저장할 수 있는 데이터 구조체를 따로 정의하는 것도 고려해볼 수 있겠습니다.

For fixtures you might consider defining a user data structure that lets you store game specific information, such as material type, effects hooks, sound hooks, etc.

	struct FixtureUserData
	{
		int materialIndex;
		…
	};

	FixtureUserData myData = new FixtureUserData;
	myData->materialIndex = 2;

	b2FixtureDef fixtureDef;
	fixtureDef.shape = &someShape;
	fixtureDef.userData = myData;

	b2Fixture* fixture = body->CreateFixture(&fixtureDef);
	…
	delete fixture->GetUserData();
	fixture->SetUserData(NULL);
	body->DestroyFixture(fixture);

# 11.2 Implicit Destruction

Box2D는 레퍼런스 카운팅을 하지 않습니다. (역주: New, Alloc, Retain, Copy 가 일어나면 레퍼런스 카운트가 증가하므로 Release를 불러 레퍼런스 카운트를 낮춰 0이 되도록 해야 객체가 사라지는 방식을 말합니다.) 그러므로 강체를 소멸한다면 바로 사라집니다. 소멸을 선언한 강체의 포인터에 접근하는 것은 정의되어 있지 않은 행동이므로 프로그램에 오류가 발생합니다. 이를 방지하기 위해 디버그 빌드상 메모리 관리자에 소멸된 개체를 FDFDFDFD로 채웁니다. 이렇게하면 몇몇 경우에 쉽게 문제를 찾을 수 있습니다. 

Box2D doesn't use reference counting. So if you destroy a body it is really gone. Accessing a pointer to a destroyed body has undefined behavior. In other words, your program will likely crash and burn. To help fix these problems, the debug build memory manager fills destroyed entities with FDFDFDFD. This can help find problems more easily in some cases.

Box2D 개체를 소멸한다면, 소멸될 객체에 대한 모든 참조를 없애는 것은 온전히 사용자의 몫입니다. 해당 개체에 대해 단 한 개의 참조를 가진다면 간단한 문제입니다. 참조를 여러 번 한다면, 원시포인터를 감싼 관리용 클래스를 구현하는 것을 고려해야 할 것 입니다.

If you destroy a Box2D entity, it is up to you to make sure you remove all references to the destroyed object. This is easy if you only have a single reference to the entity. If you have multiple references, you might consider implementing a handle class to wrap the raw pointer.

Box2D를 쓸 때, 강체와 shape와 joint를 수도 없이 생성하고 소멸해야 할 것 입니다. 이런 개체를 관리하는 것은 Box2D에서 알아서 해줍니다. 강체를 소멸하면 첨부되어 있는 shape와 joint는 자동적으로 소멸됩니다. 이를 암시적 파괴라고 합니다.

Often when using Box2D you will create and destroy many bodies, shapes, and joints. Managing these entities is somewhat automated by Box2D. If you destroy a body then all associated shapes and joints are automatically destroyed. This is called implicit destruction.

강체를 소멸하면, 이에 첨부된 모든 shape, joint, contact도 소멸됩니다. 이를 암시적 파괴라고 합니다. 이런 joint나 contact에 연결된 강체는 깨어나게 됩니다. 대체로 이 방식은 편리하지만, 다음의 중요한 사안을 명심해야 합니다:

When you destroy a body, all its attached shapes, joints, and contacts are destroyed. This is called implicit destruction. Any body connected to one of those joints and/or contacts is woken. This process is usually convenient. However, you must be aware of one crucial issue:

> 주의 <br>
> 강체가 소멸될 때, 강체에 부착되어 있던 fixture 나 joint는 자동적으로 소멸됩니다. 이런 shape 및 joint에 대한 포인터를 가지고 있다면 null 값을 주어야 합니다. 그러지 않을 경우, 차후 해당 shape 이나 joint로 접근하고자 할 때 프로그램이 에러나는 것을 볼 수 있을 것 입니다.
> Caution <br>
> When a body is destroyed, all fixtures and joints attached to the body are automatically destroyed. You must nullify any pointers you have to those shapes and joints. Otherwise, your program will die horribly if you try to access or destroy those shapes or joints later.

joint 포인터에 null값을 주는 것을 돕기 위해서 world객체에 b2DestructionListener를 구현한 구현체를 Box2D에 제공하여 암시적 파괴가 일어날 때, world object가 해당 리스너를 부르도록 할 수 있습니다.

To help you nullify your joint pointers, Box2D provides a listener class named b2DestructionListener that you can implement and provide to your world object. Then the world object will notify you when a joint is going to be implicitly destroyed

 Note that there no notification when a joint or fixture is explicitly destroyed. In this case ownership is clear and you can perform the necessary cleanup on the spot. If you like, you can call your own implementation of b2DestructionListener to keep cleanup code centralized.

Implicit destruction is a great convenience in many cases. It can also make your program fall apart. You may store pointers to shapes and joints somewhere in your code. These pointers become orphaned when an associated body is destroyed. The situation becomes worse when you consider that joints are often created by a part of the code unrelated to management of the associated body. For example, the testbed creates a b2MouseJoint for interactive manipulation of bodies on the screen.

Box2D provides a callback mechanism to inform your application when implicit destruction occurs. This gives your application a chance to nullify the orphaned pointers. This callback mechanism is described later in this manual.

You can implement a b2DestructionListener that allows b2World to inform you when a shape or joint is implicitly destroyed because an associated body was destroyed. This will help prevent your code from accessing orphaned pointers.

	class MyDestructionListener : public b2DestructionListener
	{
    	void SayGoodbye(b2Joint* joint)
    	{
        	// remove all references to joint.
    	}
	};

You can then register an instance of your destruction listener with your world object. You should do this during world initialization.

	myWorld->SetListener(myDestructionListener);

# 11.3 Pixels and Coordinate Systems

Recall that Box2D uses MKS (meters, kilograms, and seconds) units and radians for angles. You may have trouble working with meters because your game is expressed in terms of pixels. To deal with this in the testbed I have the whole game work in meters and just use an OpenGL viewport transformation to scale the world into screen space.

	float lowerX = -25.0f, upperX = 25.0f, lowerY = -5.0f, upperY = 25.0f;
	gluOrtho2D(lowerX, upperX, lowerY, upperY);

If your game must work in pixel units then you should convert your length units from pixels to meters when passing values from Box2D. Likewise you should convert the values received from Box2D from meters to pixels. This will improve the stability of the physics simulation.

You have to come up with a reasonable conversion factor. I suggest making this choice based on the size of your characters. Suppose you have determined to use 50 pixels per meter (because your character is 75 pixels tall). Then you can convert from pixels to meters using these formulas:

	xMeters = 0.02f * xPixels;
	yMeters = 0.02f * yPixels;

In reverse:

	xPixels = 50.0f * xMeters;
	yPixels = 50.0f * yMeters;

You should consider using MKS units in your game code and just convert to pixels when you render. This will simplify your game logic and reduce the chance for errors since the rendering conversion can be isolated to a small amount of code.

If you use a conversion factor, you should try tweaking it globally to make sure nothing breaks. You can also try adjusting it to improve stability.