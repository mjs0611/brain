---
id: 20260824T2318-d060
created: 2026-08-24
source_type: thought
source: framework_ppo.md (PPO Framework & Product Thinking)
tags: [레이어분리, contract, 플랫폼]
visibility: public
related: []
---

# 공통(Contract)과 개별(Decision)을 갈라두면 재사용 가능한 부분이 드러난다

> 공통(Contract): Event schema / Metric definition / Data contract / Governance
> 개별(Decision): Interpretation / Aggregation / Decision logic / Action

**해석.** 이 분리의 진짜 효용은 재사용이 아니라 **책임 소재**다. 두 층이 섞여 있으면 "이 숫자가 왜 다르냐"는 질문에 아무도 답할 수 없다. 정의가 틀린 건지 해석이 틀린 건지 구분이 안 되기 때문이다.

갈라놓으면 질문이 둘로 쪼개진다. 정의가 하나인가(Contract), 그 정의를 다르게 읽고 있나(Decision). 전자는 고쳐야 할 버그고 후자는 정상이다. 팀마다 다르게 판단하는 건 문제가 아니다 — 다르게 **정의**하는 게 문제다.

플랫폼화가 가능한 건 Contract 층뿐이다. Decision을 공통화하려 들면 아무도 안 쓰는 규칙이 된다.
