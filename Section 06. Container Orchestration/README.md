# 컨테이너 오케스트레이션 & Docker Swarm 개념 정리

---

## 1. 오케스트레이션 도구란

컨테이너의 **배포, 관리, 스케일링, 네트워크**에 관련된 작업을 자동화해주는 도구.
컨테이너가 한두 개일 땐 수동 관리가 가능하지만, 수백~수천 개 규모가 되면 컨테이너뿐 아니라 그 컨테이너들이 실행되는 **호스트 자체**도 스케줄링/관리해야 하는데 이걸 자동화해주는 것이 오케스트레이션 도구.

**전제 조건**: 오케스트레이션 도구(Kubernetes, Swarm 등)는 컨테이너 런타임(Docker) 없이는 동작 불가. 반드시 컨테이너 런타임이 먼저 설치되어 있어야 그 위에서 사용 가능.

### 오케스트레이션 도구가 하는 일

| 영역 | 내용 |
|---|---|
| 프로비저닝/배포 | 컨테이너 구성 및 일정(스케줄링) 조정 |
| 리소스 할당 | 스케일링 조건, CPU/메모리 등 리소스 관리 |
| 네트워크 | 로드 밸런싱, 트래픽 라우팅 |
| 모니터링 | 상태 확인 및 관리 |

---

## 2. 왜 Kubernetes 대신 Docker Swarm부터 배우는가

출처: [landscape.cncf.io](https://landscape.cncf.io) (CNCF, Cloud Native Computing Foundation — 리눅스 재단 산하, 클라우드 네이티브 기술/규격을 정의·소개하는 단체) → "Orchestration & Management" 카테고리에서 확인 가능.

| 비교 | Kubernetes | Docker Swarm |
|---|---|---|
| 업계 사용도 | 사실상 표준, 대부분의 기업이 채택 | 상대적으로 옛 기술, 사용 감소 추세 |
| 기능 | 매우 방대하고 강력 | Kubernetes 대비 기능 적음 |
| 설치 | 별도 설치/구성 필요 | Docker Desktop만 있으면 바로 사용 가능 |
|  |  | **오케스트레이션 개념을 가볍게 익히는 입문용**으로 먼저 사용 |

---

## 3. 핵심 개념: Host / Compose / Swarm / Service / Stack

| 개념 | 정의 |
|---|---|
| **Docker Host** | Docker(또는 Docker Desktop)가 설치되어 이미지/컨테이너를 실행할 수 있는 PC(또는 VM) 한 대 |
| **Docker Compose** | **단일 호스트** 안에서 여러 컨테이너를 하나의 파일로 묶어 관리하는 도구 (이전 학습 내용) |
| **Docker Swarm** | **여러 대의 Docker Host를 하나의 클러스터로 묶어** 관리하는 멀티 호스트 오케스트레이션 도구 |
| **Service** | Swarm 클러스터 위에, 여러 호스트에 걸쳐 배포되는 **컨테이너 묶음의 배포 단위**. `docker service` 명령어로 관리 |
| **Stack** | 여러 개의 **Service를 묶은 단위**. Compose의 멀티 호스트 버전이라고 이해하면 쉬움 |

> Swarm은 Docker Compose에서 쓰던 문법/개념을 상당 부분 그대로 재사용하기 때문에, Compose를 먼저 이해해두면 Swarm 학습이 훨씬 수월하다.

---

## 4. 단일 호스트(Compose) vs 멀티 호스트(Swarm) 구조

<img width="857" height="446" alt="image" src="https://github.com/user-attachments/assets/24b99eaa-d3f6-4dec-99af-7dbb4dca4846" />

- **왼쪽 (Compose)**: PC 1대(Windows/macOS 등) 위에 Docker Desktop 설치 = Docker Host 1개. 그 안에서 여러 컨테이너를 Compose 파일 하나로 관리. **단일 호스트 한계**를 벗어날 수 없음.
- **오른쪽 (Swarm)**: PC(혹은 VM) 여러 대 — 예: Windows 1대, macOS 1대, Linux 1대 — 각각에 Docker가 설치되어 있으면 각각이 Docker Host. 이 여러 Host를 하나의 **클러스터**로 묶는 것이 Swarm.
  - 클러스터 안에서 여러 호스트에 걸쳐 배포되는 컨테이너 묶음 = **Service**
  - 여러 Service를 묶은 것 = **Stack**

---

## 5. 요약 대응표

| 하고 싶은 것 | 사용할 도구 | 관리 단위 |
|---|---|---|
| PC 1대에서 여러 컨테이너 관리 | Docker Compose | 파일 1개 (`docker-compose.yml`) |
| PC(호스트) 여러 대를 묶어서 관리 | Docker Swarm | 클러스터 |
| 클러스터에서 컨테이너 묶음을 배포 | `docker service` | Service |
| 여러 Service를 한 번에 배포/관리 | `docker stack` | Stack |

---
