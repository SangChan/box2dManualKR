# Chapter 7 Bodies

#7.1 About

강체에는 위치와 속도에 대한 값을 가질 수 있으며, 힘, 토크(회전력), 충격을 가할 수 있습니다. 강체는 동적인 것과 정적인 것, 그리고 운동학적인 것으로 분류할 수 있습니다. 해당 정의는 다음과 같습니다 :

Bodies have position and velocity. You can apply forces, torques, and impulses to bodies. Bodies can be static, kinematic, or dynamic. Here are the body type definitions:

## b2_staticBody

정적 강체는 물리엔진의 물리법칙 하에서 어떤 동작도 하지 않으며, 마치 무한대의 질량을 가진 물체처럼 작동합니다. 내부적으로 Box2D는 질량 및 반대질량에 0을 저장합니다. 정적 강체는 사용자가 직접 움직일 수 있습니다. 또한 정적 강체는 속도값도 0입니다. 정적 강체는 다른 정적 강체 및 운동학적 강체와 충돌을 일으키지 않습니다.

A static body does not move under simulation and behaves as if it has infinite mass. Internally, Box2D stores zero for the mass and the inverse mass. Static bodies can be moved manually by the user. A static body has zero velocity. Static bodies do not collide with other static or kinematic bodies.

## b2_kinematicBody

운동학적 강체는 속도값에 따라 물리엔진의 물리법칙 하에서 동작합니다. 운동학적 강체는 힘에 반응하지 않습니다. 사용자가 직접 움직일 수 있으며, 일반적으로 설정된 속도값에 따라 움직입니다. 운동학적 강체는 마치 무한대의 질량을 가진 물체처럼 움직이지만, 질량 및 반대질량 값은 0으로 Box2D 에서 설정합니다. 운동학적 강체는 다른 정적 강체 및 운동학적 강체와 총돌을 일으키지 않습니다.

A kinematic body moves under simulation according to its velocity. Kinematic bodies do not respond to forces. They can be moved manually by the user, but normally a kinematic body is moved by setting its velocity. A kinematic body behaves as if it has infinite mass, however, Box2D stores zero for the mass and the inverse mass. Kinematic bodies do not collide with other static or kinematic bodies.

## b2_dynamicBody

동적 강체는 모든 물리법칙에 대해 동작합니다. 사용자가 직접 움직일 수 있으나, 대체로 힘에 의해 움직입니다. 동적 강체는 모든 타입의 강체와 충돌할 수 있습니다. 동적 강체는 유한하며, 0이 아닌 값의 질량을 가집니다. 동적 강체의 질량값을 0으로 주면, 자동적으로 1킬로그램이 됩니다.

A dynamic body is fully simulated. They can be moved manually by the user, but normally they move according to forces. A dynamic body can collide with all body types. A dynamic body always has finite, non-zero mass. If you try to set the mass of a dynamic body to zero, it will automatically acquire a mass of one kilogram.

강체는 fixture의 근간이 되며, 강체가 fixture를 가져다 world 내에서 작동할 수 있도록 합니다. Box2D 에서 body 라고 하면 언제나 강체를 뜻합니다. (역주 : 번역상 Body 라는 부분을 전부 강체로 하였습니다.) 이는 하나의 강체에 2개의 fixture를 붙인다면 서로 상대적으로 움직이지 않음을 뜻합니다.

Bodies are the backbone for fixtures. Bodies carry fixtures and move them around in the world. Bodies are always rigid bodies in Box2D. That means that two fixtures attached to the same rigid body never move relative to each other.

fixture 에는 충돌 구조 및 밀도값이 존재합니다. 일반적으로 강체는 fixture로 부터 질량값을 얻어옵니다. 하지만, 강체가 생성된 이후에 질량값을 주는 것도 가능합니다. 하단에서 자세히 설명하겠습니다.

Fixtures have collision geometry and density. Normally, bodies acquire their mass properties from the fixtures. However, you can override the mass properties after a body is constructed. This is discussed below.

일반적으로 사용자는 스스로 만든 강체에 대한 포인터를 스스로 관리 해야 합니다. 왜냐하면 이를 통해 화면에 해당 강체의 위치를 업데이트 하는 등의 용도로 사용해야 하기 때문입니다. 또한 이를 통해 강체의 사용이 끝났을 때, 메모리 상에서 제거하는 용도로도 사용할 수 있습니다.

You usually keep pointers to all the bodies you create. This way you can query the body positions to update the positions of your graphical entities. You should also keep body pointers so you can destroy them when you are done with them.

# 7.2 Body Definition

강체를 만들기 전에 강체를 정의하는 b2BodyDef 를 생성할 필요가 있습니다. 해당 객체에는 강체를 만들고 초기화 하기 위한 값들이 정의되어 있습니다.

Before a body is created you must create a body definition (b2BodyDef). The body definition holds the data needed to create and initialize a body.

Box2D는 외부의 강체정의 값을 강체에 복사해서 집어 넣으므로, 해당 정의에 대한 객체 포인터를 따로 가지고 있지는 않습니다. 그러므로 여러 개의 강체를 만들때 재사용할 수 있습니다.

Box2D copies the data out of the body definition; it does not keep a pointer to the body definition. This means you can recycle a body definition to create multiple bodies.

강체를 정의함에 있어 중요한 값들을 알아 보도록 하겠습니다.

Let’s go over some of the key members of the body definition.

## Body Type

본 챕터의 시작 부분에서 언급한 바와 같이, 강체에는 정적, 동적, 운동학적 강체가 존재합니다. 반드시 생성할 때 강체의 타입을 정하길 바랍니다. 이후에 바꾸려고 할 때는 여러가지로 낭비입니다.

As discussed at the beginning of this chapter, there are three different body types: static, kinematic, and dynamic. You should establish the body type at creation because changing the body type later is expensive.

	bodyDef.type = b2_dynamicBody;

필수적으로 강체의 타입을 설정해야만 합니다.

Setting the body type is mandatory.

## Position and Angle

강체 생성시에 강체의 위치를 설정하여 해당 위치에서 생성되도록 할 수 있습니다. 이는 world의 시작점에서 생성한 후, 해당 위치로 이동하는 것보다 퍼포먼스상 낫습니다.

The body definition gives you the chance to initialize the position of the body on creation. This has far better performance than creating the body at the world origin and then moving the body.

> 주의 <br>
> world 의 (0,0) 위치에서 생성한 다음 움직이지 말길 바랍니다. 여러 개의 강체를 (0,0) 에서 만드는 것은 퍼포먼스에 악영향을 끼칩니다.

> Caution <br>
> Do not create a body at the origin and then move it. If you create several bodies at the origin, then performance will suffer.

강체에는 흥미롭게 보아야할 두가지 요소가 있습니다. 첫번째 요소는 강체의 시작점입니다. fixture 나 joint 정보는 강체의 시작점에 관련되어 부착됩니다. 두번째 요소는 질량중심점입니다. 질량중심점은 해당 강체의 모양값을 통해 질량이 분포되는 것을 통해 정의되거나, b2MassData를 통해 정의될 수 있습니다. Box2D는 내부적으로 질량중심점의 위치를 사용해 물리연산을 진행합니다. 예를 들어 b2Body는 질량중심점을 위해 선속도를 저장합니다.

A body has two main points of interest. The first point is the body's origin. Fixtures and joints are attached relative to the body's origin. The second point of interest is the center of mass. The center of mass is determined from mass distribution of the attached shapes or is explicitly set with b2MassData. Much of Box2D's internal computations use the center of mass position. For example b2Body stores the linear velocity for the center of mass.

강체 정보를 사용자가 만들때, 질량중심점이 어디인지를 모를 가능성이 높습니다. 이럴때는 강체의 시작점에 설정하면 됩니다. 또한, 강체의 각도는 라디안 단위로 설정할 수 있는데, 이는 질량중심점의 위치에 영향을 받지 않습니다. 만약 질량관련 설정을 나중에 변경한다면 질량중심점 또한 변경될 수 있지만, 시작점이 바뀌지 않는다면 강체에 설정된 shape이나 joint관련 설정도 움직이지는 않을 것입니다. 

When you are building the body definition, you may not know where the center of mass is located. Therefore you specify the position of the body's origin. You may also specify the body's angle in radians, which is not affected by the position of the center of mass. If you later change the mass properties of the body, then the center of mass may move on the body, but the origin position does not change and the attached shapes and joints do not move.

	bodyDef.position.Set(0.0f, 2.0f);   // the body's origin position.
	bodyDef.angle = 0.25f * b2_pi;      // the body's angle in radians.

## Damping

damping 은 강체에 가해지는 world 의 속도를 감속하기 위해서 사용됩니다. damping 은 마찰과는 달리 닿아있지 않더라도 작용합니다. damping 은 마찰을 대신 할 수 없으므로, 두 가지 모두 사용됩니다.

Damping is used to reduce the world velocity of bodies. Damping is different than friction because friction only occurs with contact. Damping is not a replacement for friction and the two effects should be used together.

damping 관련 값은 0 부터 무한대까지 될 수 있으며, 0이 의미하는 것은 damping을 하지 않겠다는 것입니다. 무한대값은 damping 값을 최대한 사용한다는 의미입니다. 일반적으로 damping 값은 0 부터 0.1 까지 사용할 것 입니다. 필자는 강체가 붕붕뜨는 것 같아 보여 선형감쇠(linear damping)은 사용하지 않습니다.

Damping parameters should be between 0 and infinity, with 0 meaning no damping, and infinity meaning full damping. Normally you will use a damping value between 0 and 0.1. I generally do not use linear damping because it makes bodies look floaty.

	bodyDef.linearDamping = 0.0f;
	bodyDef.angularDamping = 0.01f;
	
damping은 안정성과 성능을 위한 근사치일 뿐 입니다. 작은 damping 값에 의한 감쇠효과는 world 의 time step과 별 상관이 없지만, 큰 damping 값에 의한 감쇠효과는 world 의 time step에 따라 달라질 수 있습니다. 물론, 고정된 time step 값을 사용하면 문제가 안되므로 이 방식을 추천하는 바입니다.

Damping is approximated for stability and performance. At small damping values the damping effect is mostly independent of the time step. At larger damping values, the damping effect will vary with the time step. This is not an issue if you use a fixed time step (recommended).

## Gravity Scale

각각의 강체에 중력의 크기를 설정할 수 있습니다. 중력값을 높이는 것은 안정성을 낮추는 결과를 낳을 수 있습니다.

You can use the gravity scale to adjust the gravity on a single body. Be careful though, increased gravity can decrease stability.

	// 중력 크기값을 0으로 하면 강체는 둥둥 뜰 것입니다.
	bodyDef.gravityScale = 0.0f;

## Sleep Parameters

잠자기 기능은 왜 있을까요? 물리엔진에서 강체에 물리법칙이 적용되는 것은 비용이 비싼 일입니다. 그러므로 조금이라도 적게 연산하도록 하는 것이 좋습니다. 물리법칙이 적용되지 않을때는 강체를 쉬도록 하여 연산이 되지 않도록 하는 것이 좋습니다.

What does sleep mean? Well it is expensive to simulate bodies, so the less we have to simulate the better. When a body comes to rest we would like to stop simulating it.

Box2D에서 강체(혹은 여러 강체의 그룹)를 쉬게 하는 것은, 강체가 잠자기 상태에 빠져 CPU의 자원을 아주 조금 점유하는 것을 뜻합니다. 만약 강체가 깬 상태가 되고 다른 잠자기 상태의 강체와 충돌한다면 잠자던 강체도 깬 상태가 됩니다. 또한 잠자는 강체에 붙어 있는 joint 나 contact 강체가 파괴되는 경우에도 깨어나게 됩니다. 수동으로 잠자는 강체를 깨울 수도 있습니다.

When Box2D determines that a body (or group of bodies) has come to rest, the body enters a sleep state which has very little CPU overhead. If a body is awake and collides with a sleeping body, then the sleeping body wakes up. Bodies will also wake up if a joint or contact attached to them is destroyed. You can also wake a body manually.

강체 정의 상에는 잠자기 상태가 될 수 있도록 하거나, 생성시에 잠자기 상태로 생성되도록 할 수 있습니다.

The body definition lets you specify whether a body can sleep and whether a body is created sleeping.

	bodyDef.allowSleep = true;
	bodyDef.awake = true;
	
## Fixed Rotation

캐릭터 같은 강체에 대해 회전이 불가능 해야 할 수 있습니다. 이런 강체와 강체에 속한 강체들은 회전하면 안됩니다. 다음과 같이 설정하여 고정할 수 있습니다:

You may want a rigid body, such as a character, to have a fixed rotation. Such a body should not rotate, even under load. You can use the fixed rotation setting to achieve this:

	bodyDef.fixedRotation = true;

회전을 고정하는 플래그 값에 따라 회전 관성이 발생되더라도 0으로 값이 반전되도록 합니다.

The fixed rotation flag causes the rotational inertia and its inverse to be set to zero.

## Bullets

게임을 구현한다는 것은 특정 프레임 레이트에 따라 연속적인 영상을 생성하는 것을 의미합니다. 이를 이산 시뮬레이션이라고 하는데, 해당 상황하에서 물리엔진은 한 time step 에 대해 강체가 많은 움직임을 보일 수 있습니다. 물리엔진이 큰 움직임에 대해 연산하지 않는다면, 서로 다른 객체가 부자연스럽게 통과하는 일이 벌어질 것 입니다. 이를 터널링현상이라고 부릅니다.

Game simulation usually generates a sequence of images that are played at some frame rate. This is called discrete simulation. In discrete simulation, rigid bodies can move by a large amount in one time step. If a physics engine doesn't account for the large motion, you may see some objects incorrectly pass through each other. This effect is called tunneling.

Box2D 는 CCD(지속적 충돌 감지)를 통해 정적 강체에 대한 동적 강체의 터널링 현상을 방지합니다. 새 위치로 이동하는 중에 구 위치의 모양을 지우며 수행되는데, 물리엔진은 이때 새로운 충돌을 찾으며, 해당 충돌의 TOI(충돌시각)을 계산합니다. 강체가 최초의 TOI 시점으로 이동한다면, 나머지 time step을 위해 중단이 발생합니다.

By default, Box2D uses continuous collision detection (CCD) to prevent dynamic bodies from tunneling through static bodies. This is done by sweeping shapes from their old position to their new positions. The engine looks for new collisions during the sweep and computes the time of impact (TOI) for these collisions. Bodies are moved to their first TOI and then halted for the remainder of the time step.

일반적으로 CCD는 동적 강체 사이에서는 물리엔진 연산에 대한 성능유지를 위해 사용되지 않습니다. 게임의 시나리오상 동적 강체에 대해 CCD를 사용해야만 할 수 있습니다. 예를 들어, 동적 강체로 이루어진 벽돌에 대해 빠른 속도로 날아가는 총알을 발사하고 싶을 수 있습니다. CCD가 없다면, 해당 총알은 벽돌에 대해 터널링 효과가 발생하여 통과해버리고 말 것 입니다.

Normally CCD is not used between dynamic bodies. This is done to keep performance reasonable. In some game scenarios you need dynamic bodies to use CCD. For example, you may want to shoot a high speed bullet at a stack of dynamic bricks. Without CCD, the bullet might tunnel through the bricks.

Box2D에서는 이처럼 빠르게 움직이는 객체를 bullet 이라고 지칭합니다. Bullet 속성을 가진 강체는 정적 강체와 동적 강체에 대해 CCD를 사용할 수 있습니다. 사용자는 게임상에서 어떤 강체가 bullet 이어야 하는지 결정하기만 하면 됩니다. 강체를 bullet 취급하려면 다음과 같이 하면 됩니다.

Fast moving objects in Box2D can be labeled as bullets. Bullets will perform CCD with both static and dynamic 
bodies. You should decide what bodies should be bullets based on your game design. If you decide a body should be treated as a bullet, use the following setting.

	bodyDef.bullet = true;

bullet 속성은 동적 강체만 사용할 수 있습니다.

The bullet flag only affects dynamic bodies.

Box2D는 순서에 따라 지속적으로 충돌을 체크하므로, 빨리 움직이는 강체에 대해서는 제대로 작동하지 않을 수 있습니다.

Box2D performs continuous collision sequentially, so bullets may miss fast moving bodies.

## Activation

강체는 생성하되, 물리법칙이 적용되지 않도록 하고 싶을 수 있습니다. 이 상태는 잠자기 상태와 비슷하지만 다른 강체와의 충돌로 인해 깨어나지 않으며, 강체의 fixture가 광범위하게 존재하지 않습니다. 이는 레이캐스트나 충돌 체크에 의해 감지 되지 않음을 뜻합니다.

You may wish a body to be created but not participate in collision or dynamics. This state is similar to sleeping except the body will not be woken by other bodies and the body's fixtures will not be placed in the broad-phase. This means the body will not participate in collisions, ray casts, etc.

사용자는 강체를 활성화되지 않은 상태로 생성한 후, 나중에 활성화 시킬 수 있습니다.

You can create a body in an inactive state and later re-activate it.

	bodyDef.active = true;

활성화되지 않은 강체에 연결된 joint 또한 물리연산 대상이 아니므로, 강체를 활성화 할때, 연결된 joint도 관리해주어야 합니다.

Joints may be connected to inactive bodies. These joints will not be simulated. You should be careful when you activate a body that its joints are not distorted.

## User Data

user data 는 자료형이 확정되지 않은 상태의 포인터입니다. 해당 프로그램의 객체를 강체와 연결하는 것이 가능하도록 합니다. 프로그램 내 모든 강체의 user data에 같은 자료형을 쓸 수 있도록 일관성을 유지하는게 좋습니다.

User data is a void pointer. This gives you a hook to link your application objects to bodies. You should be consistent to use the same object type for all body user data.

	b2BodyDef bodyDef;
	bodyDef.userData = &myActor;

# 7.3 Body Factory

강체는 world 클래스에서 제공되는 body factory를 통해 생성 및 소멸 될 수 있습니다. 이를 통해 world가 효율적으로 강체를 메모리에 적재할 수 있고 world 의 자료구조에 강체를 추가할 수 있게 해줍니다. 
강체는 질량 속성에 따라 동적이거나 정적 일 수 있습니다. 이 두 가지 타입은 같은 방식으로 생성되고 소멸될 수 있습니다.

Bodies are created and destroyed using a body factory provided by the world class. This lets the world create the body with an efficient allocator and add the body to the world data structure.
Bodies can be dynamic or static depending on the mass properties. Both body types use the same creation and destruction methods.

	b2Body* dynamicBody = myWorld->CreateBody(&bodyDef);
	... do stuff ...
	myWorld->DestroyBody(dynamicBody);
	dynamicBody = NULL;

> 주의 <br>
> new 혹은 malloc 를 이용해 강체를 생성하면 안됩니다. world 에서 강체에 대해 알지 못할 경우, 초기화도 불가합니다.
> 
> Caution <br>
> You should never use new or malloc to create a body. The world won't know about the body and the body won't be properly initialized.

정적 강체는 다른 강체의 영향하에서 움직이지 않도록 합니다. 정적 강체를 직접 움직일 수 있지만, 둘 이상의 정적 강체 사이에 동적 강체가 깔리도록 해서는 안됩니다. 마찰은 정적 강체를 움직일 때 정상적으로 작동하지 않을 것 입니다. 각각 한 개의 모양을 가지는 정적 강체 여러 개를 만드는 것보다 여러 개의 모양을 가지는 한 개의 정적 강체를 만드는 것이 더 빠릅니다. 내부적으로, Box2D는 정적 강체에 질량 및 반대 질량을 0으로 설정합니다. 이를 통해 대부분의 알고리즘에서 정적 강체를 특별한 경우로 취급하여 연산하지 않도록 해줍니다.

Static bodies do not move under the influence of other bodies. You may manually move static bodies, but you should be careful so that you don't squash dynamic bodies between two or more static bodies. Friction will not work correctly if you move a static body. Static bodies never collide with static or kinematic bodies. It is faster to attach several shapes to a static body than to create several static bodies with a single shape on each one. Internally, Box2D sets the mass and inverse mass of static bodies to zero. This makes the math work out so that most algorithms don't need to treat static bodies as a special case.

Box2D는 강체의 정의나 강체가 가지는 여러 데이터에 대한 참조를 따로 저장해두지는 않습니다. (user data에 대한 포인터는 예외입니다.) 그러므로 임시 강체 정의를 만들어 동일한 강체의 정의에 사용할 수 있습니다.

Box2D does not keep a reference to the body definition or any of the data it holds (except user data pointers). So you can create temporary body definitions and reuse the same body definitions.

Box2D는 모든 종류의 정리작업을 수행하며, b2World 객체를 삭제하여 강체가 소멸되는 것을 방지할 수 있습니다. 그러나 사용자의 게임엔진상에서 강체의 포인터에 null이 들어갈 수 있음을 항상 염두에 두어야 합니다.
강체를 소멸할때, 부착되어 있던 fixture와 joint도 자동적으로 소멸됩니다. 이는 shape과 joint 포인터를 관리하는데 있어 매우 중요한 점입니다.

Box2D allows you to avoid destroying bodies by deleting your b2World object, which does all the cleanup work for you. However, you should be mindful to nullify body pointers that you keep in your game engine.
When you destroy a body, the attached fixtures and joints are automatically destroyed. This has important implications for how you manage shape and joint pointers.

# 7.4 Using a Body

강체 생성 후, 강체에 대해 여러가지 수행가능한 작업이 있습니다. 여기에는 질량 특성 설정, 위치 및 속도에 접근, 힘을 가함, 포인트 및 벡터의 변환등이 있습니다.

After creating a body, there are many operations you can perform on the body. These include setting mass properties, accessing position and velocity, applying forces, and transforming points and vectors.

## Mass Data

모든 강체는 질량(스칼라값), 질량중심점(2차원 벡터), 회전관성(스칼라값)을 가지고 있습니다. 정적 강체에서는 질량과 회전관성은 0으로 설정됩니다. 강체가 회전에 대해 고정되어 있다면, 회전관성도 0입니다.

Every body has a mass (scalar), center of mass (2-vector), and rotational inertia (scalar). For static bodies, the mass and rotational inertia are set to zero. When a body has fixed rotation, its rotational inertia is zero.

바디에 fixture가 추가될 때, 질량 속성도 자동으로 설정됩니다. 또한, 실행중에 강체의 질량을 조정할 수 있습니다. 게임시나리오 상, 질량의 변동이 필요한 경우 일반적으로 많이들 합니다.

Normally the mass properties of a body are established automatically when fixtures are added to the body. You can also adjust the mass of a body at run-time. This is usually done when you have special game scenarios that require altering the mass.

	void SetMassData(const b2MassData* data);

강체의 질량을 직접 설정한 후, fixture 에서 설정한 최초값으로 돌아가고 싶다면 다음과 같이 하면 된다:

After setting a body's mass directly, you may wish to revert to the natural mass dictated by the fixtures. You can do this with:

	void ResetMassData();
	
강체의 질량 데이터는 다음과 같은 함수를 통해 확인이 가능합니다:

The body's mass data is available through the following functions:

	float32 GetMass() const;
	float32 GetInertia() const;
	const b2Vec2& GetLocalCenter() const;
	void GetMassData(b2MassData* data) const;
	
## State Information

강체의 상태에 대해서는 여러 방향으로 제공하고 있습니다. 다음의 함수를 통해 효율적으로 강체의 여러가지 상태를 알 수 있습니다:

There are many aspects to the body's state. You can access this state data efficiently through the following functions:

	void SetType(b2BodyType type);
	b2BodyType GetType();

	void SetBullet(bool flag);
	bool IsBullet() const;

	void SetSleepingAllowed(bool flag);
	bool IsSleepingAllowed() const;

	void SetAwake(bool flag);
	bool IsAwake() const;

	void SetActive(bool flag);
	bool IsActive() const;

	void SetFixedRotation(bool flag);
	bool IsFixedRotation() const;

## Position and Velocity

강체의 회전 및 위치에 대한 정보에 접근할 수 있습니다. 일반적으로 게임상의 캐릭터를 화면에 묘사하기 위함입니다. 보통 움직임은 Box2D를 통해 연산되도록 하지만, 직접 위치를 설정하는 것도 가능합니다.

You can access the position and rotation of a body. This is common when rendering your associated game actor. You can also set the position, although this is less common since you will normally use Box2D to simulate movement.

	bool SetTransform(const b2Vec2& position, float32 angle);
	const b2Transform& GetTransform() const;
	const b2Vec2& GetPosition() const;
	float32 GetAngle() const;

화면 일부 혹은 전체에 대비하여 질량중심점의 위치에 접근하는 것도 가능합니다. Box2D는 내부적으로 질량중심점을 연산에 사용하는 경우가 많습니다. 그렇지만, 일반적으로 접근을 할 필요는 없습니다. 보통 강체의 변환을 통해 작동하도록 하는 경우가 일반적일 것입니다. 예를 들어, 사각형 강체를 가지고 있다고 하겠습니다. 강체의 시작점이 사각형의 꼭지점이라면, 질량중심점은 사각형의 중심에 위치합니다.

You can access the center of mass position in local and world coordinates. Much of the internal simulation in Box2D uses the center of mass. However, you should normally not need to access it. Instead you will usually work with the body transform. For example, you may have a body that is square. The body origin might be a corner of the square, while the center of mass is located at the center of the square.

	const b2Vec2& GetWorldCenter() const;
	const b2Vec2& GetLocalCenter() const;

또한 선형속도, 각속도에 접근할 수 있습니다. 선형속도는 질량중심점을 위한 것으로, 질량 속성의 변환에 따라 다른 값이 될 수 있습니다.

You can access the linear and angular velocity. The linear velocity is for the center of mass. Therefore, the linear velocity may change if the mass properties change.