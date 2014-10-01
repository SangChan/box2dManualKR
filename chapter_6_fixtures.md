# Chapter 6 Fixtures

# 6.1 About

다시 한번 말하지만, 도형은 물리 시뮬레이션 상에서 독립적으로 사용되고 강체에 대한 정보를 모릅니다. 그래서 Box2D는 강체의 모양을 구성하는 도형에 b2Fixture 라는 클래스를 붙이는 방법을 제공합니다. 부착속성(Fixture)은 다음의 값을 가집니다.

Recall that shapes don’t know about bodies and may be used independently of the physics simulation. Therefore Box2D provides the b2Fixture class to attach shapes to bodies. Fixtures hold the following:

* 1개의 형태
* 넓은 범위를 아우르는 약속값 broad-phase proxies
* 밀도, 마찰, 탄성 density, friction, and restitution
* 충돌을 걸러내는데 사용하는 플래그값 collision filtering flags
* 부모 강체를 가르키는 포인터값 back pointer to the parent body
* 유저 데이터 user data
* 센서 관련 플래그값 sensor flag

위의 내용은 하단에 기술합니다.

These are described in the following sections.

# 6.2 Fixture Creation

속성은 속성의 정의를 내리는 과정을 통해 생성되며, 이를 통해 부모인 강체에 해당 정의가 전달됩니다.

Fixtures are created by initializing a fixture definition and then passing the definition to the parent body.

	b2FixtureDef fixtureDef;
    fixtureDef.shape = &myShape;
	fixtureDef.density = 1.0f;
	b2Fixture* myFixture = myBody->CreateFixture(&fixtureDef);
	
속성을 생성하여 강체에 적용하였습니다. 속성을 가르키는 포인터를 저장할 필요는 없습니다. 왜냐하면, 부모인 강체가 메모리에서 해제될 때, 자동적으로 속성도 해제되기 때문입니다. 한 개의 강체는 여러 개의 속성을 가질 수도 있습니다.

This creates the fixture and attaches it to the body. You do not need to store the fixture pointer since the fixture will automatically be destroyed when the parent body is destroyed. You can create multiple fixtures on a single body.

강체가 가진 속성을 직접 해제할 수도 있습니다. 아마도 부서질 수 있는 객체에 대해 사용할 것으로 사료됩니다. 강체를 파괴할 때 강체에 포함된 속성을 해제하는 것에 신경을 쓰지 않으면, 속성이 메모리에 잔존 할 수 있습니다.

You can destroy a fixture on the parent body. You may do this to model a breakable object. Otherwise you can just leave the fixture alone and let the body destruction take care of destroying the attached fixtures.

	myBody->DestroyFixture(myFixture);

## Density

Density 값을 통해 해당 강체의 질량에 대한 계산을 할 수 있습니다. Density는 0 혹은 양수여야 합니다. 안정성을 높이기 위해 일반적으로 모든 속성에 대해 비슷한 값을 주는 것을 권장합니다.
Density 값을 지정하는 시점에 강체의 질량이 조정되지 않을 수 있습니다. 이 때에는 반드시 ResetMassDate 메소드를 불러 주시기 바랍니다.

 The fixture density is used to compute the mass properties of the parent body. The density can be zero or positive. You should generally use similar densities for all your fixtures. This will improve stacking stability.
The mass of a body is not adjusted when you set the density. You must call ResetMassData for this to occur.
	
	fixture->SetDensity(5.0f);
	body->ResetMassData();
	
## Friction
Friction 값은 각각의 물체가 사실적으로 미끄러지듯이 움직일 수 있도록 사용됩니다. Box2D는 정적인 마찰과 동적인 마찰을 지원하는데, 입력되는 값은 양쪽 모두 동일합니다. Box2D에서는 Coulomb 마찰이라는 노말 벡터에 

Friction is used to make objects slide along each other realistically. Box2D supports static and dynamic friction, but uses the same parameter for both. Friction is simulated accurately in Box2D and the friction strength is proportional to the normal force (this is called Coulomb friction). The friction parameter is usually set between 0 and 1, but can be any non-negative value. A friction value of 0 turns off friction and a value of 1 makes the friction strong. When the friction force is computed between two shapes, Box2D must combine the friction parameters of the two parent fixtures. This is done with the geometric mean:

	float32 friction;
	friction = sqrtf(shape1->friction * shape2->friction);
	
So if one fixture has zero friction then the contact will have zero friction.

## Restitution
Restitution is used to make objects bounce. The restitution value is usually set to be between 0 and 1. Consider dropping a ball on a table. A value of zero means the ball won't bounce. This is called an inelastic collision. A value of one means the ball's velocity will be exactly reflected. This is called a perfectly elastic collision. Restitution is combined using the following formula.

	float32 restitution;
	restitution = b2Max(shape1->restitution, shape2->restitution);
	
Fixtures carry collision filtering information to let you prevent collisions between certain game objects.

When a shape develops multiple contacts, restitution is simulated approximately. This is because Box2D uses an iterative solver. Box2D also uses inelastic collisions when the collision velocity is small. This is done to prevent jitter.

## Filtering
Collision filtering allows you to prevent collision between fixtures. For example, say you make a character that rides a bicycle. You want the bicycle to collide with the terrain and the character to collide with the terrain, but you don't want the character to collide with the bicycle (because they must overlap). Box2D supports such collision filtering using categories and groups.

Box2D supports 16 collision categories. For each fixture you can specify which category it belongs to. You also specify what other categories this fixture can collide with. For example, you could specify in a multiplayer game that all players don't collide with each other and monsters don't collide with each other, but players and monsters should collide. This is done with masking bits. For example:

	playerFixtureDef.filter.categoryBits = 0x0002;
	monsterFixtureDef.filter.categoryBits = 0x0004;
	playerFixtureDef.filter.maskBits = 0x0004;
	monsterFixtureDef.filter.maskBits = 0x0002;

Here is the rule for a collision to occur:

	uint16 catA = fixtureA.filter.categoryBits;
	uint16 maskA = fixtureA.filter.maskBits;
	uint16 catB = fixtureB.filter.categoryBits;
	uint16 maskB = fixtureB.filter.maskBits;

	if ((catA & maskB) != 0 && (catB & maskA) != 0)
	{
		// fixtures can collide
	}
	
Collision groups let you specify an integral group index. You can have all fixtures with the same group index always collide (positive index) or never collide (negative index). Group indices are usually used for things that are somehow related, like the parts of a bicycle. In the following example, fixture1 and fixture2 always collide, but fixture3 and fixture4 never collide.

	fixture1Def.filter.groupIndex = 2;
	fixture2Def.filter.groupIndex = 2;
	fixture3Def.filter.groupIndex = -8;
	fixture4Def.filter.groupIndex = -8;
	
Collisions between fixtures of different group indices are filtered according the category and mask bits. In other words, group filtering has higher precedence than category filtering.

Note that additional collision filtering occurs in Box2D. Here is a list:

* A fixture on a static body can only collide with a dynamic body.
* A fixture on a kinematic body can only collide with a dynamic body.
* Fixtures on the same body never collide with each other.
* You can optionally enable/disable collision between fixtures on bodies connected by a joint.

Sometimes you might need to change collision filtering after a fixture has already been created. You can get and set the b2Filter structure on an existing fixture using b2Fixture::GetFilterData and b2Fixture::SetFilterData. Note that changing the filter data will not add or remove contacts until the next time step (see the World class).

# 6.3 Sensors

Sometimes game logic needs to know when two fixtures overlap yet there should be no collision response. This is done by using sensors. A sensor is a fixture that detects collision but does not produce a response.

You can flag any fixture as being a sensor. Sensors may be static or dynamic. Remember that you may have multiple fixtures per body and you can have any mix of sensors and solid fixtures.

Sensors do not generate contact points. There are two ways to get the state of a sensor:

1. b2Contact::IsTouching
1. b2ContactListener::BeginContact and EndContact
