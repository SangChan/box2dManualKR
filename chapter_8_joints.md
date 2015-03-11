# Chapter 8 Joints

# 8.1 About

강체끼리 혹은 강체와 world 를 joint로 이어줄 수 있습니다. 일반적으로 게임에서는 랙돌, 도르레, 시소(역주: 원문은 teeter 인데 시소인지 라비린스인지 잘 모르겠습니다.) 같은 게임에서 사용됩니다. 재미있는 움직임을 표현하기 위해서 여러 방법으로 joint를 조합할 수 있습니다. 

Joints are used to constrain bodies to the world or to each other. Typical examples in games include ragdolls, teeters, and pulleys. Joints can be combined in many different ways to create interesting motions.

joint에 움직일 수 있는 범위를 설정하여 가동을 제한할 수 있습니다. 또한 joint에 주어지는 힘이나 회전력이 초과되더라도 특정 속도로만 움직이는 모터기능도 제공할 수 있습니다.

Some joints provide limits so you can control the range of motion. Some joint provide motors which can be used to drive the joint at a prescribed speed until a prescribed force/torque is exceeded.

joint motor 는 여러가지로 이용될 수 있는데, 모터를 이용해 원하는 위치와 실제 위치 사이의 차이에 대해 지정된 속도로 이동하도록 할 수 있습니다. joint의 마찰을 다음과 같이 하여 보여줄 수 있습니다. joint의 속도는 0으로 설정하고 모터에 대해 힘 혹은 회전력을 작지만 가능한 최대로 주면, 부하가 너무 커져 joint의 이동을 유지하려고 하게 됩니다.

Joint motors can be used in many ways. You can use motors to control position by specifying a joint velocity that is proportional to the difference between the actual and desired position. You can also use motors to simulate joint friction: set the joint velocity to zero and provide a small, but significant maximum motor force/torque. Then the motor will attempt to keep the joint from moving until the load becomes too strong.

# 8.2 The Joint Definition

joint의 타입은 b2JointDef에서 파생된 것입니다. 모든 joint는 두 개의 다른 강체 사이에 연결됩니다. 그 중 한 개의 강체는 정적일 수 있습니다. 정적 강체 혹은 운동학적 강체를 서로 joint로 연결하는 것은 가능하지만, 물리엔진 연산 중에는 아무 일도 일어나지 않을 것 입니다.

Each joint type has a definition that derives from b2JointDef. All joints are connected between two different bodies. One body may static. Joints between static and/or kinematic bodies are allowed, but have no effect and use some processing time.

유저 데이터를 모든 joint 타입에 설정할 수 있으며, 연결된 강체끼리 서로 충돌하는 것을 방지하기 위한 플래그값을 설정할 수 있습니다. 기본적으로 이렇게 동작하며, collideConnected에 대한 불리언 값을 설정하여 연결된 강체간 동작을 어찌할지 결정할 수 있습니다.

You can specify user data for any joint type and you can provide a flag to prevent the attached bodies from colliding with each other. This is actually the default behavior and you must set the collideConnected Boolean to allow collision between to connected bodies.

대부분의 joint 속성은 도형값을 필요로 합니다. joint는 기준점(anchor point)에 의해 정의될 수 있습니다. 이 기준점은 추가된 강체에 고정되어 있습니다. Box2D는 지역좌표계를 기준으로 설정된 이 기준점이 필요합니다.  이 방식으로 강체를 정의하게 되면 강체를 이동하는 것(일반적으로 게임을 저장하고 불러오기 할 때 발생할 가능성이 높음)으로 joint의 제한사항을 위반할 수 있습니다. 덧붙여서, 몇몇 joint에는 각 강체간의 상대적인 기본 각도값을 설정할 필요가 있습니다. 이는 회전에 대한 제한을 정확하게 하기 위해 필요합니다.

Many joint definitions require that you provide some geometric data. Often a joint will be defined by anchor points. These are points fixed in the attached bodies. Box2D requires these points to be specified in local coordinates. This way the joint can be specified even when the current body transforms violate the joint constraint --- a common occurrence when a game is saved and reloaded. Additionally, some joint definitions need to know the default relative angle between the bodies. This is necessary to constrain rotation correctly.

도형값을 설정하는 것은 매우 지루합니다. 그래서 joint를 초기화하기 위한 함수를 통해 현재의 강체를 움직일 때, 많은 일을 해야하지 않아도 되도록 합니다. 하지만 이러한 초기화 함수는 일반적으로 프로토타이핑을 위해서만 사용됩니다. 상용 코드에서는 도형값을 반드시 설정해야만 합니다. 이렇게 해야 joint의 움직임이 더 자연스러워 보입니다.

Initializing the geometric data can be tedious, so many joints have initialization functions that use the 
current body transforms to remove much of the work. However, these initialization functions should usually only be used for prototyping. Production code should define the geometry directly. This will make joint behavior more robust.

관절에 대해 정의는 각 타입에 따라 다릅니다. 하기에 기술 하도록 하겠습니다.

The rest of the joint definition data depends on the joint type. We cover these now.

# 8.3 Joint Factory

joint는 world 의 factory 메소드를 통해 생성되고 소멸 될 수 있습니다. 오래된 예제는 다음과 같습니다:

Joints are created and destroyed using the world factory methods. This brings up an old issue:

> 주의 <br>
> joint를 스택이나 힙에 new 혹은 malloc 를 이용해 생성하지 마십시요. 반드시 b2World 클래스의 create, destroy 메소드를 사용해서 생성해야만 합니다.<br> 
> 다음은 회전 조인트의 생명주기에 대한 예제입니다. <br>
> 
> Caution <br>
> Don't try to create a joint on the stack or on the heap using new or malloc. You must create and destroy bodies and joints using the create and destroy methods of the b2World class.
Here's an example of the lifetime of a revolute joint:

	b2RevoluteJointDef jointDef;
	jointDef.bodyA = myBodyA;
	jointDef.bodyB = myBodyB;
	jointDef.anchorPoint = myBodyA->GetCenterPosition();
	b2RevoluteJoint* joint = (b2RevoluteJoint*)myWorld->CreateJoint(&jointDef);
	... do stuff ...
	myWorld->DestroyJoint(joint);
	joint = NULL;

joint가 소멸된 후에는 반드시 해당 포인터에 null 값을 대입하는 것이 좋습니다. 만약에 해당 포인터를 다시 사용하려고 한다면 통제상태에 있는 프로그램이 강제종료 될 것입니다.

It is always good to nullify your pointer after they are destroyed. This will make the program crash in a controlled manner if you try to reuse the pointer.

joint의 생명주기에 대한 부분은 쉽지 않습니다. 다음의 경고를 명심하십시요:

The lifetime of a joint is not simple. Heed this warning well:

> 주의 <br>
> 추가된 강체가 소멸할 때, joint도 소멸합니다.
> 
> Caution <br>
> Joints are destroyed when an attached body is destroyed.

이러한 예방 조치는 항상 필요한 것은 아닙니다. 추가된 강체의 소멸 이전에 joint를 항상 소멸하더라도 게임엔진을 구성하는데는 문제가 없습니다. 이 경우, 리스너 클래스를 만들 필요도 없습니다. 자세한 내용은 Implict Destruction 부분을 확인하시기 바랍니다.

This precaution is not always necessary. You may organize your game engine so that joints are always destroyed before the attached bodies. In this case you don't need to implement the listener class. See the section on Implicit Destruction for details.

# 8.4 Using Joints

joint를 생성하고 소멸할때까지 다시 접근하지 않는 경우가 대부분이다. 좀더 풍부하게 물리법칙을 묘사하기 위한 여러 유용한 데이터가 joint에 존재한다.

Many simulations create the joints and don't access them again until they are destroyed. However, there is a lot of useful data contained in joints that you can use to create a rich simulation.

첫째로, 강체와 기준점, user data를 joint로 부터 얻을 수 있다.

First of all, you can get the bodies, anchor points, and user data from a joint.

	b2Body* GetBodyA();
	b2Body* GetBodyB();
	b2Vec2 GetAnchorA();
	b2Vec2 GetAnchorB();
	void* GetUserData();
	
모든 joint는 반작용력과 반작용회전력을 가집니다. 이 반작용력은 2번 강체의 기준점에 적용됩니다. 이 반작용력을 통해 joint를 파괴하거나 게임 이벤트를 발생시킬 수도 있습니다. 이 함수는 관련 연산을 실행하므로 결과값이 필요하지 않다면 부르지 말기를 권장합니다.

All joints have a reaction force and torque. This the reaction force applied to body 2 at the anchor point. You can use reaction forces to break joints or trigger other game events. These functions may do some computations, so don't call them if you don't need the result.

	b2Vec2 GetReactionForce();
	float32 GetReactionTorque();

# 8.5 Distance Joint

가장 단순한 형태의 joint는 distance joint 이며, 두 개의 강체사이의 두 점의 거리가 항상 일정하게 작용합니다. 이 distance joint를 지정하기 위해서는 두 개의 강체가 해당 위치에 이미 존재하고 있어야 하며, world 좌표계상의 두 개의 기준점을 기준으로 distance joint 를 정의할 수 있습니다. 첫번째 기준점은 1번 강체에, 두번째 기준점은 2번 강체에 연결됩니다. 이 두 개의 점은 동일한 거리를 유지한다는 제한을 가지게 됩니다.

One of the simplest joint is a distance joint which says that the distance between two points on two bodies must be constant. When you specify a distance joint the two bodies should already be in place. Then you specify the two anchor points in world coordinates. The first anchor point is connected to body 1, and the second anchor point is connected to body 2. These points imply the length of the distance constraint.

다음은 distance joint를 정의하는 예제입니다. 이 경우, 연결된 강체간 충돌은 가능합니다.
 
Here is an example of a distance joint definition. In this case we decide to allow the bodies to collide.

	b2DistanceJointDef jointDef;
	jointDef.Initialize(myBodyA, myBodyB, worldAnchorOnBodyA, worldAnchorOnBodyB);
	jointDef.collideConnected = true;

distance joint는 완충 스프링 연결과 같은 부드러운 물체를 만들기 위해 사용됩니다. 웹페이지 상에 존재하는 예제페이지에서 어떻게 작동하는지 확인하기 바랍니다.

The distance joint can also be made soft, like a spring-damper connection. See the Web example in the testbed to see how this behaves.

해당 물체의 부드러움은 내부 정의값 중 주파수와 완충비율을 조절함으로써 얻어낼 수 있습니다. 주파수는 기타줄 같은 조화진동자의 주파수를 생각하면 됩니다. Hertz 값으로 정의되며, 일반적으로 time step에서 사용하는 주파수의 절반  이하이어야 합니다. time step 에서 60Hz 를 사용한다면 distance에서는 30Hz이하 값을 가져야 합니다. 이는 나이퀴스트 주파수(특정 주파수가 min, max를 오고갈 때, 다른 주파수가 동일하게 min, max를 오고 가면 데이터를 잃고 잘못된 신호로 잡을 수 있음)에 기인합니다.

Softness is achieved by tuning two constants in the definition: frequency and damping ratio. Think of the frequency as the frequency of a harmonic oscillator (like a guitar string). The frequency is specified in Hertz. Typically the frequency should be less than a half the frequency of the time step. So if you are using a 60Hz time step, the frequency of the distance joint should be less than 30Hz. The reason is related to the Nyquist frequency.

완충비율은 차원에 상관없으며 일반적으로 0~1까지의 값을 가지지만, 더 큰 값도 가질 수 있습니다. 값이 1일 때, 모든 진동이 사라지므로 최대로 완충작용을 할 수 있습니다.

The damping ratio is non-dimensional and is typically between 0 and 1, but can be larger. At 1, the damping is critical (all oscillations should vanish).

	jointDef.frequencyHz = 4.0f;
	jointDef.dampingRatio = 0.5f;

# 8.6 Revolute Joint

revolute joint는 두 개의 강체에 대해 일반적인 기준점을 공유하도록 강제하여 경첩점이 되도록 합니다. revolute joint는 두 개의 강체간 상대적인 회전에 대해서 단일자유도를 가지고 있습니다. 이를 관절 각도라고 합니다.

A revolute joint forces two bodies to share a common anchor point, often called a hinge point. The revolute joint has a single degree of freedom: the relative rotation of the two bodies. This is called the joint angle.
 
이 회전을 지정하기 위해 두 개의 강체와 한 개의 기준점이 필요하며, 초기화 함수는 강체가 이미 지정한 위치에 존재한다고 상정합니다.

To specify a revolute you need to provide two bodies and a single anchor point in world space. The initialization function assumes that the bodies are already in the correct position.

하단의 예제는 두 개의 강체가 1번 강체의 질량중심점에 대해 revolute joint 로 연결되는 것입니다.

In this example, two bodies are connected by a revolute joint at the first body's center of mass.

	b2RevoluteJointDef jointDef;
	jointDef.Initialize(myBodyA, myBodyB, myBodyA->GetWorldCenter());

revolute joint 의 회전각도는 강체B가 시계 반대 방향으로 회전하므로 양수여야 합니다. Box2D의 다른 각도값 처럼 revolte joint 에서 사용하는 각도값도 라디안 값을 사용합니다. 관례적으로 Initialize() 함수를 사용해 생성될 때에는 revolute joint의 각도값은 두 강체의 현재의 회전과는 별개로 0이 됩니다.

The revolute joint angle is positive when bodyB rotates CCW about the angle point. Like all angles in Box2D, the revolute angle is measured in radians. By convention the revolute joint angle is zero when the joint is created using Initialize(), regardless of the current rotation of the two bodies.

때때로 회전각을 제어하고 싶을 수 있습니다. 이를 위해, revolute joint는 임의로 움직임에 한계를 절거나 모터 기능을 추가하는 식으로 제어를 할 수 있습니다.

In some cases you might wish to control the joint angle. For this, the revolute joint can optionally simulate a joint limit and/or a motor.

joint는 회전각을 낮추거나 높이는 식으로 제한을 강제합니다. 제한을 거는 것으로써 이 일이 일어날 수 있는 것보다 조금 더 많은 회전력이 가해집니다. 제한값을 설정할 수 있는 범위에는 0도 포함되며, 그렇지 않을 경우 물리법칙이 적용될 때, joint가 휘청거릴 수 있습니다.

A joint limit forces the joint angle to remain between a lower and upper bound. The limit will apply as much torque as needed to make this happen. The limit range should include zero, otherwise the joint will lurch when the simulation begins.

joint 모터는 joint의 속도(각도의 시간에 대한 도함수)를 지정할 수 있습니다. 속도는 음수 혹은 양수일 수 있습니다. 모터에 무한정의 힘을 줄 수도 있으나, 일반적으로 추천하지 않습니다. 다음의 모순된 질문을 다시 물어보겠습니다:

A joint motor allows you to specify the joint speed (the time derivative of the angle). The speed can be negative or positive. A motor can have infinite force, but this is usually not desirable. Recall the eternal question:

** "물체를 무조건 움직일 수 있는 힘이 절대 움직이지 않는 물체와 만나면 무슨 일이 발생하겠습니까?" **

** "What happens when an irresistible force meets an immovable object?" **

이것이 우아하지 않다는 것만은 확실히 얘기할 수 있습니다. joint의 모터에 회전력의 최대 값을 지정할 수 있습니다. 주어진 회전력이 설정된 최대값을 초과하지 않는 한 모터는 지정된 속도를 유지할 것 입니다. 최대값을 초과한다면, joint는 느려지고, 반전된 값으로 작동하게 될 것 입니다.

I can tell you it's not pretty. So you can provide a maximum torque for the joint motor. The joint motor will maintain the specified speed unless the required torque exceeds the specified maximum. When the maximum torque is exceeded, the joint will slow down and can even reverse.

joint의 마찰력을 묘사하기 위해 joint 모터를 이용할 수 있습니다. joint의 속도를 0으로 지정하고 회전력의 최대값을 작지만 특정한 값으로 설정합니다. 모터는 joint의 회전을 방지하기 위해 노력하여, 움직임에 상당한 부하가 걸릴 것 입니다.

You can use a joint motor to simulate joint friction. Just set the joint speed to zero, and set the maximum torque to some small, but significant value. The motor will try to prevent the joint from rotating, but will yield to a significant load.

상단에서 기술한 joint 정의에 대한 내용입니다. joint에는 제한이 걸려있고 모터는 사용가능하며, 모터에는 마찰력이 설정되어 있습니다.

Here's a revision of the revolute joint definition above; this time the joint has a limit and a motor enabled. The motor is setup to simulate joint friction.

	b2RevoluteJointDef jointDef;
	jointDef.Initialize(bodyA, bodyB, myBodyA->GetWorldCenter());
	jointDef.lowerAngle = -0.5f * b2_pi; // -90 degrees
	jointDef.upperAngle = 0.25f * b2_pi; // 45 degrees
	jointDef.enableLimit = true;
	jointDef.maxMotorTorque = 10.0f;
	jointDef.motorSpeed = 0.0f;
	jointDef.enableMotor = true;

revolute joint 의 각도, 속도, 모터의 회전력 값에 접근할 수 있습니다.

You can access a revolute joint's angle, speed, and motor torque.

	float32 GetJointAngle() const;
	float32 GetJointSpeed() const;
	float32 GetMotorTorque() const;

또한 모터의 특성 값도 동시에 설정하는 것이 가능합니다.

You also update the motor parameters each step.

	void SetMotorSpeed(float32 speed);
	void SetMaxMotorTorque(float32 torque);

joint 모터는 몇몇 재밌는 기능이 있습니다. 매 time step에서 joint의 속도를 설정할 수 있으므로, joint를 사인 곡선이나 사용자가 원하는대로 왔다갔다 움직이도록 할 수 있습니다.

Joint motors have some interesting abilities. You can update the joint speed every time step so you can make the joint move back-and-forth like a sine-wave or according to whatever function you want.

	... Game Loop Begin ...
	myJoint->SetMotorSpeed(cosf(0.5f * time));
	... Game Loop End ...

또한, 지정한 관절 각도를 따라다니는 joint 모터를 사용할 수 있다. 예를 들어 다음과 같이:

You can also use joint motors to track a desired joint angle. For example:

	... Game Loop Begin ...
	float32 angleError = myJoint->GetJointAngle() - angleTarget;
	float32 gain = 0.1f;
	myJoint->SetMotorSpeed(-gain * angleError);
	... Game Loop End ...

gain값이 너무 크면 안정성이 떨어지므로 너무 크지 않도록 합니다.

Generally your gain parameter should not be too large. Otherwise your joint may become unstable.

# 8.7 Prismatic Joint

prismatic joint는 지정된 축에 따라 두 개의 강체가 상대적인 이동을 할 수 있도록 합니다. prismatic joint는 상대적인 회전은 작동하지 않도록 합니다. prismatic joint는 단일자유도를 가지고 있습니다.

A prismatic joint allows for relative translation of two bodies along a specified axis. A prismatic joint prevents relative rotation. Therefore, a prismatic joint has a single degree of freedom.

prismatic joint의 정의는 revolute joint와 비슷합니다. 단지, 회전력이 힘으로, 각도가 이동으로 대체되었을 뿐입니다. 이 유사점을 이용한 다음의 예제는 prismatic joint를 joint에 한계와 마찰에 관한 motor를 정의하고 있습니다. 
 
The prismatic joint definition is similar to the revolute joint description; just substitute translation for angle and force for torque. Using this analogy provides an example prismatic joint definition with a joint limit and a friction motor:

	b2PrismaticJointDef jointDef;
	b2Vec2 worldAxis(1.0f, 0.0f);
	jointDef.Initialize(myBodyA, myBodyB, myBodyA->GetWorldCenter(), worldAxis);
	jointDef.lowerTranslation = -5.0f;
	jointDef.upperTranslation = 2.5f;
	jointDef.enableLimit = true;
	jointDef.maxMotorForce = 1.0f;
	jointDef.motorSpeed = 0.0f;
	jointDef.enableMotor = true;

revolute joint는 화면밖에 존재하는 암시적인 축을 가지고 있습니다. prismatic joint는 명시적이고 화면에 평행한 축을 필요로 합니다. 이 축은 두 개의 강체에 대해 고정이며, 강체의 움직임을 따릅니다.

The revolute joint has an implicit axis coming out of the screen. The prismatic joint needs an explicit axis parallel to the screen. This axis is fixed in the two bodies and follows their motion.

revolute joint 처럼 prismatic joint의 이동에 대한 값은 Initialize()함수로 막 생성되었을때는 0입니다. 그러므로 0값이 이동의 최저와 최대값 사이에 존재하는지 확인이 필요합니다.

Like the revolute joint, the prismatic joint translation is zero when the joint is created using Initialize(). So be sure zero is between your lower and upper translation limits.

prismatic joint의 사용법은 revolute joint와 유사합니다. 다음은 관련 함수입니다:

Using a prismatic joint is similar to using a revolute joint. Here are the relevant member functions:

	float32 GetJointTranslation() const;
	float32 GetJointSpeed() const;
	float32 GetMotorForce() const;
	void SetMotorSpeed(float32 speed);
	void SetMotorForce(float32 force);

# 8.8 Pulley Joint

pulley는 도르래를 묘사하기 위해 사용됩니다. pulley는 두 개의 강체를 지면에 연결하고 서로를 연결합니다. 한 개의 강체가 위로 올라가면 다른 한 개는 아래로 내려갑니다. 이때, pulley의 로프 총 길이는 초기값으로 보존됩니다.

A pulley is used to create an idealized pulley. The pulley connects two bodies to ground and to each other. As one body goes up, the other goes down. The total length of the pulley rope is conserved according to the initial configuration.

	length1 + length2 == constant

비율 값을 조절하여 도르래 장치를 구현할 수 있습니다. 도르래의 한 쪽을 확장시키는 것이 다른 한 쪽을 확장하는 것보다 빠르기 때문입니다. 동시에 한 쪽에 대한 구속력이 다른 한 쪽보다 작아짐을 뜻합니다. 이를 통해 지렛대를 만드는 것이 가능합니다.

You can supply a ratio that simulates a block and tackle. This causes one side of the pulley to extend faster than the other. At the same time the constraint force is smaller on one side than the other. You can use this to create mechanical leverage.

	length1 + ratio * length2 == constant

예를 들어 비율 값을 2로 준다면, length1은 legth2의 2배 만큼 달라지게 된다. 또한 밧줄을 통해 1번 강체에 가해지는 힘은 2번 강체에 주어지는 구속력의 절반이 될 것 입니다.

For example, if the ratio is 2, then length1 will vary at twice the rate of length2. Also the force in the rope attached to body1 will have half the constraint force as the rope attached to body2.

도르래가 한쪽으로 완전히 확장되어 있다면, 여러가지 문제가 발생할 수 있습니다. 다른 방향에 대해 밧줄의 길이가 0이 될 것 입니다. 이 시점에서 제약에 대한 방정식이 특이방정식이 되는 나쁜 결과를 낳는다. 이를 막을 수 있는 충돌 도형을 만들어야만 한다.
 
Pulleys can be troublesome when one side is fully extended. The rope on the other side will have zero length. At this point the constraint equations become singular (bad). You should configure collision shapes to prevent this.

다음은 도르래를 정의하는 예제입니다:

Here is an example pulley definition:

	b2Vec2 anchor1 = myBody1->GetWorldCenter();
	b2Vec2 anchor2 = myBody2->GetWorldCenter();
	b2Vec2 groundAnchor1(p1.x, p1.y + 10.0f);
	b2Vec2 groundAnchor2(p2.x, p2.y + 12.0f);
	float32 ratio = 1.0f;
	b2PulleyJointDef jointDef;
	jointDef.Initialize(myBody1, myBody2, groundAnchor1, groundAnchor2, anchor1, anchor2, ratio);

pulley joint는 현재의 길이를 다음의 메소드로 구할 수 있습니다.

Pulley joints provide the current lengths.

	float32 GetLengthA() const;
	float32 GetLengthB() const;

# 8.9 Gear Joint

만약 정교한 기계 장치를 만들고자 한다면 톱니바퀴가 반드시 필요할 것 입니다. Box2D에서는 원칙적으로 톱니바퀴 모양으로 그려진 shape를 이용하여 톱니바퀴를 만들 수 있습니다. 하지만 이 작업은 효과적이지 않을 뿐 아니라 지루하지 까지 합니다. 또한 맞물린 톱니바퀴가 부드럽게 움직일 수 있도록 배치에도 신경써야 합니다. Box2D는 이런 문제점을 해결할 수 있도록 간단하게 톱니바퀴를 생성하는 gear joint를 제공하고 있습니다. 

If you want to create a sophisticated mechanical contraption you might want to use gears. In principle you can create gears in Box2D by using compound shapes to model gear teeth. This is not very efficient and might be tedious to author. You also have to be careful to line up the gears so the teeth mesh smoothly. Box2D has a simpler method of creating gears:  the gear joint.

gear joint는 revolute joint 혹은 prismatic joint와 연결 가능합니다.
 
The gear joint can only connect revolute and/or prismatic joints.

pulley joint에서 사용된 비율 처럼 gear joint 에도 비율값이 있습니다. 하지만 gear joint 에서는 무조건 음수값이어야 합니다. 한 개는 revolute joint(회전각) 이고 다른 하나는 prismatic joint(이동) 일 때, 기어비는 길이 단위 만큼 이거나 한 개 이상의 길이 일 수 있음을 명심하기 바랍니다.

Like the pulley ratio, you can specify a gear ratio. However, in this case the gear ratio can be negative. Also keep in mind that when one joint is a revolute joint (angular) and the other joint is prismatic (translation), and then the gear ratio will have units of length or one over length.

	coordinate1 + ratio * coordinate2 == constant

하기에는 gear joint 예제가 있습니다. myBodyA와 myBodyB라는 강체는 같은 강체가 아니므로, 두 개의 joint 도 각자에 기인합니다.

Here is an example gear joint. The bodies myBodyA and myBodyB are any bodies from the two joints, as long as they are not the same bodies.
	
	b2GearJointDef jointDef;
	jointDef.bodyA = myBodyA;
	jointDef.bodyB = myBodyB;
	jointDef.joint1 = myRevoluteJoint;
	jointDef.joint2 = myPrismaticJoint;
	jointDef.ratio = 2.0f * b2_pi / myLength;

gear joint 는 두 개의 다른 joint에 걸려있습니다. 이는 깨지기 쉬운 상황을 초래합니다. 해당 joint가 소멸한다면 어떻게 될까요?

Note that the gear joint depends on two other joints. This creates a fragile situation. What happens if those joints are deleted?

> 주의 <br>
> 반드시 gear joint에 포함되어 있는 revolute / prismatic joint를 삭제하기 전에 해당 gear joint를 삭제하기 바랍니다. 그렇지 않으면 프로그램이 gear joint 안에 joint 관련 포인터의 부모를 찾지 못하는 문제가 발생하여 문제가 생길 수 있습니다. 마찬가지로 gear joint 내부의 강체를 소멸시키기 전에 gear joint를 먼저 소멸하는 것이 옳습니다.
> 
> Caution <br>
> Always delete gear joints before the revolute/prismatic joints on the gears. Otherwise your code will crash in a bad way due to the orphaned joint pointers in the gear joint. You should also delete the gear joint before you delete any of the bodies involved.

# 8.10 Mouse Joint

mouse joint는 테스트 케이스 상에서 마우스로 강체를 조정하기 위해서 사용되었습니다. 이는 커서의 현재 위치로 강체를 움직이도록 하기 위해 사용되었습니다. 또한, 회전 관련한 제한도 없습니다.

The mouse joint is used in the testbed to manipulate bodies with the mouse. It attempts to drive a point on a body towards the current position of the cursor. There is no restriction on rotation.

mouse joint에는 목표지점, 힘의 최대값, 주파수, 감쇠비를 정의할 수 있습니다. 목표지점은 강체의 첫 기준점과 일치합니다. 힘의 최대값은 여러 개의 동적 강체와의 상호작용에 대해 격렬한 반응을 보이는 것을 막는데 사용합니다. 이는 원하는 만큼을 사용할 수 있습니다. 주파수와 감쇠비는 distance joint에서 사용하는 스프링과 댐퍼 효과와 비슷하게 작용합니다.

The mouse joint definition has a target point, maximum force, frequency, and damping ratio. The target point initially coincides with the body’s anchor point. The maximum force is used to prevent violent reactions when multiple dynamic bodies interact. You can make this as large as you like. The frequency and damping ratio are used to create a spring/damper effect similar to the distance joint.

보통 mouse joint를 게임을 플레이하기 위해 가져다 쓰곤 합니다만, 위치 변경이나 응답속도에 있어 게임 플레이어의 욕구를 충족시켜주지는 못하므로 해당 용도로 사용하지는 못할 것 입니다. 운동학적 강체에 사용해보는 것을 추천합니다.

 Many users have tried to adapt the mouse joint for game play. Users often want to achieve precise positioning and instantaneous response. The mouse joint doesn’t work very well in that context. You may wish to consider using kinematic bodies instead.

# 8.11 Wheel Joint

wheel joint는 강체 B의 특정 위치에서 부터 연결된 강체 A를 제한합니다. wheel joint 에는 완충용 스프링도 제공합니다. 자세한 내용은 b2WheelJoint.h 및 Car.h 를 참조하십시요.

The wheel joint restricts a point on bodyB to a line on bodyA. The wheel joint also provides a suspension spring. See b2WheelJoint.h and Car.h for details.
 
# 8.12 Weld Joint

weld joint는 두 강체 사이의 상대적인 모든 움직임을 제한합니다. 테스트 케이스 중 Cantilever.h 를 참조하면 해당 joint가 어떻게 작동하는지 알 수 있을 것 입니다.

The weld joint attempts to constrain all relative motion between two bodies. See the Cantilever.h in the testbed to see how the weld joint behaves.

부숴질 수 있는 구조를 정의하고자 할 때 weld joint를 사용하는게 좋습니다. 하지만 Box2D의 solver는 joint 가 강체보다는 약한 재질로 반복적으로 인식하므로, 연속된 강체를 weld joint로 연결한 것은 휘어질 수 있음을 의미합니다.

It is tempting to use the weld joint to define breakable structures. However, the Box2D solver is iterative so the joints are a bit soft. So chains of bodies connected by weld joints will flex.

한 개의 강체에서 시작해서 여러 개의 모양으로 부숴질 수 있는 강체를 만드는 것보다 낫습니다. 강체가 부숴지면, fixture를 소멸시키고 새로 생긴 강체에 다시 fixture를 생성하면 됩니다. 테스트 케이스 상의 Breakable 예제를 참조하시기 바랍니다.

Instead it is better to create breakable bodies starting with a single body with multiple fixtures. When the body breaks, you can destroy a fixture and recreate it on a new body. See the Breakable example in the testbed.

# 8.13 Rope Joint

rope joint는 두 지점간의 최대 거리를 제한합니다. 이는 연속된 강체를 늘리거나 무거운 하중이 가해지는 것을 방지하는데 유용하게 사용될 수 있습니다. b2RopeJoint.h 와 RopeJoint.h 를 참조하시기 바랍니다.

The rope joint restricts the maximum distance between two points. This can be useful to prevent chains of bodies from stretching, even under high load. See b2RopeJoint.h and RopeJoint.h for details.

# 8.14 Friction Joint

friction joint는 하강 관련 마찰에 사용됩니다. joint는 2차원 이동관련 마찰 및 회전 관련 마찰을 제공합니다. b2FrictionJoint.h 및 ApplyForce.h를 참조하시기 바랍니다. 

The friction joint is used for top-down friction. The joint provides 2D translational friction and angular friction. See b2FrictionJoint.h and ApplyForce.h for details.