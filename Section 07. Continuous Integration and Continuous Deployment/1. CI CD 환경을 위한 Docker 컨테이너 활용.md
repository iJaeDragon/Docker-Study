# CI/CD와 Docker의 역할

---

## 1. DevOps ≠ CI/CD (자주 헷갈리는 개념 구분)

| 용어 | 의미 |
|---|---|
| **DevOps** | 개발(Dev)과 운영(Ops) 환경을 지속적으로 연결시켜, 서비스를 계속 개선해나가는 **문화이자 과정** |
| **CI/CD** | 완성된 결과물을 **지속적으로 통합(CI)**하고 **지속적으로 배포(CD)**하는 **구체적인 실행 체계** |

> Cloud Native Architecture의 4대 구성요소: DevOps / CI-CD / 마이크로서비스 아키텍처 / (Docker) 컨테이너 가상화 기술. 이 강의는 그중 컨테이너 가상화(Docker)를 다루고 있고, 이번 섹션에서 CI/CD와 어떻게 맞물리는지를 본다.

---

## 2. DevOps 무한 루프 (Plan → Code → Build → Test → Deploy → Operate → Monitor → 다시 Plan)

| 단계 | 의미 | 대표 도구/개념 |
|---|---|---|
| Plan | 프로젝트/기능 계획 | 형상관리, 이슈(이벤트) 트래킹 |
| Code | 개발(구현) | Git 등 |
| Build | 컴파일 및 실행 가능한 형태로 변환 | Maven, Gradle (Java 기준) |
| Test | 단위 테스트 및 테스트 프레임워크 실행 | 각종 테스트 프레임워크 |
| Deploy | 배포 가능한 형태로 패키징 → 전달 | **Docker 이미지가 배포 단위** |
| Operate | 운영 | Vagrant(가상화 유틸리티), Ansible/Terraform(IaC) |
| Monitor | 지속적인 모니터링 | Nagios, Prometheus |

이 사이클을 계속 반복하는 것이 지속적 통합/배포(CI/CD)의 실질적인 의미.

**Docker의 위치**: 이론상 Build/Deploy/Operate 어디에나 쓰일 수 있지만, 특히 **Deploy(배포)와 Operate(운영)** 단계에서 핵심적으로 사용됨.

---

## 3. CI/CD 파이프라인과 Docker의 위치

<img width="867" height="374" alt="image" src="https://github.com/user-attachments/assets/6542433d-de0c-4fe0-b94d-673b8abb341e" />

### 흐름 요약

1. 개발자가 소스 코드를 **commit/push**(Git 등 형상관리 도구) → 이 순간부터 파이프라인이 트리거될 수 있음
2. **CI (Continuous Integration)**: 소스를 가져와 **빌드 → 테스트 → (필요시 SonarQube로 정적 분석/보안 취약점 점검) → 패키징**
   - 대표 도구: Jenkins, Travis CI, CircleCI (오픈소스로 많이 사용)
   - Docker를 쓴다면 이 단계의 **패키징 산출물 = Docker 이미지**
3. **이미지 레지스트리**: 만들어진 이미지를 저장 — Docker Hub(퍼블릭) / **Harbor**(프라이빗) / AWS ECR 등
4. **CD (Continuous Delivery/Deployment)**: 레지스트리의 이미지를 실제 서버(테스트/스테이징/운영)에 배포·실행
   - 컨테이너 런타임(Docker)이 실행을 담당하고, 규모가 커지면 **Kubernetes** 같은 오케스트레이션 도구가 스케줄링·스케일링·리소스 관리를 담당
5. **모니터링** 결과를 바탕으로 다시 Plan 단계로 — 무한 반복

---

## 4. CI vs CD 용어 정확히 구분하기

| 용어 | 의미 |
|---|---|
| **CI (Continuous Integration)** | 소스 코드 → 빌드 → 테스트 → 패키징까지의 **통합 과정** |
| **CD** | 배포 가능한 상태로 만들어 실제 서버/클라우드/인프라에 전달·실행 |
| ㄴ **Continuous Delivery** | 배포 과정 중 **사람의 개입(승인 등)이 일부 필요**한 경우 |
| ㄴ **Continuous Deployment** | **사람 개입 없이 완전 자동으로** 운영까지 배포되는 경우 |

> 실무에서는 보통 소스 → **테스트/스테이징(UAT)** 환경 → 검증 통과 시 → **운영(Production)** 순으로 승격시키며, 마지막 운영 단계로의 전환은 프로젝트 성숙도에 따라 Delivery(수동 승인)일 수도, Deployment(완전 자동)일 수도 있다.

> Jenkins는 CI 도구로 잘 알려져 있지만, 배포(CD) 작업까지 함께 수행할 수 있어 **CI와 CD 양쪽 모두**에 걸쳐 사용되는 경우가 많다.

---

## 5. CI/CD에서 자주 등장하는 도구 정리

| 카테고리 | 도구 예시 | 역할 |
|---|---|---|
| CI 툴 | Jenkins, Travis CI, CircleCI | 형상관리 소스 변경 감지 → 빌드/테스트/패키징 자동 실행 |
| 코드 품질/보안 검사 | SonarQube | 정적 분석, 보안 취약점, 코드 스멜 탐지 |
| 인프라 자동화(IaC) | Ansible, Terraform | 서버/네트워크/미들웨어를 코드/스크립트로 관리·배포 |
| 가상화 유틸리티 | Vagrant | 가상화 환경을 쉽게 구성해주는 도구 |
| 컨테이너 런타임 | **Docker** | 이미지 빌드 및 컨테이너 실행 |
| 이미지 레지스트리 | Docker Hub(퍼블릭), **Harbor**(프라이빗), AWS ECR | 이미지 저장 및 배포 대상에서 pull 가능하게 관리 |
| 오케스트레이션 | Kubernetes | 컨테이너의 스케줄링/스케일링/리소스 관리 자동화 |
| 모니터링 | Nagios, Prometheus | 운영 상태·지표 수집 및 이슈 파악 |

---

## 6. 레지스트리를 쓰는 이유 (복습)

- 이미지를 레지스트리에 등록해두면, **접근 권한이 있는 어디서든** 동일한 이미지를 가져와 처음 의도한 것과 동일한 환경을 그대로 재현할 수 있다.
- 스케일링(복제) 시에도 레지스트리에서 이미지를 가져와 여러 개로 손쉽게 확장 가능.
- 불필요해지면 삭제도 자유롭게 가능 — 이미지 생명주기 관리가 쉬워짐.

---

## 7. 최종 프로젝트(섹션 10) 아키텍처 미리보기

강의 후반부 최종 프로젝트에서 구축할 전체 파이프라인의 개략도.

<img width="772" height="326" alt="image" src="https://github.com/user-attachments/assets/a6493e94-5818-4375-947b-6bffe4bfc061" />

| 구성요소 | 역할 |
|---|---|
| **GitLab** | 소스 코드 저장(형상관리) |
| **Jenkins (DinD 방식으로 설치)** | OpenJDK 17 기반. CI(빌드/테스트/패키징) + CD 트리거 담당. `~/.jenkins`에 워크스페이스 생성, 프로젝트별 디렉토리 관리 |
| **Harbor** | 프라이빗 이미지 레지스트리. Jenkins가 빌드한 이미지를 여기에 push |
| **Kubernetes** | Harbor의 이미지를 가져와 실제로 배포·운영하는 오케스트레이션 도구 |
| **Argo CD** | Kubernetes에 무엇을 배포할지 **GitOps 방식**으로 관리. `argocd sync` 명령으로 GitLab에 정의된 배포 스펙을 Kubernetes에 지속적으로 동기화 |

> **DinD를 여기서도 사용하는 이유**: Docker Desktop은 기본적으로 도커 하나만 제공하는데, 최종 프로젝트에서는 Jenkins 등을 위한 별도의 컨테이너 환경이 필요하므로, Swarm 실습 때와 마찬가지로 **호스트 Docker 위에 컨테이너를 하나 더 올리고 그 안에 Jenkins용 Docker를 설치**하는 방식을 사용한다.

이 구성요소들(Harbor, Kubernetes, Argo CD, Jenkins, GitLab)은 이후 실습에서 하나씩 직접 설치/구성하게 된다.

---

## 8. 핵심 정리 (체크리스트)

- [ ] DevOps는 문화/과정, CI/CD는 그 안에서 쓰이는 구체적인 자동화 파이프라인
- [ ] Docker의 이미지 = CI 단계의 **패키징 산출물**, 레지스트리(Harbor 등)에 저장, Kubernetes가 이를 가져와 CD(배포/운영) 수행
- [ ] CI = 통합(빌드~패키징까지), CD = Delivery(사람 개입 일부) / Deployment(완전 자동)로 세분화
- [ ] Jenkins는 CI 전용이 아니라 CD 작업도 함께 수행 가능
- [ ] 최종 프로젝트 구성: GitLab → Jenkins(DinD) → Harbor → Argo CD(GitOps) → Kubernetes

---
