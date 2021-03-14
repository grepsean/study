# Postmortem Culture: Learning from Failure
"The cost of failure is education." Devin Carraway

SRE 측면에서 large-scale, complex, distributed systems 상에서 작업한다. 우리는 끊임없이 우리의 서비스에 새로운 피처와 시스템을 추가하여 개선한다. 장애와 중단은 서비스의 스케일과 변화의 속도에 따라서 피할 수 없는 것이다. 장애가 일어났을때 이슈를 해결하고 서비스를 정상화한다. 이런 장애로부터 배운것이 형식화된 절차로 적용하지 않는다면, 장애는 무한정 반복될 것이다. 해결하지 않은 상태로 두면, 장애의 복잡도는 계속해서 증가하여 시스템을 압도되고 결국 사용자에게 영향을 미칠 것이다. 그러므로 postmorterm은 SRE에 필수적인 툴이다. 

Postmortem의 컨셉은 technology industry에서 잘 알려져 있다. postmortem은 장애와 그 영향, 완화 해결하기위한 조치, 근본적인 원인과 장애 재발 방지를 위한 후속조치에 대한 기록이다. 이 챕터에서는 postmorterm을 실행할 시기를 결정하기 위한 기준, postmorterm에 대한 몇가지 모범 사례와, 수년간 얻은 경험을 통해서 postmorterm 문화를 어떻게 키울지 조언한다. 

# Google’s Postmortem Philosophy
postmortem 작성의 주요한 목표는 장애를 문서화하는 것을 보장하는 것이다. 관련된 근본적인 원인을 잘 이해하고, 특히 재발 가능성과 영향을 줄이기 위해 효율적인 예방 조치를 취하는 것이다. 근본 원인 분석 테크닉의 자세한 조사는 이 챕터의 scope은 아니다. ([여기](https://asq.org/quality-progress/articles/root-cause-analysis-for-beginners?id=0228b91456514ba490c89979b577abb4)를 참조하세요) 하지만 문서, 모범 사례, 도구는 많이 있다. 팀의 근본 원인 분석을 위해 여러 기술을 사용하는데 가장 적절한 기술을 선택하자. Postmortems는 어떤 원치 않은 중요한 이벤트 후에 진행되는것으로 예상된다. Postmortems을 작성하는 것은 punishment가 아니라 회사 전체적으로 배움의 기회이다. postmortem 절차는 시간이나 노력과 같은 비용으로 이어지므로, 작성 시점을 신중하게 선택해야한다. 팀은 내부적인 유연성을 가져야하지만, 일반적인 postmorterm의 시작은 아래를 포함한다.
* user에게 보여지는 downtime이나 특정 threshold 이하의 성능 저하
* 데이터 유실
* On-call 엔지니어의 개입 (릴리즈 롤백, 트래픽 rerouting 등)
* 어떤 threshold를 넘어설때
* monitoring failure (보통 매뉴얼 장애 복구를 포함)

장애 발생전에 postmortem 기준을 정하는 것은 중요하다. 그래야 모든 사람들이 postmortem이 필수적이라는 것을 알 수 있다. 게다가 이러한 정해진 Trigger에 이외에도, 이해 관계자자들이 postmorterm을 요청할 수도 있다.

Blameless postmortems는 SRE 문화의 원칙이다. postmortem가 진정 blameless하려면, 어떤 개인이나 팀의 잘못을 드러내기보다는 장애의 원인을 찾는데 포커스를 맞춰야한다. blameless한 postmortem이라는 것은 장애와 관련된 모든 사람이 좋은 의도로 가지고 있는 정보를 이용해서 제대로 일했다고 가정한다. 
만약 지적하고 개인이나 팀의 "잘못"을 따지는 것은 결국 punishment가 두려워서 이슈를 들어내지 않게 할 것이다.

Blameless culture는 헬스케어와 항공처럼 실수 하나가 아주 치명적인 산업에서부터 시작했다. 이러한 산업에서는 모든 "mistake"가 시스템을 더 강하게 만드는 기회로 보는 환경을 육성한다. 어떤 개인이나 팀의 잘못을 조사해서 비난하는 것이 아닌 postmortems으로 변화했을때, 효과적인 예방 계획이 만들어진다. 사람을 바꿀 수 없지만, 사람들이 복잡한 시스템을 설계하고 운영할때 제대로 선택할 수 있도록 시스템과 절차를 바꿀 수 있다.

중단이 일어났을때, postmortem이 잊혀지도록 형식적으로 작성되지 않는다. 대신에 postmortem은 weakness 보완하거나 더 resliient한 시스템으로 만들 수 기회로써 엔지니어가 보게된다. blameless postmortem는 단순히 지적하는게 아니라, 어느 부분에서 어떻게 서비스를 개선시킬지 알려야한다. 

### Pointing fingers
"우리는 복잡한 백엔드 시스템 전체를 다시 작성해야한다!. 이전의 3분기동안 매주를 문제가 발생했고, 몇몇의 fixing하는데 지쳐있다. 진짜 한번 더 알림을 받는다면 직접 재작성 하겠다..."

### Blameless
"전체 백엔드 시스템을 재작성하는 액션 아이템 이러한 계속 발생하는 annoying 알람을 예방할 수 있다. 그리고 이 버전의 운영 매뉴얼은 굉장히 길고 익숙해지는데 정말 어렵다. 다음 on-call 엔지니어가 고마워할 것이라는 걸 확신한다."


## Best Practice: Avoid Blame and Keep It Constructive
Blameless postmortems 작성은 여렵다. 왜냐면 postmortem format은 명확히 장애를 일으키는 액션을 구분하기 때문이다. postmortems에서 blame을 제거하면 사람들에게 두려움없이 이슈를 escalate할 수 있다는 자신감을 준다. 그리고 자주 production postmorterm을 작성하는 것에도 오명을 씌워선 안된다. 비난하는 분위기는 장애와 이슈를 숨기는 문화를 만들어내고 결국 더큰 조직의 위험을 초래한다. 


# Collaborate and Share Knowledge
협업은 가치가 있고, postmortem 절차는 예외가 없다. postmortem은 모든 단계에 대한 협업과 지식 공유를 포함한다. 

우리의 postmortem 문서는 Google docs를 사용하며 in-house template을 활용한다. (see [Example Postmortem](https://sre.google/sre-book/postmortem-culture/)) 특정 툴의 사용과 상관없이 아래의 피쳐를 확인해보자.

* Real-time collaboration : 데이터와 아이디어를 빠를게 취합할 수 있게해준다. postmortem을 작성하는 초기에 필수적이다. 
* An open commenting/annotation system : crowdsourcing solutions을 쉽게 만들고 커버리지를 개선한다.
* Email notifications : 공동 작업자에게 지시하거나 다른사람들에게 필요한 input(정보)을 제공하는데 사용된다. 

postmortem을 작성하는 것은 형식적인 리뷰와 발행을 포함한다. 실제로, 팀은 내부적으로 첫번째 postmorterm draft을 공유하고 완성도를 높이기위해 시니어 엔지니어 그룹이 draft에 접근할 수 있도록 요청하다. 리뷰 조건을 아래를 포함할 수 있다.
* 장애 데이터가 수집되었는가?
* 영향 평가는 완료되었는가?
* 근본 원인을 충분히 확인하였는가?
* 액션 플랜은 적절하고, 적절한 우선순위로 버그 수정이 되는 액션 플랜인가?
* 관련자들에게 결과가 공유되었는가?

initial review가 완료되면, postmortem은 더 광역적으로 공유된다. 보통 더 많은 엔지니어링 팀이나 내부 메일링 리스트로 공유된다. 우리의 목표는 postmortems을 공유해서 가능한 넓은 청자에게 장애로 인한 지식과 경험의 공유하는 것이다. 구글은 end-user를 식별하는 어떤 정보에 접근하기 위해서는 엄격한 규칙을 가지고 있다. postmorterm 같은 내부적인 문서일지라도 이러한 정보를 절대 포함하지 않는다.


## Best Practice: No Postmortem Left Unreviewed
unreviewed postmortem는 존재하지 않을 수 있다. 각 draft 리뷰가 완료되는 것을 보장하기 위해서, postmortems을 위한 일반적인 리뷰 세션을 권장한다. 이런 미팅에서, 아이디어를 찾고 상태를 종료시키기 위해서 진행중인 discussion과 코멘트는 중단하는것이 중요하다. 

관련된 사람들이 이 문서와 액션 아이템에 만족을 한다면, postmortem은 팀이나 조직의 이전 장애 보관소에 추가된다. 이렇게 투명하게 공유해서 다른 사람들이 postmorterm으로부터 쉽게 찾고 배울 수 있도록 한다. 


# Introducing a Postmortem Culture
postmortem 문화를 조직에 도입하는 것은 말처럼 쉽지는 않다. 계속적으로 구축하고 강화해나가려는 노력이 필요하다.







