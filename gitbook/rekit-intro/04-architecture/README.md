---
description: 그림 하나가 질문 하나에 답한다. 이 장의 모든 그림은 같은 범례를 쓴다.
---

# 시스템 구성

{% hint style="info" %}
**핵심 — 서버 1대 위의 컨테이너들은 주요 Docker 망 4개로 나뉘고, 바깥과 닿는 지점은 nginx 하나다.**
{% endhint %}

## 범례

```mermaid
flowchart LR
  classDef ext fill:#F1F5F9,stroke:#64748B,stroke-dasharray:5 4,color:#0F172A
  classDef public fill:#FED7AA,stroke:#EA580C,stroke-width:3px,color:#7C2D12
  classDef internal fill:#FFFFFF,stroke:#475569,color:#0F172A
  classDef planned fill:#FFFFFF,stroke:#94A3B8,stroke-dasharray:3 3,color:#64748B
  classDef bad fill:#FEE2E2,stroke:#DC2626,stroke-width:2px,color:#7F1D1D

  subgraph NET["Docker 망 = 색 영역"]
    direction TB
    subgraph E["rekit_edge"]
      e1["컨테이너"]:::internal
    end
    subgraph B["rekit-backend_default"]
      b1["컨테이너"]:::internal
    end
    subgraph S["seaweedfs_default"]
      s1["컨테이너"]:::internal
    end
    subgraph M["rekle-monitoring_default"]
      m1["컨테이너"]:::internal
    end
  end

  subgraph BOX["상자 = 공개 수준 · 상태"]
    direction TB
    g1["외부 공개 지점"]:::public
    g2["서버 내부 전용"]:::internal
    g3["서버 밖 (인터넷 · 외부 서비스)"]:::ext
    g4["아직 없음 · 계획"]:::planned
    g5["문제 지점"]:::bad
  end

  subgraph LINE["선 = 무엇이 흐르는가 · 회색 영역 = 묶음"]
    direction TB
    l1["A"]:::internal -->|"요청 · 데이터"| l2["B"]:::internal
    l3["Prometheus"]:::internal -.->|"수집 (pull)"| l4["대상"]:::internal
    l5["운영자"]:::ext ==>|"운영자 접근"| l6["운영 도구"]:::internal
  end

  style E fill:#DBEAFE,stroke:#3B82F6,color:#1E3A8A
  style B fill:#DCFCE7,stroke:#22C55E,color:#14532D
  style S fill:#FEF3C7,stroke:#F59E0B,color:#713F12
  style M fill:#EDE9FE,stroke:#8B5CF6,color:#4C1D95
```

_색 영역은 Docker 망, 상자 색은 공개 수준, 선 모양은 흐르는 것의 종류다. 한 컨테이너가 여러 망에 붙어 있으면 대표 망 하나에 그리고 상자 안에 나머지 망 이름을 적는다._

## 이 장의 그림

| 페이지 | 그림이 답하는 질문 |
| --- | --- |
| [전체 구성](overview.md) | rekit 은 무엇과 연결되어 있는가 |
| [컨테이너와 네트워크](containers.md) | 어떤 컨테이너가 어느 망에서 서로 닿고, 무엇이 밖에 열려 있는가 |
| [요청 흐름](request-flow.md) | 요청 하나의 로그를 nginx 와 앱에서 어떻게 같은 ID 로 찾는가 |
| [앱 구조와 결제](app.md) | 앱 안에서 요청은 어떤 층을 지나고, 결제 한 건은 어떻게 확정되는가 |
| [빌드와 배포 (CI/CD)](build-deploy.md) | 검사 · 빌드 · 배포는 어떻게 나뉘고, 이미지는 어디로 옮겨져 어디에 남는가 · 롤백은 어떻게 하는가 |
| [모니터링](monitoring.md) | 숫자 · 로그 · 운영자 접근은 각각 어디로 흐르는가 |

{% hint style="warning" %}
**공개 범위** — 이 문서에는 포트 번호, 서버 주소, 내부 관리 경로를 적지 않는다. 대신 "외부 공개 / 서버 내부 전용" 수준만 표시한다.
{% endhint %}
