# clean_architecture
만들면서 배우는 클린아키텍처 With Java
## 학습목표
- 계층형 아키텍처를 사용했을 때의 잠재적인 단점들을 파악할 수 있다.
- 아키텍처 경계를 강제하는 방법들을 적용할 수 있다.
- 잠재적인 지름길들이 소프트웨어 아키텍처에 어떻게 영향을 미칠 수 있는지 파악할 수 있다.
- 언제 어떤 스타일의 아키텍처를 사용할 것인지에 대해 논할 수 있다.
- 아키텍처에 따라 코드를 구성할 수 있다.
- 아키텍처의 각 요소들을 포함하는 다양한 종류의 테스트를 적용할 수 있다.

<details>
<summary>1장 계층형 아키텍처의 문제는 무엇일까?</summary>
<div markdown="1">

#### 웹  계층, 도메인 계층, 영속성 계층으로 구성된  전통적인 웹 애플리케이션 구조
[![2022-09-29-01-17-58.png](https://i.postimg.cc/wBp1HRMf/2022-09-29-01-17-58.png)](https://postimg.cc/PCSXQxmY)
- 위의 웹계층에서는 요청을 받아 도메인 혹은 비즈니스 계층에 있는 서비스로 요청을 보낸다. 서비스에서는 필요한 비즈니스 로직을 수행하고, 도메인 엔티티의 현재 상태를 조회하거나 변경하기 위해서 영속성 컴포넌트를 호출한다.
- 사실 계층형 아키텍처는 견고한 패턴이다. 계층을 잘 이해하고 구성한다면 웹 계층이나 영속성 계층에 독립적으로 도메인 로직을 작성할 수 있다.

#### 그렇다면 계층형 아키텍처의 문제점은 무엇일까?
- 코드에 나쁜 습관들이 스며들기 쉽게 만들고 시간이 지날수록 S/W를 점점 더 변경하기 어렵게 만드는 수많은 허점들을 노출한다.

계층형 아키텍처는 데이터베이스 주도 설계를 유도
----------------
- 정의에 따르면 전통적인 아키텍처의 토대는 DB이다.
- 웹 계층은 도메인 계층에 의존하고, 도메인 계층은 영속성 계층에 의존하기 때문에 자연스레 DB에 의존하게 된다.

### 그렇다면 우리는 왜 '도메인 로직'이 아닌 '데이터베이스'를 토대로 아키텍처를 만드는 걸까?
- 우리는 보통 DB구조를 먼저 생각하고, 이를 토대로 도메인 로직을 구현한다. 이는 전통적인 계층형 아키텍처에서는 합리적인 방법이다. 하지만 비즈니스 관점에서는 전혀 맞지 않다.
- 다른 무엇보다도 도메인 로직을 먼저 만들어야 한다. 그래야만 우리가 로직을 제대로 이해했는지 확인할 수 있다.
- 또 우리가 DB 중심적으로 아키텍처가 만들어지는 이유 중 큰 원인은 ORM 프레임워크를 사용하기 때문이다. 왜냐하면 ORM 프레임워크를 계층형 아키텍처와 결합하면 비즈니스 규칙을 영속성 관점과 섞고 싶은 유혹을 쉽게 받기 때문이다.
  [![2022-09-29-01-31-24.png](https://i.postimg.cc/BZNsnYQf/2022-09-29-01-31-24.png)](https://postimg.cc/JtySTQkp)
- 위 그림과 같이 ORM에 의해 관리되는 엔티티들은 일반적으로 영속성 계층에 둔다. 계층은 아래 방향으로만 접근 가능하기 때문에 도메인 계층에서는 이러한 엔티티에 접근 할 수 있다. 그리고 이러한 엔티티에 접근할 수 있다면 분명 사용되기 마련...!!!
- 하지만 이렇게 되면 영속성 계층과 도메인 계층 사이에 강한 결합이 생기됩니다.
- 영속성 코드가 사실상 도메인 코드에 녹아들어가서 둘 중 하나만 바꾸는 것이 어려워진다.. 이는 유연하고 선택의 폭을 넓혀준다던 계층형 아키텍처의 목표와 반대되는 상황이다.

테스트하기 어려워진다.
--------------
- 계층형 아키텍처를 사용할 때 일반적으로 나타나는 변화의 형태는 계층을 건너뛰는 것이다. 엔티티의 필드를 단 하나만 조작하면 되는 경우 웹 계층에서 바로 영속성 계층에 접근하면 도메인 계층을 건드릴 필요가 없지 않나..?
  [![2022-09-29-01-38-50.png](https://i.postimg.cc/x1VcKpHp/2022-09-29-01-38-50.png)](https://postimg.cc/GHzLRQXv)
### 위 그림에는 두 가지 문제점이 있다.
1. 단 하나의 필드를 조작하는 것에 불과하더라도 도메인 로직을 웹 계층에 구현하게 된다는 것. 앞으로 유스케이스가 확장된다면 더 많은 도메인 로직을 웹 계층에 추가해서 애플리케이션 전반에 걸쳐 책임이 섞이고 핵심 도메인 로직들이 퍼져나갈 확률이 높다.
2. 웹 계층 테스트에서 도메인 계층뿐 아니라 영속성 계층도 모킹(Mocking) 해야 한다는 것이다. 이렇게 되면 단위 테스트의 복잡도가 올라간다.(이건 나의 경험담...🥹)

동시 작업이 어려워진다.
---------------
**"지연되는 소프트웨어 프로젝트에 인력을 더하는 것은 개발을 늦출뿐이다."** - 브룩스
- 만약 애플리케이션에 새로운 유스케이스를 추가한다고 생각해보자. 개발자는 3명이 있다. 한명은 웹 계층에 필요한 기능을 추가할 수 있고, 다른 한 명은 도메인 계층에, 그리고 나머지개발자는 영속성 계층에 기능을 추가한다?? **계층형 아키텍처에서는 이렇게 작업을 할 수 없다.** 모든 것이 영속성 계층 위에 만들어지기 때문에 영속성 계층을 먼저 개발해야 하고, 그 다음에 도메인, 마지막에 웹 계층을 만들어야한다... 그렇기 때문에 동시작업은 불가능하다.
</div>
</details>

<details>
<summary>2장 의존성 역전하기</summary>
<div markdown="1">

- 이번장에서는 단일 책임 원칙과 의존성 역전 원칙에 대해 이야기하는 것으로 시작한다. 두 원칙은 SOLID 원칙에서 'S'와 'D'를 담당하고 있다.
----------
단일 책임 원칙
--------------
- 일반적인 해석은 다음과 같다.
  **하나의 컴포넌트는 오로지 한 가지 일만 해야하고, 그것을 올바르게 수행해야 한다**
  맞는 말이지만 실제 의도는 아니다.

- 실제 정의는 다음과 같다.
  **컴포넌트를 변경하는 이유는 오직 하나뿐이어야 한다.**
  즉, 컴포넌트를 변경할 이유가 오로지 한 가지라면 컴포넌트는 딱 한 가지 일만 하게 된다. 하지만 이보다 더 중요한 것은 변경할 이유가 오직 한 가지라는 그 자체다.

[![2022-09-29-22-32-50.png](https://i.postimg.cc/L81zXDfz/2022-09-29-22-32-50.png)](https://postimg.cc/xX9kpK6C)
- 위 그림에서 컴포넌트 A는 다른 여러 컴포넌트에 의존하는(직접적이든 전이된 것이든) 반면 컴포넌트 E는 의존하는 것이 전혀 없다.
- 컴포넌트 E를 변경할 유일한 이유는 새로운 요구사항에 의해 E의 기능을 바꿔야 할 때 뿐이다. 반면 컴포넌트 A의 경우는 모든 커포넌트에 의존하고 있기 때문에 다른 어떤 컴포넌트가 바뀌든지 같이 바뀌어야함... (오늘 버그를 수정하는 API Service Layer 진짜 딱 이 모양 이꼴이어서 진짜 너무 힘들었다...🥹)

→ 많은 콛느느 단일 책임 원칙을 위반하기 때문에 시간이 갈수록 변경하기가 더 어려워지고 그로 인해 변경 비용도 증가한다.(진짜 뼈저리게 느끼고 있음...하..)
-------------

의존성 역전 원칙
-----------------
- 계층형 아키텍처에서 계층 간 의존성은 항상 다음 계층인 아래 방향을 가리킨다. 단일 책임 원칙을 고수준에서 적용할 때 상위 게층들이 하위 계층들에 비해 변경할 이유가 더 많다는 것을 알 수 있다.
  그렇다면 영속성 계층에 대한 도메인 계층의 의존성 때문에 영속성 계층을 변경할 때마다 잠재거으로 도메인 계층도 변경해야 할까??? 놉!!! 그래서는 안된다. **도메인 코드**는 애플리케이션에서 가장 중요한 코드이기 때문에 변경이 되면 안된다. 그렇다면 어떻게 How?? 이 의존성을 제거할 수 있을까?

### 의존성 역전 원칙
- 코드상의 어떤 의존성이든 그 방향을 바 꿀 수(역전시킬 수) 있다.

[![2022-09-29-23-51-59.png](https://i.postimg.cc/CxVhtR9s/2022-09-29-23-51-59.png)](https://postimg.cc/bs3fSwyZ)
- 도메인 계층에 인터페이스를 도입함으로써 의존성을 역전시킬 수 있고, 그 덕분에 영속성 계층이 도메인 계층에 의존하게 된다.
- ----------

 클린 아키텍처
-----------------
- 로버트 C.마틴은 비즈니스 규칙은 프레임워크, 데이터베이스, UI 기술, 그 밖의 외부 애플리케이션이나 인터페이스로부터 독립적일 수 있다고 이야기했는데, 이는 **도메인 코드가 바깥으로 향하는 어떤 의존성도 없어야 함을 의미한다.**
  [![2022-09-30-00-00-45.png](https://i.postimg.cc/JhKRsmPw/2022-09-30-00-00-45.png)](https://postimg.cc/R33kYkkG)
- 클린 아키텍처의 추상적 모습이다.
- 도메인 코드에서 어떤 영속성 프레임워크나 UI 프레임워크가 사용되는지 알 수 없기 때문에 특정 프레임워크에 특화된 코드를 가질 수 없고 비즈니스 규칙에 집주할 수 있다.

### 클린 아키텍처의 대가
- 도메인 계층이 영속성이나 UI 같은 외부 계층과 철저하게 분리돼야 하므로 애플리케이션의 엔티티에 대한 모델을 각 계층에서 유지보수 해야한다는 단점이 있다.

**예를들어보자**
- 영속성 계층에서 ORM( 객체-관계 매핑 ) 프레임워크를 사용한다고 한다고 치자. 일반적으로 ORM 프레임워크는 데이터베이스 구조 및 객체 필드와 데이터베이스 칼럼의 매핑을 서술한 메타데이터를 담고 있는 엔티티 클래스를 필요로 한다. 하지만 도메인 계층은 영속성 계층을 모르기 때문에 도메인 계층에서 사용한 엔티티 클래스를 영속성계층에서 함께 사용할 수 없고 두 계층에서 각각 엔티티를 만들어야한다. **즉, 도메인 계층과 영속성 계층이 데이터를 주고받을 때 두엔티티를 서로 변환해야 한다는 뜻이다.**
- 하지만 이 부분은 바람직하다고 한다. 왜냐!! 이것이 바로 도메인 코드를 프레임워크에 특화된 문제로 부터 해방 시키고자 했던 결합이 제거된 상태이기 때문이다...!

</div>
</details>
