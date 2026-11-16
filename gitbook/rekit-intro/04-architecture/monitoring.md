---
description: 숫자는 Prometheus 로 모이고, 로그는 아직 컨테이너 안에 머물며, 운영자는 HTTPS 와 SSH 두 길로만 들어온다.
---

# 모니터링

{% hint style="info" %}
**핵심 — 숫자는 같은 서버의 Prometheus 로 모여 알림과 대시보드가 되고, 로그는 아직 컨테이너 안에 머문다. 운영자가 들어오는 길은 HTTPS 와 SSH 두 가지뿐이다.**
{% endhint %}

## 메트릭 — 수집 → 규칙 → 알림 → 대시보드

```mermaid
flowchart LR
  classDef internal fill:#FFFFFF,stroke:#475569,color:#0F172A
  classDef planned fill:#FFFFFF,stroke:#94A3B8,stroke-dasharray:3 3,color:#64748B

  subgraph BACK["rekit-backend_default"]
    APP["rekle-app 메트릭<br/>워커 4개 값을 합산"]:::internal
  end
  subgraph SW["seaweedfs_default"]
    S["seaweedfs 메트릭"]:::internal
  end
  subgraph MON["rekle-monitoring_default"]
    EX["exporter 4종<br/>호스트 · 컨테이너 · DB · Redis"]:::internal
    P["Prometheus<br/>15초 간격 · 30일 보관"]:::internal
    RU["규칙<br/>알림 16 · 기록 8"]:::internal
    AM["Alertmanager<br/>묶기 · 중복 억제"]:::internal
    G["Grafana<br/>대시보드 3"]:::internal
  end
  SL["슬랙 (미연결)"]:::planned

  P -.-> APP
  P -.-> S
  P -.-> EX
  P --> RU --> AM --> SL
  G -->|"조회만"| P

  style BACK fill:#DCFCE7,stroke:#22C55E,color:#14532D
  style SW fill:#FEF3C7,stroke:#F59E0B,color:#713F12
  style MON fill:#EDE9FE,stroke:#8B5CF6,color:#4C1D95
```

_관측 스택을 앱과 다른 compose 프로젝트로 두고 앱 망에 "합류"하게 한 이유는, 앱을 배포할 때마다 관측 컨테이너가 다시 만들어지지 않게 하기 위해서다. Grafana 는 Prometheus 만 조회하므로 DB 비밀번호를 갖지 않는다._

| 구성 | 사실 |
| --- | --- |
| 수집 대상 | 8개 — rekle-app · node · cAdvisor · postgres · redis · seaweedfs · prometheus · alertmanager |
| 수집 간격 · 보관 | 15초 (cAdvisor 30초) · 30일 또는 10GB 중 먼저 닿는 쪽 |
| 규칙 | 알림 16개 · SLI 기록 규칙 8개 (SLI 5개 중 4개, 가동률은 `up` 으로 직접 계산) |
| 알림 묶기 | 같은 알림 이름·심각도끼리 묶음, critical 은 1시간마다 다시 알림 |
| 알림 억제 | 앱이 통째로 죽으면 앱 지표 경고는 숨김 · 같은 대상의 critical 이 있으면 warning 숨김 |
| 대시보드 | 3개 — 서비스 · 인프라 · 부하테스트 |
| 알림 전송 | `[미연결]` 슬랙 수신 주소가 아직 없어 판정만 한다 |

### 워커 4개의 숫자를 하나로

gunicorn 워커 4개는 메모리를 나눠 쓰지 않는다. 그대로 두면 수집할 때마다 다른 워커의 값이 잡혀 그래프가 톱니 모양이 된다. 그래서 워커마다 값을 파일에 기록하고, 메트릭을 내보낼 때 합산한다. 운영에서 이벤트 루프 지연 측정 횟수가 23,284 ≈ 4워커 × 약 5,800 으로 찍혀 합산이 동작함을 확인했다(11.10).

### 알림 16개

| 묶음 | 알림 |
| --- | --- |
| 앱 | RekleAppDown · RekleHigh5xxRatio · RekleEventLoopLagHigh · RekleDbPoolTimeouts |
| 결제 | ReklePaymentUnknown (1건이라도 즉시) · RekleTossLatencyHigh |
| DB · Redis | ReklePostgresDown · ReklePostgresDeadlocks · ReklePostgresConnectionsHigh · RekleRedisDown |
| 호스트 · 컨테이너 | HostDiskUsageHigh · HostDiskUsageCritical · ContainerRestarted |
| 사진 저장소 | SeaweedFSDown · SeaweedFSS3ErrorsHigh |
| 감시 자체 | MonitoringTargetDown |

_규칙은 `promtool test rules` 단위 테스트로 검증한다 (커밋 `422bbd3`)._

## 로그 — 지금과 계획

```mermaid
flowchart LR
  classDef internal fill:#FFFFFF,stroke:#475569,color:#0F172A
  classDef planned fill:#FFFFFF,stroke:#94A3B8,stroke-dasharray:3 3,color:#64748B
  classDef bad fill:#FEE2E2,stroke:#DC2626,stroke-width:2px,color:#7F1D1D

  subgraph NOW["지금"]
    A1["rekle-app<br/>JSON 로그 한 줄 → stdout"]:::internal --> J1["docker json-file<br/>20MB × 5개"]:::internal --> X1["컨테이너를 다시 만들면<br/>로그 파일도 삭제"]:::bad
  end
  subgraph PLAN["계획 · Loki + Alloy"]
    J2["docker json-file"]:::internal --> AL["Alloy<br/>수집"]:::planned --> LK["Loki<br/>30일 보관"]:::planned --> GR["Grafana<br/>request_id 로 검색"]:::planned
  end
  NOW ~~~ PLAN
```

_지금 로그는 컨테이너 디렉터리 안 파일에만 있어서, 배포로 컨테이너를 다시 만들면 함께 지워진다. 0.0.6 배포 직후 rekle-app 로그는 101줄만 남아 있었다. 계획은 로그를 컨테이너 밖 Loki 에 30일 보관하고, Grafana 에서 `request_id` 로 찾는 것이다._

| 항목 | 지금 | 계획 |
| --- | --- | --- |
| 형식 | structlog JSON 한 줄, `request_id` · `user_id` 자동 포함 | 같음 |
| 보관 | 컨테이너당 20MB × 5개, 재생성 때 삭제 | Loki 30일 |
| 로테이션 없는 컨테이너 | nginx · jenkins · seaweedfs · registry · certbot · rekle-web | 로테이션 추가 |
| 검색 | `docker logs <컨테이너> \| grep <id>` | Grafana 에서 ID 클릭 → 같은 ID 로그 |
| 라벨 | — | 컨테이너 · 서비스 · env 만. `request_id` 는 라벨로 쓰지 않는다 (값 종류가 무한해 색인이 커진다) |
| 오류 추적 | Sentry 코드만 있음, 키 없음 | Sentry 무료 플랜 연결 |

## 운영자 접근 경로

```mermaid
flowchart LR
  classDef ext fill:#F1F5F9,stroke:#64748B,stroke-dasharray:5 4,color:#0F172A
  classDef public fill:#FED7AA,stroke:#EA580C,stroke-width:3px,color:#7C2D12
  classDef internal fill:#FFFFFF,stroke:#475569,color:#0F172A

  OP["운영자 노트북"]:::ext
  N["nginx<br/>HTTPS 서브경로"]:::public
  T["SSH 터널<br/>서버 내부 전용 포트로"]:::internal
  G["Grafana<br/>로그인 필수 · 익명 차단"]:::internal
  J["Jenkins<br/>로그인 필수"]:::internal
  P["rekle-postgres"]:::internal
  PR["Prometheus · Alertmanager"]:::internal
  S["SeaweedFS 관리 화면"]:::internal

  OP ==> N
  N ==> G
  N ==> J
  OP ==> T
  T ==> P
  T ==> PR
  T ==> S
```

_로그인 화면이 있는 도구(Grafana · Jenkins)만 HTTPS 경로로 열고, 로그인 기능이 없는 화면(Prometheus · SeaweedFS 관리 화면)과 DB 는 SSH 터널로만 연다. 인증이 없는 화면을 인터넷에 두지 않는다는 기준이다._

| 도구 | 경로 | 인증 |
| --- | --- | --- |
| Grafana | nginx HTTPS 서브경로 | 관리자 로그인, 익명 접근 차단, 비밀번호 미설정 시 기동 거부 |
| Jenkins | nginx HTTPS 서브경로 | 로그인. 서버 Docker 를 다룰 수 있는 도구라 직접 포트는 닫았다 |
| PostgreSQL | SSH 터널 | DB 계정 |
| Prometheus · Alertmanager | SSH 터널 | 없음 → 그래서 터널로만 |
| SeaweedFS 관리 화면 | SSH 터널 | 없음 → 그래서 터널로만 |

{% hint style="warning" %}
**알려진 대가** — Grafana · Jenkins 가 쇼핑몰과 같은 주소(서브경로) 아래에 있어서, 두 도구에 XSS 가 있으면 쇼핑몰 쿠키와 저장소에 닿을 수 있다. 서브도메인 분리 또는 IP 제한을 장기 과제로 두었다.
{% endhint %}
