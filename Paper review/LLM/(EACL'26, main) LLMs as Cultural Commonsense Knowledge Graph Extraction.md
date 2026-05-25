- 한 줄 요약: LLM을 "문화적 아카이브"로 간주하고, 반복적 프롬프트 기반 프레임워크를 통해 문화별 if-then 추론 체인을 추출하여 Cultural Commonsense Knowledge Graph(CCKG)를 구축하는 방법을 제안
- Ref: [paper](https://arxiv.org/pdf/2601.17971)

# Abstract

[문제]

* LLM은 다양한 웹 규모 데이터에서 학습한 풍부한 문화적 지식을 인코딩하고 있어, 대규모로 문화적 상식(commonsense)을 모델링할 수 있는 전례 없는 기회를 제공한다.
* 하지만, 이 지식은 대부분 암묵적이고 비구조적인 상태로 남아 있어 해석가능성과 활용이 제한된다.

[해결]

* 본 논문은 LLM을 문화적 아카이브로 취급하여, 문화 특화 엔티티, 관계, 관습을 체계적으로 끌어내고 이를 다국어 다단계 추론 체인으로 구성하는 반복적 프롬프트 기반 프레임워크(CCKG)를 제시한다.
* 5개국에 대해 문화적 관련성, 정확성, 경로 일관성에 대한 인간 평가를 수행했으며, 비영어권 문화(중국어, 인도네시아어, 아랍어)라도 영어로 추출한 문화 지식 그래프가 더 우수하게 실현됨을 발견하였다.
	* LLM이 각 문화에 대한 지식을 언어별로 균등하게 학습하지 않았다는 것을 의미한다.
	* 현재 LLM의 불균등한 문화 인코딩을 문제를 나타낸다.

소규모 LLM에 CCKG를 증강하면 문화적 추론 및 스토리 생성 성능이 향상되며, 영어 체인에서 가장 큰 성능 향상이 관찰된다.

# 1. Introduction

문화와 상식 추론은 깊이 얽혀 있으며, 문화는 사람들이 일상적 상황, 사회적 관습, 인과 관계를 해석하는 방식을 형성한다.
- 문화는 공동체 내에서 해석과 행동을 안내하는 공유되고 학습된 가치, 규범, 관행을 포괄한다.
- 한 공동체에서 자명해 보이는 것이 다른 공동체에서는 낯설거나 오해를 불러일으킬 수 있다.

기존에는 상식을 문화 중립적(culture-neutral)로 다뤘으나, 최근에는 문화적 측면(dimension)을 인식하고 모델링하기 시작했다.
- 공동체 전반에 걸쳐 발생하는 언어적(linguistic), 인지적(congnitive) 다양성을 강조한다.

언어 모델이 구조화된 문화 표현을 갖추면 맥락에 따라 더 적절하게 추론할 수 있다. (특히 문화적인 관점에서 불균등한 온라인 데이터 공간에서)

ATOMIC 같은 초기 연구는 일상적 사건 간 추론적 지식 모델링의 가치를 보여주었지만, 이러한 자원들은 대부분 서구 중심적이며 문화 간 일반화가 부족하다.
- "X가 파티에 간다 → X는 즐거움을 느낀다" 같은 사건 간 추론적 관계
- 즉, 단어 수준이 아니라 일상적 상황/이벤트 수준에서 "이 일이 일어나면 그 다음엔 어떤 일이 일어나는가"를 모델링한 것

저자들은 LLM이 문화적 아카이브로서 역할을 얼마나 잘 하는가를 연구했다.
1. LLM이 실제 문화적 관련성에 부합하는 지식을 어느 정도 인코딩하는가?
2. LLM에서 추출할 때 어떤 언어가 문화 지식을 가장 잘 표현하는가?
3. 추출된 문화 지식이 소규모/약한 모델의 문화적 추론 능력을 향상시킬 수 있는가?

저자들은 처음엔 모국어로 나타낸 문화가 진정한 representaiton을 만들 수 있을거라 가정했지만, 실험을 통해 영어가 일관성 있는 문화적 지식 그래프를 생성한다는 것을 발견했다.
- 모델이 학습한 언어의 편향이 있고, 모델이 영어에서 가장 잘 훈련되어 있어 논리적으로 더 잘 짜여진 그래프를 생성
- 즉, "진짜 그 문화를 잘 아는 것"과 "그 지식을 잘 정리해서 표현하는 것"이 다른 차원의 문제라는 것

이전 work에서는 Wikipedia, CommonCrawl, social media를 통해 cultural knowledge base를 구축했다. 하지만 2가지 limitations이 있다. 
* 기존 접근법은 문화를 고립된 정적 사실로 표현하여, 문화적 관행의 절차적·맥락적 특성을 간과한다.
    - 많은 전통은 순서가 있는 행동 시퀀스로 구성되며(예: 인도네시아 결혼식의 청혼-결혼 과정), 이를 원자적(atomic) 진술로 축소하면 추론과 스토리 생성에 활용할 핵심 맥락이 누락된다.
    - ```Text
      인도네시아 전통 결혼 과정
      1. 남자 가족이 여자 집을 방문 (Melamar)
      2. 양가 가족이 결혼 날짜 협의
      3. 약혼식 진행 (Tunangan)
      4. 신랑이 지참금(Mas Kawin) 준비
      5. 결혼 전날 밤 전통 의식 (Midodareni)
      6. 결혼식 당일 아침 신랑이 신부 집 방문
      7. 종교적 서약 (Akad Nikah)
      8. 하객들과 피로연
      9. 신혼부부가 양가 부모님께 인사
         
      Atomic Statement (기존 지식 베이스)
      - 인도네시아 결혼식에는 지참금이 있다
      - 인도네시아 결혼식에는 Akad Nikah가 있다
      - 인도네시아 결혼식에는 피로연이 있다
      
      한계점 (파악 못 하는 것)
      - 순서: Midodareni가 결혼식 전날 밤이라는 것
      - 인과관계: 지참금을 준비해야 그 다음 Akad Nikah가 가능하다는 것
      - 조건: 양가 날짜 협의가 완료되어야 약혼식이 진행된다는 것
      - 맥락: 각 단계가 왜 존재하는지, 어떤 의미인지
        
      추론 질문: "인도네시아 남자가 결혼하려면 무엇을 먼저 해야 하나?"
      - Atomic 방식: 지참금이 필요하다, Akad Nikah가 있다 → 순서를 모르니 답 불가
      - CCKG 방식: Melamar → 날짜 협의 → Tunangan → 단계별로 추론 가능
    ```
	- 다른 논문에서 다뤘던 [[(ACL'25, short) Towards Geo-Culturally Grounded LLM Generations#4. Human Evaluation (인간 평가)|문제]]를 해결할 수 있는 방법 중 하나일 것 같다.
* 대부분의 문화 지식 베이스는 영어로 구축되기 때문에 문화적 뉘앙스를 잘 나타내지 못한다.
	* 한국어 '눈치'를 정확하게 표현할 수 있는 영어 표현이 없다.
    - 논문에서도 영어 추출이 더 일관적인 답변을 생성한다는 것은 LLM 모델의 한계를 나타내는 것이지 위의 사실을 비판하는 것은 아님.

![[Figure1.png|Figure 1: Application of our framework for constructing a partial Cultural Commonsense Knowledge Graph (CCKG) capturing culturally grounded reasoning about breakfast in Indonesia. Given an input prompt specifying the subtopic, language, country, and task-specific constraints, GPT-4o generates English if–then commonsense assertions (actioni ,relation, actionj ) to form an initial knowledge base (KB). Assertions with relations (xNext, oNext) are iteratively expanded by reprompting GPT-4o to generate intermediate action expansions that decompose actioni into finer-grained steps leading to actionj and forward actions occurring after actionj . In this example, only the first assertion in the expansion list is expanded for a single iteration. The resulting assertions are added to the KB, post-processed and composed into the final CCKG subgraph.|625]]


[Contribution]
* CCKG 프레임워크 제안 (Figure 1)
	* LLM에서 문화 특화 if-then 추론 체인을 추출하는 반복적 프롬프트 기반 방법.
	- ATOMIC 스타일을 따라 단일 관계를 다단계 체인으로 확장.
	- LLM에서 문화 지식을 체계적으로 뽑아내는 방법을 만듦.
* 언어별 품질 비교 발견 (인간이 평가)
    - 모국어 → 더 풍부한 문화적 세부사항.
    - 영어 → 더 일관적이고 전반적으로 선호됨.
    - 모국어가 뉘앙스는 풍부하지만, 영어 추출이 전체적으로 더 잘 작동함.
- 소규모 LLM 성능 향상
    - 소규모/약한 LLM에 CCKG를 증강하면 성능 향상
    - 문화적 추론 + 스토리 생성 두 task 모두에서 효과 확인

# 2. Related Work

## 2.1. Knowledge Bases for Commonsense Reasoning.

**[Knowledge Base]**
**(2014) WebChild**
- 명사에 형용사를 연결하는 방식
- ```Text
  눈(snow) → isWhite → 하얗다
  불(fire) → isHot → 뜨겁다
  개(dog) → hasFourLegs → 네 발이 있다.
  ```
* 한계: 단순한 물리적(physical)/지각적(perceptual) 속성만 표현하기 때문에 사건 간 관계가 없다.

**(2017) ConceptNet**
* 사람들이 직접 작성한 assertion을 triplet 형태로 표현한 지식 그래프    
    - Assertion: 사실이라고 주장하는 문장
    - Ex) "인도네시아에서 아침을 먹으려면 론통을 먹는다"
- ```Text
   칼(knife) → UsedFor → 자르기 
   의사(doctor) → CapableOf → 치료하기 
   사과(apple) → IsA → 과일
  ```

**(2019) Quasimodo**
- QA 포럼(Reddit, Yahoo Answers 등)과 웹 텍스트에서 자동으로 상식 assertion을 mining
- ```Text
   왜 사람들은 우산을 들고 다니나요? → 비가 올 때 쓰기 위해 
   코끼리는 왜 물을 좋아하나요? → 피부가 건조해지기 때문에
  ```
- 한계: 여전히 사건 간 인과/순서 추론보다는 사실적 설명에 집중

**(2019) ATOMIC**
- 이벤트 수준의 if-then 추론으로 전환  → 인과/영향/사회적 행동을 추론.
- ```Text
  X가 파티를 연다 
  → xNeed: X는 음식을 준비해야 한다 
  → xEffect: X는 피곤해진다 
  → oEffect: 다른 사람들이 즐거워한다 
  → xNext: X는 손님들을 맞이하고 싶어한다 
  → oNext: 다른 사람들은 선물을 가져오고 싶어한다
  ```
- 단순 사실이 아니라 "이 일이 일어나면 그 다음엔?" 이라는 추론 가능    
- 한계: 대부분 서구 중심적, 문화 간 일반화 부족


**[Culture Knowledge Base]**
**(2023) CANDLE**
- 웹에서 지역별 사실적 지식을 대규모로 추출
- ```Text
  한국에서는 설날에 세배를 한다
  인도에서는 소를 신성하게 여긴다
  ```

**(2024) Mango**
- LLM에 직접 프롬프트해서 문화 간 사실 추출
- ```Text
  프롬프트: "이집트의 결혼 문화에 대해 알려줘" 
  → "이집트에서는 결혼식에 대가족이 참여한다"
  → "신랑이 신부 가족에게 Mahr를 제공한다"
  ```

**(2024) CultureAtlas**
- Wikipedia + Wikidata에서 문화 지식 베이스 구축
- ```Text
  Wikipedia "Japanese Wedding" 항목 
  → "산산구도(三三九度): 신랑신부가 술을 세 번 나눠 마신다" 
  → "시로무쿠(白無垢): 전통 흰 기모노를 입는다"
  ```

**(2024) CultureBank**
- TikTok, Reddit 등 소셜 미디어에서 문화적 기술자 수집
- ```Text
  TikTok 댓글: "인도네시아에서는 어른한테 두 손으로 물건을 드려야 해" 
  Reddit: "중국에서는 시계를 선물하면 안 돼, 죽음을 의미하거든"
  ```

**공통된 한계**
- 문화를 고립된 정적 사실로 표현하여, 문화적 관행의 절차적·맥락적 특성을 간과.
- 많은 전통은 순서가 있는 행동 시퀀스로 구성되며, 이를 원자적 진술로 축소하면 추론과 스토리 생성에 활용할 핵심 맥락이 누락됨.
- ```Text
  기존 방식: "인도네시아 결혼식에는 지참금이 있다" ← 고립된 사실 
  CCKG 방식: (순서와 인과관계가 있는 체인) 청혼 방문 → 날짜 협의 
  → 약혼식 → 지참금 준비 → 전날 밤 의식 → 종교적 서약 → 피로연
  ```

## 2.2. Evaluating Cultural Commonsense Knowledge.

이전 works는 대부분 보편적인 서구적 문화권에 대한 물리적 상식 및 사회적 추론(인간의 감정, 의도, 사회적 규범 등)에 대한 평가였다.
최근에는 문화적 상식 추론을 검토하기 시작했다.
- 11개 인도네시아 지방 상식을 추론하는 IndoCulture와 아랍 문화를 위한 보완적 벤치마크 ArabCulture를 제안했다.

또 다른 연구는 상식 추론을 넘어 문화적 다양성을 탐구했다. 
- World Values Survey기반으로 LLM 생성 응답의 국가 간 차이를 분석하는 GlobalOpinionQA를 소개.
- CultureNLI는 인도와 미국 문화적 맥락에서의 implicit 관계를 검토

본 논문에서는 IndoCulture, ArabCulture로 모델을 평가한다.
- 문화적 상식 추론을 직접 타겟으로 하며 고품질 수동 주석으로 구성되어 있기 때문

# 3. Cultural Commonsense Knowledge Graph (CCKG)

## 3.1. Preliminaries

![[3.1. preliminaries.webp|330x230]]

* Action
	* 문화 기반 행동을 의미하는 활동, 이벤트, 프로세스를 설명하는 구문
* Assertion
	* Triple $A_i, R, A_j \in E$
	* 행동 $A_i$와 행동 $A_j$ 사이의 인과적(causal), 동기적(motivational), 결과적(consequential) 연결을 정의하는 관계 $R$을 통해 특정 문화 상식을 표현
* Path
	* $A_0 \xrightarrow{R_1} A_1 \xrightarrow{R_2} A_2 \xrightarrow{R_3} \cdots \xrightarrow{R_k} A_j$
	* 각 $A_{i-1}, R_i, A_i \in E \ \text{ for } \ i = 1, \dots, k$에서 $A_0$는 initial action, $A_k$는 resulting action, $A_1, \cdots A_{k-1}$은 intermediate actions이다.
	* 처음 action에서 마지막 action까지 relation을 통해 연결하는 assertion의 sequence이다.
* Relation
	* ![[Table1.webp|Table 1: Types of relations between actions. Three are adopted from ATOMIC Sap et al. 2019a (oEffect, xEffect, xNeed) and two are newly introduced (oNext, xNext). In these relation types, x denotes the agent or primary person performing the action, while o refers to others who interact with or are affected by x’s action. Text in brackets shows the "if $action_1$, then $action_2$" version.]]
	* 5가지 관계 타입을 정의
	* ```Text
	  (Ai, xNext, Aj) → x가 Ai 후 Aj를 하고 싶어함
	  * (소토 아얌을 주문한다, xNext, 삼발 소스를 추가 요청한다)
	  * x가 Ai 후에 자발적으로 Aj를 하고 싶어함
	
	  (Ai, xEffect, Aj) → Ai가 x에게 Aj 결과를 가져옴
	  * (소토 아얌을 먹는다, xEffect, 따뜻함과 포만감을 느낀다)
	  * Ai가 x에게 자동으로 Aj라는 결과를 가져옴
	  
	  (Ai, xNeed, Aj) → Ai 전에 x가 Aj를 해야 함
	  * (와룽에서 아침을 먹는다, xNeed, 집을 나서야 한다)
	  * Ai를 하려면 먼저 Aj가 필요함
	  
	  (Ai, oNext, Aj) → x가 Ai 하면 타인이 Aj를 함
	  * (와룽에 들어온다, oNext, 주인이 주문을 받으러 온다)
	  * x가 Ai를 하면 타인이 Aj를 하고 싶어함
	  
	  (Ai, oEffect, Aj) → x가 Ai 하면 타인에게 Aj 발생
	  * (음식값을 지불한다, oEffect, 와룽 주인이 기뻐한다)
	  * x가 Ai를 하면 타인에게 Aj라는 결과가 생김
	  ```
	  
## 3.2. Knowledge Graph Construction

CCKG는 iterative, prompt-based 방법으로 영어 또는 대상 국가의 모국어로 구축할 수 있다.
![[Algorithm1.jpg]]
총 2단계로 나눠져 있다.
### 3.2.1. Initial Generation
![[Figure5.webp]]

생성된 assertions는 knowledge base $\mathcal{K}_c^L$에 저장한다.
```json
{
    "action": "looks for breakfast",        ← Ai
    "knowledge": "finds soto ayam",         ← Aj
    "relation_type": "xNext",               ← R
    "result": "If someone looks for breakfast, then they find soto ayam"
}
```


* 나중에 downstream task(MCQA, 스토리 생성)에서 모델에게 context로 제공할 때 자연어 형태가 필요하기 때문에 result를 생성.

### 3.2.2. Iterative Expansion

![[Figure6.png]]

oNext, xNext인 assertions $(A_i, R, A_j)$ with $R \in \{\text{oNext, xNext}\}$에서 시작해 사용자 지정 확장 횟수 $N$에 걸쳐 path를 정교화한다.
1. Intermediate action expansion
	* LLM이 각 assertion을 decompose 해서 $A_i$와 $A_j$ 사이에 intermediate steps를 추가해 sequence를 생성한다.
	* $A_i \xrightarrow{R_0} \cdots \xrightarrow{R_k} A_j$
	* Intermediate triples를 $\mathcal{K}_c^L$에 저장한다.
2. Forward expansion
	* $A_j$를 새로운 시작점으로 해서, LLM이 새로운 후속 assertion $A_j, R^\prime, A_k$ where $R^\prime \in \{\text{oNext ,xNext}\}$을 생성한다.
	* 중복 expandsion을 방지하기 위해 $\mathcal{K}_c^L$에 존재하는 모든 action 집합 $\mathcal{U}$을 가져와 생성된 action $A_k$와의 유사도를 통해 $A_k$를 대체하거나 추가한다.
		* 유사도가 큰 게 이미 있으면 $A_k$를 해당 action으로 대체한다.
	* $A_j$마다 생성되는 assertion 개수가 다르므로 최대 3개로 제한한다.
		* Cost(메모리/시간)을 고려해 균형 있게 전체 주제를 다루기 위해 제한한다.
		* Iteration이 반복될수록 지수적으로 knowlege가 확장되기 때문에 제한이 필요한다.
 
```Text
(예시) 국가: 인도네시아, 하위 주제: 아침(Breakfast)

Step 1 Initial Generation에서 한 번에 여러 assertions 생성:
(아침을 찾는다, xNext, 소토 아얌을 찾는다)
(아침을 찾는다, xNext, 와룽에서 아침을 주문한다)
(소토 아얌을 찾는다, xEffect, 따뜻하고 풍미 있는 국물 요리의 온기를 느낀다)
(아침을 찾는다, xNext, 론통을 먹는다)
...

Step 2 Iterative Expansion에서 추가 assertions 생성:
(아침을 찾는다, xNext, 론통을 먹는다) 확장
→ (론통을 먹는다, xNext, 전통 반찬을 고른다)
→ (전통 반찬을 고른다, xNext, 소토 아얌을 찾는다)
→ (소토 아얌을 찾는다, xNext, 와룽에서 아침을 주문한다)
→ (소토 아얌을 찾는다, xNext, 삼발 반찬을 주문한다)
```

# 4. Experiments
## 4.1. CCKG Extraction

[Topic Taxonomy and Country Selection.]
5개국(중국, 인도네시아, 일본, 영국, 이집트)을 선정하여 지리적 다양성과 문화적 다양성을 확보했으며, 각국의 모국어는 중국어(CHI), 바하사 인도네시아어(IND), 일본어(JAP), 영어(EN), 현대 표준 아랍어(MSA)이다.

Koto et al. (2024)에서 가져온 ==11개의 일상 주제와 65개의 세부 주제==를 정의했으며, 음식, 결혼식, 예술, 습관, 일상 활동, 가족 관계, 임신 및 육아, 죽음, 종교 공휴일, 전통 놀이, 사회종교적 관행 등 다양한 일상 영역을 포함한다.

| Topics                          | Subtopics                                                                                                                                                                                                                                                                                                |
| ------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Food                            | Breakfast, lunch, dinner, traditional foods and beverages, cutlery, cooking ware, fruit, food souvenirs, snacks                                                                                                                                                                                          |
| Wedding                         | Wedding location, wedding food, wedding dowry, traditions before marriage, traditions when getting married, traditions after marriage, men’s wedding clothes, women’s wedding clothes, songs and activities during the wedding, invited guests at a wedding, gift brought to weddings, food at a wedding |
| Habits                          | Eating habit, greetings habits, financial habits (saving, debit/credit), punctuality habit, cleanliness habit, shower time habit, transportation habit, popular sports                                                                                                                                   |
| Art                             | Musical instruments, folks songs, traditional dances, use of art at certain events, poetry or similar literature                                                                                                                                                                                         |
| Daily activities                | Morning activities, afternoon activities, evening activities, leisure and relaxation activities, household activities (cleaning, home management)                                                                                                                                                        |
| Family relationship             | Traditions during pregnancy, traditions after birth, how to care for a newborn baby, how to care for toddlers, how to care for teenagers, parents and children interactions as adults                                                                                                                    |
| Pregnancy and kids              | Traditions during pregnancy, traditions after birth, how to care for a newborn baby, how to care for toddlers, how to care for teenagers, parents and children interactions as adults                                                                                                                    |
| Death                           | When death occurs, the process of dealing with a corpse, traditions after the body is buried, the clothes of the mourners, inheritance matters                                                                                                                                                           |
| Religious holiday               | Traditions before religious holidays, traditions leading up to religious holidays, traditions during religious holidays, traditions after holidays                                                                                                                                                       |
| Traditional games               | Traditional game types                                                                                                                                                                                                                                                                                   |
| Socio-religious aspects of life | Regular religious activities, mystical things, traditional ceremonies, lifestyle, self care, traditional medicine, traditional sayings                                                                                                                                                                   |

[Evaluation Setup]
CCKG의 품질을 평가하기 위해 이진 레이블(yes/no)을 사용하여 세 가지 차원에서 manual evaluation를 수행했다.
1. 정확성(COR): $A_i$와 $A_j$가 유효한 행동인지, 관계 $R$이 정확한지를 평가
2. 문화적 관련성(CR): 해당 주장이 대상 국가에 문화적으로 특화되어 있는지를 평가
3. 논리적 경로 일관성(LPC): 행동의 순서가 일관되고 논리적으로 구조화된 모순 없는 추론 체인을 형성하는지를 평가

평가는 해당 언어의 원어민이면서 영어 능통자, 최소 고등학교 졸업 이상의 학력을 가진 전문 주석자들이 수행했다.
* 각 국가마다 2명의 평가자를 모집했으며, 부주의한 응답을 방지하기 위해 5개의 골드 스탠다드 샘플을 무작위로 삽입했고, 최소 4개 이상 정확히 레이블링해야 했다. 모든 주석자는 해당 국가의 최저 임금으로 보상받았으며, 각 작업은 평균 약 3시간이 소요되었다.

[Preliminary Experiments]
Main 실험에 가장 적합한 model을 선택하기 위해 GPT-4o(클로즈드 소스)와 Llama-3.3-70B-IT(오픈 소스)를 비교했다.
* 중국과 인도네시아에 집중했다.
* [[(EACL'26, main) LLMs as Cultural Commonsense Knowledge Graph Extraction#3.2.1|iterative expansion stage]]는 [[(EACL'26, main) LLMs as Cultural Commonsense Knowledge Graph Extraction#3.2.1|first-stage extraction]]의 영향을 많이 받기 때문에 first-stage extraction에 관해 CR, COR 평가를 진행했다.
	* 11개 주제에 걸쳐 100개의 assertion을 무작위 샘플링하였다.

![[Figure7.png|456x354]]
GPT-4o가 Llama-3.3-70B보다 일관되게 우수한 성능을 보여 이후 모든 실험에 GPT-4o를 채택했다.

[Extraction and Evaluation]
GPT-4o를 사용하여 각 국가에 대해 Algorithm 1을 적용하고, 영어와 해당 모국어 두 가지로 주장을 생성했다 (temperature = 1, N = 3).
* temperature = 1: 생성의 다양성을 높이기 위해 설정 (0에 가까울수록 결정적, 1에 가까울수록 다양한 출력)
* N = 3: 반복 확장(Iterative Expansion)을 3번 수행

중복을 제거하기 위해 영어에는 all-MiniLM-L6-v2, 다른 언어에는 stsb-xlm-r-multilingual 문장 임베딩을 사용했다.
* 즉, 텍스트가 완전히 똑같지 않더라도 의미적으로 유사한 주장은 중복으로 간주하여 제거하였다.


![[Table7.png|413]]
중복 및 잘못된 형식의 주장을 필터링한 후, 38,858개 중 37,363개의 영어 assertion과 17,043개 중 16,709개의 모국어 assertion이 남았다. 이를 바탕으로 27,649개의 영어 path와 6,571개의 모국어 path를 구성했다.
* 영어가 모국어보다 훨씬 많은 assertion을 생성한 것을 알 수 있다.
* 이는 LLM이 영어로 더 풍부하게 생성한다는 것을 간접적으로 보여준다.

[Result]
![[Table2.png|411]]

영어 CCKG가 거의 모든 평가에서 모국어 버전을 일관되게 능가했다. 평균적으로 영어 버전이 정확성, 문화적 관련성, 논리적 경로 일관성에서 더 높은 점수를 기록하여, LLM이 영어로 문화 지식을 더 정확하고 일관되게 표현함을 보여준다.
* 이 패턴은 아랍어, 중국어, 일본어 등 다양한 언어 계열에서 공통적으로 나타났다.
* 모국어 CCKG는 지역적 뉘앙스를 포착하기도 하지만, 언어별 사전 학습 데이터 부족으로 인해 <mark style="background: #FFF3A3A6;">덜 일관된</mark> 추론 체인을 생성하는 경향이 있다.
* 이는 현재 다국어 LLM의 핵심 비대칭성을 보여준다: 지역 문화 추론을 목표로 하면서도 영어를 통해 문화 상식을 가장 효과적으로 표현한다.

## 4.2. Evaluation on Cultural Commonsense Reasoning

[Dataset]
두 개의 인간 구축 벤치마크를 사용했다.
* ArabCulture는 13개 아랍 국가의 문화 상식을 현대 표준 아랍어(MSA)로 다룬다. (이집트로 포함)
* IndoCulture는 11개 인도네시아 지방의 문화 추론을 Bahasa 인도네시아어로 다룬다. 
 
두 데이터셋 모두 다양한 문화 영역을 포함하며, MCQA(Multiple-choice question answering)과 sentence completion tasks(open-ended generation) 두 가지 형식으로 평가된다.
* MCQA는 accuracy로 평가한다.
	* ```Text
	  질문: 인도네시아에서 결혼식 전날 밤에 신부 가족이 전통적으로 하는 것은?

	  1. 신부에게 henna(헤나) 문신을 그려준다
	  2. 신부에게 케이크를 선물한다
	  3. 신부와 함께 영화를 본다
	
	  정답: 1
	  ```
* Sentence completion은 BERTScore-F1과 sentence similarity로 평가한다.
	* BERTScore-F1은 언어 모델의 semantic embedding을 사용하여 두 문장의 의미적 유사도를 측정한다.
	* Sentence Similarity는 단어 수준의 표면적 유사성 (비슷한 단어/표현을 사용했는지)를 측정한다.
	*  ```Text		
		입력: 인도네시아에서 결혼식 전날 밤에 
		      신부 가족이 전통적으로 _______
		
		모델 출력 예시: "신부에게 헤나 문신을 그려주며 
		               축복을 기원하는 의식을 진행한다"
		
		참조 정답: "신부에게 헤나 문신을 그려주는 
		           전통 의식을 행한다"
		
		```

| 상황               | BERTScore | Sentence 유사도 |
| ---------------- | --------- | ------------ |
| 의미는 같지만 표현이 다름   | 높음        | 낮음           |
| 표현은 비슷하지만 의미가 다름 | 낮음        | 높음           |
| 표현도 비슷하고 의미도 같음  | 높음        | 높음           |
* 두 지표를 함께 보면 의미적 정확성과 표현적 유사성을 동시에 포착할 수 있기 때문에 보완적으로 사용.

[Models]
총 13개 모델을 실험했다.
* 기본 모델 Llama3.2-1B/3B, Llama3.1-8B, Qwen2.5-0.5B/1.5B/3B/7B, Gemma2-2B/9B -> MCQA
* Instruction-tuned 모델 Llama3.1-8B-I, Gemma2-9B-I, Qwen2.5-7B-I  -> MCQA + Sentence completion

[Augmentation Methods]
CCKG에서 관련된 assertion 또는 path를 가져와 두 가지 task에서 in-context augmentation을 진행했다.
저자들은 SBERT embedding(stsb-xlm-r-multilingual)에 기반해 semantic search하여 가장 관련있는 assertions, paths를 찾았다.
* 5개의 assertion과 1개의 path를 가져온다.

| 설정     | 출처       | 제공 정보 형태              | 언어  | 개수  | 사용 태스크            |
| ------ | -------- | --------------------- | --- | --- | ----------------- |
| Base   | -        | 없음                    | -   | -   | MCQA, 문장완성, 스토리생성 |
| CoT    | -        | 없음 (추론 유도만)           | -   | -   | MCQA              |
| Mango  | Mango KB | 단순 사실 문장              | 영어  | 5개  | MCQA, 문장완성        |
| E-Asrt | CCKG     | if-then 주장(assertion) | 영어  | 5개  | MCQA, 문장완성        |
| E-Path | CCKG     | 연결된 경로(path)          | 영어  | 1개  | MCQA, 문장완성, 스토리생성 |
| N-Asrt | CCKG     | if-then 주장(assertion) | 모국어 | 5개  | MCQA, 문장완성        |
| N-Path | CCKG     | 연결된 경로(path)          | 모국어 | 1개  | MCQA, 문장완성, 스토리생성 |

[Results on MCQA]
![[Table3.png]]

Accuracy를 평가할 때 benchmark prompt(english)를 그대로 사용했다.
* KB를 어떻게 prompt에 적용하는지 안 나옴. (논문과 code 모두)
* 모국어 prompt를 사용해도 비슷한 경향이 나옴. (논문 Table 14 참고)

CCKG 지식 통합은 IndoCulture에서 일관되게 성능을 향상시켰다. 
ArabCulture에서는 Mango가 아닌 CCKG 주장에서만 개선이 관찰되었다.

가장 큰 향상은 모국어 assertion으로 증강했을 때 나타났다.
* Qwen2.5-7B는 인도네시아어 assertion으로 IndoCulture에서 58.5% → 60.8%, 아랍어 assertion으로 ArabCulture에서 49.3% → 59.6%로 향상되었다.
* 평균적으로 모국어 증강이 +1.2포인트로 가장 높은 향상을 달성했다.

| Section | 측정 대상                             | 결과              |
| ------- | --------------------------------- | --------------- |
| 4.1     | CCKG 자체의 품질 (정확성, 문화 관련성, 경로 일관성) | 영어 CCKG가 더 높음   |
| 4.2     | CCKG를 활용했을 때 모델 성능 향상             | 모국어 CCKG가 더 효과적 |

(Section 4.1)
* LLM은 영어 데이터로 더 많이 학습됨
* 영어로 생성할 때 더 정확하고 일관된 추론 체인 생성
* 사람이 평가해도 영어 CCKG가 더 논리적으로 보임
(Section 4.2)
* IndoCulture 질문: 인도네시아어로 작성됨
* 모국어 CCKG: 인도네시아어로 작성됨
* 같은 언어끼리 매칭되어 모델이 더 직접적으로 참고 가능
* * 영어 CCKG를 인도네시아어 질문에 사용하면 언어 변환 과정에서 뉘앙스 손실 가능

Small 또는 base 크기의 model에서 in-context 증강이 효과적이었다.
* CCKG의 명시적 추론 단서가 제한된 내재적 문화 지식을 보완하기 때문

반면 Gemma2-9B-IT, Qwen2.5-7B-IT 같은 large instruction tuned model은 이미 일반적인 문화 지식을 인코딩하고 있어 추가 context의 효과가 적었다.

CoT prompting은 거의 모든 모델과 데이터셋에서 baseline보다 낮은 성능을 보였다. 이는 문화 상식 추론이 단계별 논리적 추론보다 직관적이고 맥락 의존적인 지식에 더 의존함을 시사한다.
* IndoCulture에서 평균 3.8포인트, ArabCulture에서 4.6포인트 하락

[Results on Sentence Completion]
![[Table4.png]]

CCKG 컨텍스트 통합은 전반적으로 유사도와 BERTScore-F1을 향상시켰다.
* 모국어 assertion과 path가 유사도에서 가장 높은 향상을 보였다.
	* Llama3.1-8B-IT는 IndoCulture에서 32.6% → 36.0%, ArabCulture에서 29.9% → 33.5%로 향상
	* Qwen2.5-7B-IT도 각각 39.6% → 43.0%, 38.8% → 42.4%로 향상.

## 4.3. Evaluation on Story Generation

![[Figure10.png]]

25개의 무작위 선정 세부 주제를 대상으로 스토리 생성 작업을 수행했다. 이집트, 중국, 인도네시아의 영어 및 모국어로 스토리를 생성했으며, 세 가지 설정을 비교했다.
* Zero-shot
* Mango(5-shot assertion)
* CCKG(1-shot 경로).

![[Table5.png]]

* Section 4.2와 같이 assertion, path는 SBERT embedding 유사도로 찾음.
* 평가 지표 (English Story)
	* CR (Cultural relevance)
		* 전통, 관습, 가치, 사회 규범의 정확한 반영 여부
	* FL (Fluency)
		* 문법적 정확성, 문장 구조, 어휘, 가독성
	* CO (coherence)
		* 사건과 캐릭터 행동의 논리적 흐름과 일관성.

CCKG path가 story 품질을 일관되게 향상시켰으며, 가장 큰 향상은 CR에서 나타났다. FL와 CO도 조금 향상되어, CCKG path 기반 augmentaiton이 논리적으로 구조화된 story 생성에 도움을 준다.

![[Figure2.png]]

Baseline 대비 CCKG augmentation 향상률
* 3개 국가 x 3개 model x 영어/모국어 스토리 생성 = 18개 비교
* 영어가 평균적으로 높게 나옴. (CCKG path 품질이 영어가 상대적으로 더 좋았던 것과 관련 있나...?)

LLM-as-a-Judge 분석
* CR 평가에서 모국어에서의 LLM 평가는 인간 평가와 잘 일치(평균 상관관계는 0.8)했지만, 영어에서는 중간 정도의 일치(평균 상관관계는 0.4)만 보였다.
* FL 평가와 CO 평가에서 영어는 유창성(–0.1)과 일관성(0.0)에서 음의 상관관계가 나타났다. 
* 반면, 모국어의 FL 평가와 CO 평가에서 사람과 LLM의 상관관계는 각각 0.9, 0.8이었다.
	* 한 가지 가능한 설명은 영어 평가가 문체적 변화(stylistic variation)에 더 민감할 수 있다는 것이다. (사람마다 글쓰기 스타일에 선호도가 다르다는 말.)
		* 예를 들어 간결한(concise) 글쓰기와 정교한(elaborative) 글쓰기에 대한 선호도 차이가 있을 수 있다. 
	* 또한 문화적으로 특수하거나 전통과 관련된 표현들은 영어 번역보다 모국어에서 더 자연스럽고 진정성 있게 들리는 경우가 많으며, 이는 평가자들의 선호도에 추가적인 영향을 미칠 수 있다.

# 5. Conclusion

Contribution
* 정적이고 영어 중심적인 자원을 넘어 다국어 문화 상식 지식 체인을 구성하는 프레임워크를 제시
* 문화를 고립된 사실이 아닌 절차적/순차적으로 모델링함으로써 언어 전반에 걸쳐 문화적 관행의 흐름을 저장.
* LLM의 암묵적 문화 지식을 구조화된 형태로 추출
* 경험적으로 CCKG로 LLM을 증강하면 문화 상식 추론과 스토리 생성 성능이 향상

Results
* 인간 평가 결과, 모국어가 더 풍부한 문화적 깊이를 전달하지만 영어 출력이 더 일관되고 선호된다는 것을 보여줌.
	* 모국어 CCKG -> 문화적 깊이는 더 풍부
	* 영어 CCKG -> 일관성과 선호도는 더 높음

Limitation
* 프롬프트 민감성
	* CCKG 구성 방법은 프롬프팅에 의존하기 때문에 특정 프롬프트 형식에 민감하다. 
	* 따라서 새로운 모델에 적용할 때 어느 정도의 프롬프트 튜닝이 필요할 수 있다. 
* 고정관념 위험성 (Future work)
	* LLM에서 문화 상식 지식을 자동으로 추출하면 고정관념을 재현할 잠재적 위험이 있다. 
	* 이 연구에서는 그러한 편향을 탐지하거나 평가하는 데 집중하지 않았다.
	* 그러나 인간 평가자들이 추출된 내용의 일부를 품질 분석을 위해 검토했으며, 대부분의 항목은 고정관념적이거나 유해한 내용으로 표시되지 않았다. 이 문제에 대한 더 심층적인 조사는 향후 연구에서 수행할 계획이다.
* 문화 범위의 한계
	* 이 연구는 resource가 풍부한 문화에 집중했다. 인간 평가가 가능해야 하고, LLM이 사전 학습 중에 이미 상당한 지식을 습득했다는 가정을 반영하기 위함이다.
	* 또한 문화를 국가 수준에서 평가했으며, 향후 더 세밀한 수준으로 확장할 계획이다.
* 데이터 사용 주의사항
	* LLM에서 추출한 데이터는 연구 프로토타입이자 LLM을 문화 아카이브로 보는 개념의 탐색적 기반으로 사용된다.
	* 이 추출된 내용은 공식 데이터셋으로 간주되어서는 안 된다.
	* 고정관념 강화나 기타 의도치 않은 편향의 가능성을 포함한 도전과 위험을 신중히 고려하지 않고 프로덕션 시스템에서 사용하는 것을 권장하지 않는다.