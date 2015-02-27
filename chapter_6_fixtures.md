# Chapter 6 Fixtures

# 6.1 About

다시 한번 말하지만, 도형은 물리 시뮬레이션 상에서 독립적으로 사용되고 강체에 대한 정보를 모릅니다. 그래서 Box2D는 강체의 모양을 구성하는 도형에 b2Fixture 라는 클래스를 붙이는 방법을 제공합니다. 부착속성(Fixture)은 특성을 가집니다.


* 1개의 형태
* 넓은 범위를 아우르는 약속값
* 밀도, 마찰, 탄성
* 충돌을 걸러내는데 사용하는 플래그값
* 부모 강체를 가르키는 포인터값
* 유저 데이터
* 센서 관련 플래그값

위의 내용은 하단에 기술합니다.

# 6.2 Fixture Creation

Fixture는 fixture의 특성을 생성하고 부모 강체에 해당 특성을 넘김으로써 생성됩니다.

	b2FixtureDef fixtureDef;
    fixtureDef.shape = &myShape;
	fixtureDef.density = 1.0f;
	b2Fixture* myFixture = myBody->CreateFixture(&fixtureDef);
	

상기의 코드는 강체에 특성을 붙이는 것을 의미합니다. 해당 fixture의 포인터는 해당 강체가 제거될 때, 자동으로 제거되므로 따로 저장해둘 필요는 없습니다. 또한, 한 개의 강체에 여러 개의 fixture를 붙이는 것도 가능합니다.

강체에서 fixture를 직접 제거하는 것도 가능합니다. 보통 파괴 가능한 물체인 경우 먼저 fixture를 제거하여, 물체가 제거될 때 fixture 제거에 대해 신경 쓰지 않도록 하는 것이 좋습니다.

	myBody->DestroyFixture(myFixture);

## Density

밀도는 해당 강체의 질량을 계산하고자 사용합니다. 값은 0 혹은 양수입니다. 안정성을 높이기 위해 밀도값은 거의 비슷한 값을 주는 것을 추천합니다.
밀도값을 주는 것 만으로 질량이 설정되지는 않고, 반드시 ResetMassData를 호출해야만 합니다.
	
	fixture->SetDensity(5.0f);
	body->ResetMassData();
	
## Friction

마찰은 물체가 현실적으로 미끄러지도록 만들기 위해 사용됩니다. Box2D는 정적 마찰, 동적 마찰 양쪽을 지원하지만 적용되는 값은 같습니다. 마찰은 Box2D 상에 정확히 구현되어 있으며, 마찰력은 쿨롬 마찰이라고 불리우는 수직항력이 비례합니다. 마찰값은 0 부터 1 사이의 값을 사용하며, 음수가 될 수 없습니다. 마찰력은 값이 0일 때 없고, 1에 가까워질수록 강해집니다. 마찰력이 두 개의 모양에 대해 작용할 때, 마찰값은 각각의 강체에 정의가 되어 있어야 합니다. 다음과 같이 합니다:


	float32 friction;
	friction = sqrtf(shape1->friction * shape2->friction);
	
따라서, 한 개의 속성값을 0 으로 해놓았다면, 마찰값은 당연하게도 0이 됩니다.

## Restitution

복원은 객체의 튕기는 정도를 나타내기 위해 사용합니다. 값은 0부터 1 사이의 값을 사용합니다. 테이블 위에 공을 떨어뜨린다고 가정할 때, 값이 0이라면 튕기지 않습니다. 이를 비탄성총돌이라고 합니다. 값을 1로 준다면, 공의 속도와 동일하게 반사됨을 뜻합니다. 이를 완전 탄성 총돌이라고 합니다. 복원력은 다음의 수식을 통해 합성할 수 있습니다.

	float32 restitution;
	restitution = b2Max(shape1->restitution, shape2->restitution);
	
충돌 관련 필터링 정보를 통해 게임내 특정 객체에 대한 충돌을 막을 수 있도록 특성을 설정할 수도 있습니다.

모양에 접촉점이 많은 경우, 복원도 즉시 계산됩니다. 왜냐하면 box2d 는 반복적으로 연산을 하기 때문입니다. 충돌속도가 작다면 비탄성충돌이 일어납니다. 이는 강체가 덜덜떠는 문제를 막기 위해서 입니다.

## Filtering

충돌 필터링은 각각의 fixture 간의 충돌을 막을 수 있도록 합니다. 예를 들어, 캐릭터를 자전거에 타게 만든다고 가정해 보겠습니다. 자전거와 캐릭터는 각각 지면과 충돌하도록 해야 하겠지만, 캐릭터와 자전거는 서로 겹칠 수 있어야 하므로 이 두개간에 충돌이 일어나서는 안됩니다. Box2D는 이러한 충돌 필터링을 카테고리와 그룹 기능을 이용해 지원합니다.

Box2D는 16개의 충돌 카테고리를 지원합니다. 각각의 fixture 에 대해 카테고리를 지정해주는 것이 가능합니다. 또한 이를 통해 서로 충돌할 수 있는 다른 카테고리를 지정할 수 있습니다. 예를 들어, 멀티플레이어 게임을 만든다고 할 때, 플레이어들은 서로간에 충돌하면 안되고 몬스터도 몬스터간에 충돌을 하면 안되겠지만 플레이어와 몬스터간에는 충돌이 일어나야 합니다. 이럴때는 다음과 같은 마스킹 비트를 사용합니다. 다음의 예를 참고하세요:

	playerFixtureDef.filter.categoryBits = 0x0002;
	monsterFixtureDef.filter.categoryBits = 0x0004;
	playerFixtureDef.filter.maskBits = 0x0004;
	monsterFixtureDef.filter.maskBits = 0x0002;

충돌이 일어나게 하기 위해서 다음과 같은 조건식을 사용하면 됩니다.

	uint16 catA = fixtureA.filter.categoryBits;
	uint16 maskA = fixtureA.filter.maskBits;
	uint16 catB = fixtureB.filter.categoryBits;
	uint16 maskB = fixtureB.filter.maskBits;

	if ((catA & maskB) != 0 && (catB & maskA) != 0)
	{
		// fixtures can collide
	}

해당 구성 그룹 인덱스 번호를 지정함으로써 충돌 그룹이 되는데, 양수로 된 인덱스 번호를 동일하게 가지는 것 끼리는 항상 충돌하며, 음수로 된 인덱스 번호를 동일하게 가지는 경우, 충돌이 일어나지 않습니다. 그룹 인덱스는 보통 자전거의 부품들 처럼 서로 관련 있는 것에 사용됩니다. 다음의 예제에서 fixture1 과 fixture2 는 항상 충돌하지만, fixture3 과 fixture4는 절대 총돌하지 않습니다.

	fixture1Def.filter.groupIndex = 2;
	fixture2Def.filter.groupIndex = 2;
	fixture3Def.filter.groupIndex = -8;
	fixture4Def.filter.groupIndex = -8;

인덱스 그룹 번호가 다른 것 끼리의 충돌을 필터링 하기 위해서는 카테고리와 마스크 비트를 사용합니다. 바꿔말하자면, 그룹 기능을 이용한 필터링이 더 상위에 있는 것입니다.

Box2D에는 다음과 같은 충돌 필터링 기능이 더 존재합니다.

* 정적 강체는 오로지 동적 강체와 충돌할 수 있습니다.
* 키네마틱 강체(챕터7 참조)는 오로지 동적 강체와 충돌할 수 있습니다.
* 한 강체 내에 존재하는 fixture 끼리는 절대 충돌하지 않습니다.
* 관절구조로 연결된 강체간의 충돌은 개발자가 조절할 수 있습니다.

Fixture를 이미 만들어둔 이후에 충돌 필터링의 내용을 바꿔야 할 수 있습니다. 이럴때는 b2Fixture::GetFilterData 를 통해 데이터를 b2Filter 형태로 받아온 후, b2Fixture::SetFilterData을 통해 값을 바꾸는 것이 가능합니다. 이때, 바뀐 데이터는 다음 타임 스텝까지 추가 및 제거에 대한 접근이 되지 않습니다. (챕터 10 월드 클래스 참조)

# 6.3 Sensors

때때로, 게임 로직상 두 개의 fixture 가 충돌 반응 없이 겹쳐야 할 수 있습니다. 이런 기능은 센서라는 것을 통해 구현할 수 있습니다. 센서를 통해 두 물체의 충돌을 충돌반응 없이 검출할 수 있습니다.

정적이거나 동적인 fixture를 센서로 지정할 수 있습니다. 사용자는 한개의 강체에 여러 개의 fixture를 조합 할 수 있듯이 센서도 여러 개를 조합할 수 있습니다.

센서를 제어할 수 있는 접근을 허용하지 않지만 다음의 두 가지 방법으로 센서의 상태를 얻을 수 있습니다.

1. b2Contact::IsTouching
1. b2ContactListener::BeginContact and EndContact
