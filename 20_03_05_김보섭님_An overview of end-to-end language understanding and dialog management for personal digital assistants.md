# An overview of end-to-end language understanding and dialog management for personal digital assistants

## Abstract
 Spoken language understanding and dialog management have emerged as key technologies in interacting with personal digital assistants (PDAs). The coverage, complexity, and the scale of PDAs are much larger than previous conversational understanding systems. As such, new problems arise. ***In this paper, we provide an overview of the language understanding and dialog management capabilities of PDAs, focusing particularly on Cortana, Microsoft’s PDA.*** We explain the system architecture for language understanding and dialog management for our PDA, indicate how it differs with prior state-of-the-art systems, and describe key components. We also report a set of experiments detailing system performance  on a variety of scenarios and tasks. We describe how the quality of user experiences are measured end-to end and also discuss open issues.

## Introduction
- `Personal digital assistant (PDA)`는 어플리케이션 또는 서비스의 인터페이스로 활용되고 있음.
	 `PDA`는 `proactive` 또는 `reactive`한 방식으로 갈림	
	
	> With `proactive assistance`, the system takes an action based on the events it has been tracking

	> With `reactive assistance`, the system responds to the user's explicit spoken or typed request

- 본 논문에서는 `inhomogenous multiple back-end service`, `open domain dialog`를 잘 수용할 수 있는 구조를 제안함.
	
	> *In the system we describe here, language understanding and some of the dialog components are centralized to provide additional cross-domain flexibility and re-use, improving the system's ability to ramp up support for new domains.*

## PDA reactive assistance requirements
사용자가 아닌 개발자의 관점에서는 적절한 비용으로 사용자의 기대를 만족시키는 것을 목표로 한다. 아래의 사항들을 고려할 필요가 있다.  먼저 아래의 용어를 정의

1. the breadth of LU domains and experiences
	> *an `experience` as the PDA equivalent of a smart phone or desktop application.*

	> *an `LU domain` as a collection of related `intents` and `slots` for which no operational conflict arises in the semantic space that they span.*

2. the naturalness of user language

3. the complexity of dialogs
	> *`conversation structure` ranges from unstructured experiences (e.g. single-turn responses that present query results), multi-turn browsing/search refinement, chit-chat style dialogs, through to goal-oriented, transactional interactions.*

	> *`allowed forms of initiative;` beyond who can influence the dialog flow, e.g. system, user or mix initiative dialogs, limits may exist on the form of initiative can be taken.*

	> *`information sharing between experiences;` options include treating each dialog as being independent with no sharing of information, long-term storage of information related to individuals, short term storage and passing of information between different experiences.*

	> *`automation level` ranges from fully-automated dialogs to human-in-the-loop, the latter allowing more complex queries to be handled by a human agent. This directly impacts the trade-off between latency and accuracy - dimension 7.*

4. the range of modalities and devices the PDA can interact with

5. the supported range of expertises of experience authors

6. the latency and capacity of back-end or cloud services that can be accommodated

7. the overall latency and accuracy of system responses

8. allowable costs, e.g. computational, implementation, and maintenance

9. support for development of uniform user experiences, e.g. the PDA "personality"

10. support for easy upgrading of experiences

## System architecture
본 논문에서 제안하는 구조는 넓게는 아래의 세 개로 나누어 볼 수 있으며, 각각의 세 개의 단계에서 여러가지 가능한 hypotheses를 추적하고, 마지막에 가장 좋은 것을 고른다.
![architecture](https://raw.githubusercontent.com/aisolab/blog/master/_posts/_An%20overview%20of%20end-to-end%20language%20understanding%20and%20dialog%20management%20for%20personal%20digital%20assistants/fig1.png)

1. input processing, including LU
2. updating the dialog state, and
3. applying a policy to select and execute the system action

`experience`의 관점에서 보면 전체 플랫폼은 layer approach를 취하고 있다.

1. a web search service (base layer)
	
	> _fall-back experience for queries which cannot be processed by a more specific provider._

2. question-answering service
	> _covering a variety of domains, that directly answer user queries without requiring them to navigat to realted web-sites._

	> _To provide a more sophisticated multi-turn experience, this layer can benefit from carrying over contextual information from previous turns, especially to answer contextual questions `The slot/entity carry over (SCO) module` provides such functionality._

3. other layer
	> `use flexible item selection (FIS)` to take system initiative disambiguration turns, e.g. prompting the user to select between a number of items.
  
    > make use of session storage capabilities.
## Language understanding
`language understanding (LU)` component는 typed text 또는 speech transcription에 대해서 semantic analysis를 수행한다.
### Domain, Intent, Slot Modeling
query (e.g. typed text 또는 speech transcription)에 semantic analysis를 수행하여, `semantic frame (SF)`를 생성, `SF`는 `<domain, intent, slots>`의 tuple 형태로 구성되며, 이를 `semantic space`라고도 함. `slots`은 `<slotname, slotvalue>`의 key-value pair로 구성됨.

- 본 논문에서는 각각의 `domain` 별로 `intent`, `slot`에 관련된 모형을 구축함.

- `domain`, `intent`의 경우에는 `classification` 문제로, `slot`의 경우에는 `sequence tagging`의 문제로 접근함.
	- `slotvalue`의 경우는 더 작은 `entity`로 쪼개질 수 있음.
	  
	  > *The slot values can be further resolved into entities. (e.g. a strongly typed object in some back-end data source) or canonicalized into a standard form (e.g. time/date values).*

- `LU` modeling은 contextual manner로 이루어져야함.

  > *conversational session information from the history greatly reduces the ambiguity of the current turn.*

  > *Contextual modeling of queries reduces the likelihood of abrupt intent or domains switching leading to more coherent interaction during a multi-turn session.*

  ```
  Turn 1: "how is the weather in New York" (weather)
  Turn 2: "What about the weekend?" (weather)
  ```

    ```
  Turn 1: "how is the weather in New York" (calendar)
  Turn 2: "What about the weekend?" (calendar)
    ```

## Dialog
### Slot/Entity Carry Over (SCO) and Co-reference
> *`SCO` decides which slots from previous turns are still relevant in the current turn of a multi-turn conversation.*

```
Turn 1: "find french restaurants in seattle"
State 1: cuisine="french", place_type="restaurants", absolute_location="seattle"
Turn 2: "how about chinese"
State 2: cuisine="chinese", place_type="restaurants", absolute_location="seattle"
```

> *For experiences that use strongly typed entities, e.g. celebrity question-answer experiences, an alternative `co-reference resolution` model exists that learns relationships such as ‘him’ or ‘her’ from mining knowledge bases, e.g. Satori – Microsoft’s equivalent of Freebase or Google’s Knowledge Graph.*

### Flexible Item Selection
`PDA`는 다양한 디바이스와 폼팩터에서 실행 될 수 있으며, 시각으로 정보를 제공할 수 있는 경우가 있다. `flexible item selection (FIS)` 모형은 사람이 고를만한 것을 학습한다.

### Experience Providers
> *This multitude of experience providers execute in parallel, taking advantage of the range of signals from the multiple upstream LU models, `SCO` and `FIS`. What is ultimately shown to the user depends on the ranking and selection component, HRS, discussed next.*

### Hypothesis Ranking and Selection
> *`HRS` is the mechanism by which the platform ranks and selects the final response shown to the user. The processing is done in two stages, with an initial ranking of the hypotheses (HR) followed by selection of the best hypothesis according to rank score and other features (HS).* 

> *HS encodes meta-scenario policy decisions. In general, the top ranked item by HR is selected. However, sometimes it is desirable to override the ranking, e.g. if the two highest rank hypotheses are very close then it might be better to confirm the user’s intention rather than just picking the top one.*

> *One of the key decisions in dealing with open domain requests is the recognition of requests that are not handled by any specialized experience provider and letting such requests fall through to web search.*

## Challenges and discussions
 In this paper, we have attempted to summarize the many dimensions and constraints that are faced when developing a large scale PDA architecture, and we have proposed a PDA architecture that attempts to address these with an eye towards expansion and component reuse. ***We demonstrate experimentally that performing late binding allows the system to correct mistakes made during the initial LU analysis through use of additional features extracted from the dialog state and experience-specific providers.*** The subjective scores obtained from end-to-end analysis show that much of the user dissatisfaction is caused by the system’s inability to service queries which had been understood correctly, rather than mistakes during the understanding or ranking components - in other words, due to engineering or other practical limitations, rather than fundamental architecture decisions.

 Many unsolved challenges related to language understanding and dialog management for PDAs still remain. ***On the LU side, a key issue is that of domain scaling, i.e. the ability to understand everything a user  might say. In particular, quick ramp-up of LU models for a new domain, starting with minimal or no in domain data, is of great importance to PDA developers. On the dialog management side, of great importance is the ability to develop domain-independent state tracking and policy models, which can then be reused across all new experiences.*** From an engineering perspective, heterogeneous back-ends and application interfaces remain bottlenecks for expanding to new domains, as each new back-end requires custom query building and processing of results. Addressing these issues will likely require new trade-offs on the continuum of dimensions that span the requirements for a successful PDA.