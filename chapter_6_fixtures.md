# Chapter 6 Fixtures

# 6.1 About

다시 한번 말하지만, 도형은 물리 시뮬레이션 상에서 독립적으로 사용되고 강체에 대한 정보를 모릅니다. 그래서 Box2D는 강체의 모양을 구성하는 도형에 b2Fixture 라는 클래스를 붙이는 방법을 제공합니다. 부착속성(Fixture)은 특성을 가집니다.

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

# 6.2 Fixture Creation<<<<<<< HEAD
Fixture는 fixture의 특성을 생성하고 부모 강체에 해당 특성을 넘김으로써 생성됩니다.

Fixtures are created by initializing a fixture definition and then passing the definition to the parent body.

	b2FixtureDef fixtureDef;
    fixtureDef.shape = &myShape;
	fixtureDef.density = 1.0f;
	b2Fixture* myFixture = myBody->CreateFixture(&fixtureDef);
	

상기의 코드는 강체에 특성을 붙이는 것을 의미합니다. 해당 fixture의 포인터는 해당 강체가 제거될 때, 자동으로 제거되므로 따로 저장해둘 필요는 없습니다. 또한, 한 개의 강체에 여러 개의 fixture를 붙이는 것도 가능합니다.

This creates the fixture and attaches it to the body. You do not need to store the fixture pointer since the fixture will automatically be destroyed when the parent body is destroyed. You can create multiple fixtures on a single body.

강체에서 fixture를 직접 제거하는 것도 가능합니다. 보통 파괴 가능한 물체인 경우 먼저 fixture를 제거하여, 물체가 제거될 때 fixture 제거에 대해 신경 쓰지 않도록 하는 것이 좋습니다.

You can destroy a fixture on the parent body. You may do this to model a breakable object. Otherwise you can just leave the fixture alone and let the body destruction take care of destroying the attached fixtures.

	myBody->DestroyFixture(myFixture);

## Density

밀도는 해당 강체의 질량을 계산하고자 사용합니다. 값은 0 혹은 양수입니다. 안정성을 높이기 위해 밀도값은 거의 비슷한 값을 주는 것을 추천합니다.
밀도값을 주는 것 만으로 질량이 설정되지는 않고, 반드시 ResetMassData를 호출해야만 합니다.

 The fixture density is used to compute the mass properties of the parent body. The density can be zero or positive. You should generally use similar densities for all your fixtures. This will improve stacking stability.
The mass of a body is not adjusted when you set the density. You must call ResetMassData for this to occur.
	
	fixture->SetDensity(5.0f);
	body->ResetMassData();
	
## Friction

마찰은 물체가 현실적으로 미끄러지도록 만들기 위해 사용됩니다. Box2D는 정적 마찰, 동적 마찰 양쪽을 지원하지만 적용되는 값은 같습니다. 마찰은 Box2D 상에 정확히 구현되어 있으며, 마찰력은 쿨롬 마찰이라고 불리우는 수직항력이 비례합니다. 마찰값은 0 부터 1 사이의 값을 사용하며, 음수가 될 수 없습니다. 마찰력은 값이 0일 때 없고, 1에 가까워질수록 강해집니다. 마찰력이 두 개의 모양에 대해 작용할 때, 마찰값은 각각의 강체에 정의가 되어 있어야 합니다. 다음과 같이 합니다:

Friction is used to make objects slide along each other realistically. Box2D supports static and dynamic friction, but uses the same parameter for both. Friction is simulated accurately in Box2D and the friction strength is proportional to the normal force (this is called Coulomb friction). The friction parameter is usually set between 0 and 1, but can be any non-negative value. A friction value of 0 turns off friction and a value of 1 makes the friction strong. When the friction force is computed between two shapes, Box2D must combine the friction parameters of the two parent fixtures. This is done with the geometric mean:

	float32 friction;
	friction = sqrtf(shape1->friction * shape2->friction);
	
따라서, 한 개의 속성값을 0 으로 해놓았다면, 마찰값은 당연하게도 0이 됩니다.

So if one fixture has zero friction then the contact will have zero friction.

## Restitution

복원은 객체의 튕기는 정도를 나타내기 위해 사용합니다. 값은 0부터 1 사이의 값을 사용합니다. 테이블 위에 공을 떨어뜨린다고 가정할 때, 값이 0이라면 튕기지 않습니다. 이를 비탄성총돌이라고 합니다. 값을 1로 준다면, 공의 속도와 동일하게 반사됨을 뜻합니다. 이를 완전 탄성 총돌이라고 합니다. 복원력은 다음의 수식을 통해 합성할 수 있습니다.

Restitution is used to make objects bounce. The restitution value is usually set to be between 0 and 1. Consider dropping a ball on a table. A value of zero means the ball won't bounce. This is called an inelastic collision. A value of one means the ball's velocity will be exactly reflected. This is called a perfectly elastic collision. Restitution is combined using the following formula.

	float32 restitution;
	restitution = b2Max(shape1->restitution, shape2->restitution);
	
충돌 관련 필터링 정보를 통해 게임내 특정 객체에 대한 충돌을 막을 수 있도록 특성을 설정할 수도 있습니다.

Fixtures carry collision filtering information to let you prevent collisions between certain game objects.

모양에 접촉점이 많은 경우, 복원도 즉시 계산됩니다. 왜냐하면 box2d 는 반복적으로 연산을 하기 때문입니다. 충돌속도가 작다면 비탄성충돌이 일어납니다. 이는 강체가 덜덜떠는 문제를 막기 위해서 입니다.

When a shape develops multiple contacts, restitution is simulated approximately. This is because Box2D uses an iterative solver. Box2D also uses inelastic collisions when the collision velocity is small. This is done to prevent jitter.

## Filtering

충돌 필터링은 

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
