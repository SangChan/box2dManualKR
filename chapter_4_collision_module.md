
# Chapter 4 Collision Module

# 4.1 About

충돌 모듈에는 도형과 기능에 대한 작동부를 가지고 있습니다. 또한 대형 시스템의 가속 충돌 처리에 대한 동적 트리 및 광범위 상을 포함하고 있습니다.
본 모듈은 역학적인 환경 이외에서도 효율적으로 사용할 수 있도록 설계되어 있습니다. 예를 들어, 동적 트리 같은 것은 게임의 물리엔진 외의 부분에도 적용해서 사용할 수 있습니다.
어쨌든, Box2D는 강체물리엔진이므로 본 모듈을 다른 종류의 앱에서 사용하기에는 뭔가 한계가 있는듯한 느낌이 들 것입니다. 마찬가지로 본 문서 및 API를 우아하게 만드는데에는 많은 노력을 쓰지 않을 예정입니다.

# 4.2 Shapes

도형은 충돌 좌표를 묘사하며, 독립적으로 물리법칙을 시뮬레이션 하는데 사용됩니다. 최소한, 이것을 어떻게 생성하고 강체에 붙이는지를 이해해야만 합니다.
Box2D 는 도형을 b2Shape 클래스를 통해 구현하며, 다음과 같은 기능을 가지고 있습니다.

* 도형의 좌표가 겹치지 않는지 테스트
* 도형에 대해 raycast (a에서 b 방향으로 ray를 쏴서 충돌하는 객체를 찾는 것) 실행 
* 도형의 AABB(Axis Align Bounding Box : 충돌체크용 충돌경계상자) 계산
* 도형의 질량 속성 계산

덧붙여, 각각의 도형은 타입과 반지름 값을 가지고 있습니다. 다각형도 반지름 값을 가질 수 있습니다.
도형은 강체에 대해 모르며, 역학적으로 단절되어 있음을 명심하시기 바랍니다. 도형은 최적화를 위해 최소화 하여 저장됩니다. 도형은 쉽게 이동되지 않으며, 임의로 움직일 정점을 지정해야만 합니다. 하지만 도형이 fixture 를 통해 강체에 적용되면, 강체를 움직이는데 사용될 수 있습니다. 요약하자면 다음과 같습니다.

* 도형이 강체에 부착되지 않으면, world 기준의 좌표계로 도형의 각 정점이 표시됩니다.
* 도형이 강체에 부착되고 나면, 지역 좌표계로 변환되어 도형의 각 정점이 표시됩니다.

# 4.3 Circle Shapes

원은 위치와 반지름을 가집니다.
속이 꽉 찬 원만 만들어지며, circle shape 를 통해 속이 빈 원을 만드는 것은 불가능 합니다.

	b2CircleShape circle;
	circle.m_p.Set(2.0f, 3.0f);
	circle.m_radius = 0.5f;

# 4.4 Polygon Shapes

다각형 도형은 속이 차있으며 요철이 있는 다각형을 뜻합니다. 다각형은 두 개의 점이 연결될 때 도형의 다른 변을 건너지 못하도록 합니다. 다각형은 속이 차있으며, 테두리만 존재하는 도형이 아닙니다. 3개 이상의 점이 있어야 합니다.

다각형의 점들은 CCW(역시계 방향) 으로 지정합니다. CCW는 z축이 면을 향하도록 한 상태의 오른손 좌표계을 따르므로 주의가 필요합니다. 좌표계 방향에 따라 사용자의 화면에 시계방향으로 표시될 수 있습니다.

> ** (원문) ** Polygons vertices are stored with a counter clockwise winding (CCW). We must be careful because the notion of CCW is with respect to a right-handed coordinate system with the z-axis pointing out of the plane. This might turn out to be clockwise on your screen, depending on your coordinate system conventions.

다각형 객체의 멤버변수는 public 이지만, 생성하고 초기화한 이후에 사용해야만 합니다. 초기화 함수를 통해 법선 벡터를 생성하고 유효성 검증을 합니다.
정점 배열을 넘겨서 생성하는 것도 가능합니다. 이때, 배열의 최대 갯수는 b2_maxPolygonVertices 에 의해 정해지는데 초기값은 8입니다. 다각형의 요철을 묘사하기에 아주 좋습니다. b2_maxPolygonVertices의 값을 증가시키면, 요철을 계산하는 것이 느려질 수 있습니다. 

b2PolygonShape::Set 함수가 자동으로 요철을 계산하고 선이 그려질 순서를 설정합니다. 정점의 수가 적을수록 빠르게 계산이 가능합니다. 해당 기능은 사용자가 제공한 정점을 제거하거나 순서를 다시 짜기도 합니다.

	// This defines a triangle in CCW order.
	b2Vec2 vertices[3];
	vertices[0].Set(0.0f, 0.0f);
	vertices[1].Set(1.0f, 0.0f);
	vertices[2].Set(0.0f, 1.0f);
	int32 count = 3;

	b2PolygonShape polygon;
	polygon.Set(vertices, count);
	
다각형 도형은 사각형을 생성하는 쉬운 함수를 가지고 있습니다.

	void SetAsBox(float32 hx, float32 hy);
	void SetAsBox(float32 hx, float32 hy, const b2Vec2& center, float32 angle);

다각형은 b2Shape 에서 상속 받은 반지름값을 가지고 있습니다. 반지름값은 다각형의 주위에 표면을 만듭니다. 표면은 다각형을 분리시키는 용도로 사용됩니다. 이를 통해 핵심 다각형에 대한 연쇄충돌이 가능 합니다.
	
다각형의 표면은 다각형 간의 분리로 인해 터널링(장벽투과)이 발생하는 것을 방지하는데 도움을 줍니다. 이로인해 도형간에 약간의 틈이 생깁니다. 이 틈을 막기 위해 그래픽 표현을 다각형보다 조금 더 크게 됩니다..
 

# 4.5 Edge Shapes

모서리 형은 선을 뜻합니다. 이는 자유로운 형태의 정적 환경을 만들고자 제공되는 기능으로, 원이나 다각형의 도형과 충돌하는 것은 가능하지만 같은 형태의 모서리 형과의 충돌은 되지 않는 한계를 가지고 있습니다. 왜냐하면 Box2D 의 충돌 알고리즘에서 둘 중 하나의 물체는 부피를 가져야 하는데 모서리 형은 부피가 없기 때문입니다. 그래서 둘 다 부피가 없는 모서미 대 모서리 형의 충돌은 불가합니다.

	// This an edge shape.
	b2Vec2 v1(0.0f, 0.0f);
	b2Vec2 v2(1.0f, 0.0f);

	b2EdgeShape edge;
	edge.Set(v1, v2);

대체로 게임은 모서리 형의 끝과 끝을 잇는 식으로 구성된 경우가 많습니다. 모서리와 모서리가 이어진 부분에서 예기치 않은 결과가 나올 수 있는데, 하단 그림에서 보듯이 내부 정점에서 사각형 충돌이 발생합니다. 이 알 수 없는 충돌은 내부 정점에서 내부 충돌 법선 벡터가 발생하여 생기는 문제입니다.

만약 edge1 이 존재하지 않았다면 당연히 발생해야 하는 일인지도 모르지만, edge1이 존재함으로 이 충돌은 버그처럼 보입니다. 하지만 보통 Box2D 에서 두 개의 물체가 충돌한다면, 각각은 별개로 취급됩니다.
다행히, 유령 정점을 만들어 이러한 충돌을 제거할 수 있도록 하고 있습니다.  

 
	// This is an edge shape with ghost vertices.
	b2Vec2 v0(1.7f, 0.0f);
	b2Vec2 v1(1.0f, 0.25f);
	b2Vec2 v2(0.0f, 0.0f);
	b2Vec2 v3(-1.7f, 0.4f);

	b2EdgeShape edge;
	edge.Set(v1, v2);
	edge.m_hasVertex0 = true;
	edge.m_hasVertex3 = true;
	edge.m_vertex0 = v0;
	edge.m_vertex3 = v3;
	
이러한 모서리 형을 서로 잇는 작업은 매우 지루하고 낭비적이므로 체인 형을 제공합니다.


# 4.6 Chain Shapes
정적인 게임내에 수많은 모서리를 연결하기 위한 효율적인 방법으로 체인 형을 제공합니다. 체인 형은 알 수 없는 충돌을 자동적으로 제거하고, 쌍방 충돌을 제공합니다.
 
	// This a chain shape with isolated vertices
	b2Vec2 vs[4];
	vs[0].Set(1.7f, 0.0f);
	vs[1].Set(1.0f, 0.25f);
	vs[2].Set(0.0f, 0.0f);
	vs[3].Set(-1.7f, 0.4f);

	b2ChainShape chain;
	chain.CreateChain(vs, 4);
	
스크롤이 되는 게임을 만든다면 여러개의 체인을 연결할 필요가 있습니다. b2EdgeShape 에서 한 것 처럼 유령 정점을 만들어 여러 개의 체인을 연결 할 수 있습니다.

	// Install ghost vertices
	chain.SetPrevVertex(b2Vec2(3.0f, 1.0f));
	chain.SetNextVertex(b2Vec2(-2.0f, 0.0f));
	
또 다음과 같이 해서 루프 형태를 만들 수 도 있습니다.

	// Create a loop. The first and last vertices are connected.
	b2ChainShape chain;
	chain.CreateLoop(vs, 4);

자체적인 교차 기능은 지원되지 않습니다. (어쩌면, 될 수도 있고 아닐 수도 있습니다.) 알 수 없는 충돌을 방지하는 코드는 자체적인 교차 기능이 없다는 전제하에 만들어 졌습니다. 그리고 너무 가까운 정점은 문제를 일으 킬 수 있습니다. 각 모서리의 길이는 b2_linearSlop (5mm) 보다 길어야 합니다.

체인 안에 있는 모서리 들은 각각 자식 형태로 존재하며 인덱스를 통해 접근이 가능합니다. 강체에 체인 형이 연결된다면, 각 모서리는 광범위 충돌 트리를 통해 각자의 상자형태의 경계를 얻습니다.

	// Visit each child edge.
	for (int32 i = 0; i < chain.GetChildCount(); ++i)
	{
		b2EdgeShape edge;
		chain.GetChildEdge(&edge, i);
		…
	}
	
# 4.7 Unary Geometric Queries
한 개의 도형에 대해 여러 개의 형상검색을 실행할 수 있습니다.

# 4.8 Shape Point Test

해당 도형이 해당 위치에서 겹쳐지는지 여부를 알 수 있습니다. 도형의 이동에 대해 world 상의 좌표를 제시하면 됩니다.

	b2Transfrom transform;
	transform.SetIdentity();
	b2Vec2 point(5.0f, 2.0f);
	bool hit = shape->TestPoint(transform, point);
	
모서리와 체인 (루프 포함)형은 항상 false 값을 반환합니다.

# 4.9 Shape Ray Cast

광선을 추적하여 해당 도형의 첫 교차점 및 법선 벡터를 구할 수 있습니다. 광선이 도형 내부에서 시작된다면 hit 값에 false 가 반환됩니다. 광선 추적은 단일 모서리에 대한 값만을 확인하기 때문에 체인 형에 대한 값을 얻기 위해 child index 가 있어야 합니다.

	b2Transfrom transform;
	transform.SetIdentity();
	b2RayCastInput input;
	input.p1.Set(0.0f, 0.0f, 0.0f);
	input.p2.Set(1.0f, 0.0f, 0.0f);
	input.maxFraction = 1.0f;
	int32 childIndex = 0;
	b2RayCastOutput output;
	bool hit = shape->RayCast(&output, input, transform, childIndex);
	if (hit)
	{
		b2Vec2 hitPoint = input.p1 + output.fraction * (input.p2 – input.p1);
		…
	}
	
# 4.10 Binary Functions

충돌모듈은 두 개의 도형에 대한 쌍방함수를 포함하고 있습니다.

* 중첩
* 다중 접촉 
* 거리
* 충돌 시간

# 4.11 Overlap

다음 합수를 통해 두 개의 도형에 대한 중첩을 테스트 할 수 있습니다.

	b2Transform xfA = …, xfB = …;
	bool overlap = b2TestOverlap(shapeA, indexA, shapeB, indexB, xfA, xfB);

체인 형에 대해서는 자식의 숫자를 알 수 있는 index값이 제공되어야 합니다.

# 4.12 Contact Manifolds

Box2D 는 도형이 겹치는 접점을 계산하는 함수를 가지고 있습니다. 원 대 원 이나 원 대 다각형 일 때는 1개의 접점과 법선 벡터를 갖지만, 다각형 대 다각형의 경우 2개의 접점이 생깁니다. 각각의 접점은 동일한 법선 벡터를 가지므로 Box2D 는 해당 접점을 구조체로 묶습니다. contact solver 는 이를 통해 안정성을 높여 이점을 취합니다.

보통 다중 접점을 따로 계산할 필요는 없지만, 시뮬레이션 상에서 만들어진 결과를 선호할 수도 있습니다.
b2Manifold 구조체는 한 개의 법선벡터와 두 개의 접점을 가지고 있으며, 좌표계는 지역 좌표계를 사용합니다. contact solver 의 편의를 위해, 각각의 점점은 법선 벡터와 접선 (마찰) 충격량을 저장합니다.
b2Mainfold 구조체에 저장된 데이터는 내부 이용을 위해서 쓰이며, 필요한 경우 b2WorldManifold를 통해 world 좌표계로 작성된 접점과 법선 벡터를 사용하기 바랍니다. b2Manifold 와 도형 변환 값과 반지름을 제공해야 합니다.

	b2WorldManifold worldManifold;
	worldManifold.Initialize(&manifold, transformA, shapeA.m_radius,transformB, shapeB.m_radius);
	for (int32 i = 0; i < manifold.pointCount; ++i)
	{
		b2Vec2 point = worldManifold.points[i];
		…
	}
	
> world manifold 에서는 기존 manifold 의 갯수를 필요로 합니다.
>

시뮬레이션 진행 중 도형이 움직이거나 하여 해당 객체가 바뀔 수 있습니다. 접점은 추가되거나 제거될 수 있으니 b2GetPointStates를 이용해 검색해보시기 바랍니다.

	b2PointState state1[2], state2[2];
	b2GetPointStates(state1, state2, &manifold1, &manifold2);
	if (state1[0] == b2_removeState)
	{
		// process event
	}
	
# 4.13 Distance

b2Distance 를 통해 2개의 도형의 거리를 구할 수 있습니다. b2DistanceProxy 내부에서 계산되기 위해 두 도형이 모두 필요합니다. 반복적인 요청을 위해 캐쉬를 가지고 있으며 자세한 사항은 b2Distance.h f를 참조하시기 바랍니다.

# 4.14 Time of Impact
두 개의 물체가 빠르게 움직인다면, 특정 시간 단계에서 서로가 통과해버리는 터널링이 발생 할 수 있습니다.
 
b2TimeOfImpact 기능은 두 개의 움직이는 물체가 충돌할 시간을 결정하는데 사용됩니다. 이를 TOI(충돌 시간)이라고 하며, b2TimeOfImpact는 터널링을 방지하는 것이 존재의 이유 입니다. 특히, 정적인 기하학을 벗어나서 이동하는 물체 간의 터널링 현상을 방지하기 위해 설계되었습니다.
이 기능은 두 도형의 회전(충분히 크게 회전할 경우)과 이동에 대해 충돌을 놓칠 수도 있습니다. 그러나 중첩이 안되었던 시간 과 이동 중 발생하는 충돌에 대해서는 보고를 할 수 있습니다. TOI 기능은 각각의 분리된 축을 설정하고 해당 축을 넘지 못하도록 하는 것을 보장할 수 있습니다. 최종 위치에 대해 충돌을 놓칠 수 있지만, 속도면에서 터널링 방지에 최적입니다.

회전각에 대해서는 제한을 넣기가 힘듭니다. 작은 회전에 의한 충돌을 놓치는 경우도 있습니다. 보통 게임에 영향을 끼치는 경우가 없어서 경시하는 경향이 있습니다.
해당 기능에는 b2DistanceProxy 를 통해 변환된 두 도형과 b2Sweep 구조체 두개가 필요합니다. sweep 구조체는 도형의 최초와 최종 변형을 정의합니다. 도형 선언에 대해 회전을 못하도록 할 수 있으며, 이 경우 TOI 는 어떠한 충돌도 놓치지 않습니다.

# 4.15 Dynamic Tree
b2DynamicTree 클래스는 Box2Dd에서 많은 양의 도형을 효율적으로 관리하기 위해 사용됩니다. 해당 클래스는 도형에 대해서는 모르며, 대신 AABB(충돌체크용 충돌경계상자)를 위한 유저데이터 포인터를 운영합니다.

동적 트리는 AABB 트리의 계층 구조이며, 각각의 내부 노드들은 두 개의 자식을 가집니다. 리프노드(후계노드를 가지지 는 끝노드)는 AABB의 싱글 유저입니다. 잘못된 입력값이 들어오더라도 트리는 밸런스를 유지하기 위해 회전을 할 수 있습니다.
트리 구조는 효율적인 광선 추적 및 지역 검색을 가능하게 합니다. 예를 들어, 화면에 몇 백개의 도형이 있을 수 있습니다. 이때, 무작위로 각각의 도형에 대해 광선추적을 하는 것은 도형이 분산되어 있을 경우 매우 비효율적일 수 있습니다. 대신 동적 트리를 할당 하고 해당 트리에 대해서만 광선 추적을 하는 겁니다. 이를 통해 많은 도형에 대한 광선 추적을 피할 수 있습니다.

지역 검색을 통해 검색에 사용할 AABB 와 겹치는 노드상의 AABB 를 찾는 것은 무작위로 접근하는 것보다 빠릅니다. 왜냐하면 필요한 대상에만 집중할 수 있기 때문입니다.

일반적으로 동적트리를 직접 사용하지는 않을 것입니다. b2World 클래스의 광선 추적 및 지역 검색을 통해서 사용할 것이며, 혹시 해당 기능을 가져다 쓰고자 한다면 Box2D 에서 어떻게 사용하는지 참조하시기 바랍니다.


# 4.16 Broad-phase

충돌 계산은 물리적으로 협소한 단계와 광범위 단계로 나눌 수 있습니다. 협소한 경우, 여러 쌍의 도형에 대한 접점을 계산하는 것 입니다. N개의 도형이 있다고 가정합니다. 무작위로 협소한 단계로 계산한다면 N*N/2 쌍에 대해 계산해야 합니다. 
b2BroadPhase 클래스는 동적 트리를 통해 여러 쌍을 관리해 과부하를 줄여줍니다. 협소한 단계에 의한 호출을 많이 줄일 수 있습니다.  

보통 광범위 단계에 대해 직접 알 필요는 없습니다. 대신, Box2D는 광범위 단계에 대해 내부적으로 관리합니다. b2BroadPhase는 Box2D의 시뮬레이션 루프를 염두에 두고 설계되었으므로 다른 사용 사례에는 적합치 않습니다.
