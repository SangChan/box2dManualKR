# Chapter 11 Loose Ends

# 11.1 User Data

b2Fixture, b2Body, j2Joint 클래스는 보이드 포인터 형으로 된 user data를 추가하는 것이 가능합니다. Box2D 데이터 구조체를 검사하거나 게임 엔진상의 객체와 얼마나 연관되어 있는지 관여하고 있는지를 밝히고자 할 때 유용합니다.

예를 들어, 일반적으로 이 기능은 강체를 사용하고 있는 actor의 포인터를 저장하는데 사용됩니다. 이는 선형 참조가 되므로 actor 가 있다면 이를 통해 강체를 얻을 수 있습니다. 반대로 강체가 있다면 이를 통해 actor를 얻는 것도 가능합니다.

	GameActor* actor = GameCreateActor();
	b2BodyDef bodyDef;
	bodyDef.userData = actor;
	actor->body = box2Dworld->CreateBody(&bodyDef);
	
user data가 반드시 필요한 곳은 다음과 같습니다:

* 충돌결과에 따라 actor에 데미지를 줘야함.
* 캐릭터가 축따라 정렬된 박스(AAB) 내부에 있을 때, 정해진 이벤트가 일어나야 함.
* Box2D가 joint의 소멸을 알려주고자 게임 구조체에 접근하려 함.

user data는 부가적인 정보이므로 무엇이든 넣을 수 있다는 것을 명심하십시요. 하지만 일관성을 유지해야 합니다. 여기서 말하는 일관성이란 강체에 actor의 포인터값을 저장하고자 한다면 모든 강체에 그렇게 해야 한다는 뜻입니다.

한 개의 강체에는 actor 포인터를 저장하고 다른 강체에는 다른 종류의 포인터를 저장하지 마십시요. 타입캐스팅을 하는 순간 프로그램에 오류가 발생할 수 있습니다.

user data에는 기본적으로 NULL값이 주어져 있습니다.

게임의 특정 정보, 그러니까 재료의 종류라던가 반복되는 효과나 반복되는 사운드 같은 것을 저장할 수 있는 데이터 구조체를 따로 정의하는 것도 고려해볼 수 있겠습니다.

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

Box2D 개체를 소멸한다면, 소멸될 객체에 대한 모든 참조를 없애는 것은 온전히 사용자의 몫입니다. 해당 개체에 대해 단 한 개의 참조를 가진다면 간단한 문제입니다. 참조를 여러 번 한다면, 원시포인터를 감싼 관리용 클래스를 구현하는 것을 고려해야 할 것 입니다.

Box2D를 쓸 때, 강체와 shape와 joint를 수도 없이 생성하고 소멸해야 할 것 입니다. 이런 개체를 관리하는 것은 Box2D에서 알아서 해줍니다. 강체를 소멸하면 첨부되어 있는 shape와 joint는 자동적으로 소멸됩니다. 이를 암시적 파괴라고 합니다.

강체를 소멸하면, 이에 첨부된 모든 shape, joint, contact도 소멸됩니다. 이를 암시적 파괴라고 합니다. 이런 joint나 contact에 연결된 강체는 깨어나게 됩니다. 대체로 이 방식은 편리하지만, 다음의 중요한 사안을 명심해야 합니다:

> 주의 <br>
> 강체가 소멸될 때, 강체에 부착되어 있던 fixture 나 joint는 자동적으로 소멸됩니다. 이런 shape 및 joint에 대한 포인터를 가지고 있다면 null 값을 주어야 합니다. 그러지 않을 경우, 차후 해당 shape 이나 joint로 접근하고자 할 때 프로그램이 에러나는 것을 볼 수 있을 것 입니다.

joint 포인터에 null값을 주는 것을 돕기 위해서 world객체에 b2DestructionListener를 구현한 구현체를 Box2D에 제공하여 암시적 파괴가 일어날 때, world object가 해당 리스너를 부르도록 할 수 있습니다.

joint 나 fixture가 언제 명확하게 소멸되는지를 따로 알려주지는 않습니다. 이런 경우, 누가 주체적으로 해당 지점에서 객체를 깨끗하게 비워주어야 하는지는 명확합니다. b2DestructionListener의 구현체를 만들어 코드상 객체를 깨끗이 비울 수 있도록 하는 것이 좋겠습니다.
 
암시적 파괴는 많은 경우에 매우 편리합니다. 또한 프로그램을 조각조각으로 파괴해버릴 수도 있습니다. shape와 joint의 포인터를 코드 어딘가에 저장해놓길 바랍니다. 해당 포인터를 사용하던 강체가 소멸하면 포인터는 가르키는 곳에 아무것도 없는 신세가 되어버립니다. 그렇다고 joint를 사용하는 강체를 관리하기 위해 joint를 자주 생성하는 코드를 별 상관없이 사용한다면 상황은 점점 더 나빠집니다. 예를 들어, 테스트 케이스에서 화면상의 강체간의 상호작용이 있는 조작계를 위해 b2MouseJoint를 만들었습니다.

Box2D는 암시적 파괴가 일어났다는 것을 알리는 콜백을 제공합니다. 해당 프로그램에 문제가 생긴 포인터에 null값을 줄 기회를 줍니다. 해당 내용은 차후에 기술하겠습니다.

b2DestructionListener를 구현함으로써, b2World에서 shape과 joint가 관련된 강체가 소멸됨으로써 암시적 파괴가 일어났음을 알리는 기능을 사용하도록 합니다. 이를 통해 가르키는 곳에 아무 것도 없는 고아 포인터들을 방지하는 코드를 작성하는 도움이 될 것 입니다.

	class MyDestructionListener : public b2DestructionListener
	{
    	void SayGoodbye(b2Joint* joint)
    	{
        	// remove all references to joint.
    	}
	};

그리고 world가 초기화 될 때, 해당 리스너의 구현된 인스턴스를 world 객체에 등록해 둘 수 있습니다.

	myWorld->SetListener(myDestructionListener);

# 11.3 Pixels and Coordinate Systems

Box2D는 MKS(미터,킬로그램,초)단위를 사용하고 각도를 표현할 때는 라디안을 사용함을 얘기한 바 있습니다. 게임은 보통 픽셀로 표현되므로 미터로 표현하는데 애로사항이 꽃필 수도 있습니다. 테스트 케이스에서는 이를 관리하기 위해 화면상에 world를 가늠하기 위해 OpenGL의 뷰포트 변환을 사용하여 모든 게임이 미터 단위로 작동하도록 하였습니다.

	float lowerX = -25.0f, upperX = 25.0f, lowerY = -5.0f, upperY = 25.0f;
	gluOrtho2D(lowerX, upperX, lowerY, upperY);

만약 게임이 꼭 픽셀단위로 움직여야 한다면 픽셀단위를 미터단위로 바꾸어 Box2D에 해당 값을 넘겨줄 필요가 있습니다. 마찬가지로 Box2D에서 받은 미터단위의 데이터를 픽셀단위로 바꿀 필요도 있습니다. 이는 물리엔진 상의 물리연산에 대한 안정성을 높여줍니다.

합리적인 변환 계수를 만들어야 할 필요가 있습니다. 캐릭터의 사이즈에 비례해 제작하는 것을 권장합니다. 캐릭터의 크기가 75픽셀이라면 50픽셀 정도를 1미터 잡는다고 가정하겠습니다. 이런 경우, 다음과 같은 공식을 사용할 수 있습니다:

	xMeters = 0.02f * xPixels;
	yMeters = 0.02f * yPixels;

반대로는 다음과 같습니다:

	xPixels = 50.0f * xMeters;
	yPixels = 50.0f * yMeters;

게임 코드에서는 MKS를 사용하고, 화면에 뿌릴때만 픽셀 단위로 바꾸는 것을 추천합니다. 이는 게임로직을 단순하게 할 수 있으며, 그리는 부분에서만 변환하는 코드가 들어가므로 에러처리와 코드관리에 이점이 있습니다.

변환 계수를 사용한다면, 전역으로 배치하여 충돌이 나지 않도록 하는게 좋습니다. 안정성을 높이기 위해 조정을 해야 할 수도 있습니다.