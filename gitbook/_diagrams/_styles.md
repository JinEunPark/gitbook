# 그림 범례 (모든 그림 공통)

모든 `.mmd` 는 아래 classDef 를 복사해서 쓴다. 바꾸면 전 그림을 같이 바꾼다.

| 요소 | 표현 | classDef / 문법 |
| --- | --- | --- |
| Docker 망 `rekit_edge` | 파란 영역 | `style <subgraph> fill:#DBEAFE,stroke:#3B82F6` |
| Docker 망 `rekit-backend_default` | 초록 영역 | `fill:#DCFCE7,stroke:#22C55E` |
| Docker 망 `seaweedfs_default` | 노란 영역 | `fill:#FEF3C7,stroke:#F59E0B` |
| Docker 망 `rekle-monitoring_default` | 보라 영역 | `fill:#EDE9FE,stroke:#8B5CF6` |
| 서버 밖 (인터넷·외부 서비스) | 회색 점선 테두리 | `classDef ext fill:#F1F5F9,stroke:#64748B,stroke-dasharray:5 4,color:#0F172A` |
| 외부 공개 지점 | 주황 채움 | `classDef public fill:#F97316,stroke:#C2410C,stroke-width:2px,color:#FFFFFF` |
| 서버 내부 전용 | 흰 채움 + 회색 실선 | `classDef internal fill:#FFFFFF,stroke:#475569,color:#0F172A` |
| 아직 없음 / 계획 | 흰 채움 + 옅은 점선, 옅은 글자 | `classDef planned fill:#FFFFFF,stroke:#94A3B8,stroke-dasharray:3 3,color:#64748B` |
| 문제 지점 | 빨간 테두리 | `classDef bad fill:#FEE2E2,stroke:#DC2626,stroke-width:2px,color:#7F1D1D` |
| 요청·데이터 흐름 | 실선 화살표 `-->` | |
| Prometheus 수집 (pull) | 점선 화살표 `-.->` | |
| 운영자 접근 | 굵은 화살표 `==>` | |

빌드: `node build.mjs` (scratchpad 의 mermaid-cli) → `../.gitbook/assets/intro-*.svg`
