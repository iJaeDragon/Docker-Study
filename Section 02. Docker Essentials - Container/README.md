# Docker Essentials - Container

## CNCF Landscape
- **CNCF(Cloud Native Computing Foundation)**: 리눅스 재단 산하, 클라우드 네이티브 기술/서비스/용어를 정리한 단체
- Landscape에는 런타임, 스토리지, 네트워크, 컨테이너 런타임 등 파트별로 기술 정리
- 컨테이너 런타임 파트에 **Containerd, CRI-O** 등 핵심 엔진 소개
- 진하게 표시된 항목 = CNCF 공식 관리/졸업/상용 프로젝트

## Docker의 위치 (Stack Overflow 설문, 2023)
- "Other Tools" 항목에서 Docker가 **1위**
- NPM, PIP, Homebrew, Kubernetes 등과 함께 언급되지만 Docker가 압도적 1위 → 컨테이너 가상화 기술 중 Docker의 위상이 매우 높음

## Docker 개요
- **2014년** Docker 1.0 발표, 컨테이너 기반 오픈소스 가상화 플랫폼
- 로컬 PC, AWS/Azure/GCP 등 클라우드에서 사용 가능
- OS뿐 아니라 백엔드/프론트엔드/DB/메시지 큐 등 미들웨어도 컨테이너화 가능
- **2019년** Mirantis가 Docker Enterprise 부문 인수 → 상용 사용에는 일부 제약 있음 (개인/학습용 Docker Desktop은 문제없음)

## 기존 가상화 vs Docker 컨테이너 가상화
| 구분 | 서버 가상화 (VMWare, VirtualBox) | Docker 컨테이너 |
|---|---|---|
| 방식 | 호스트 위 하이퍼바이저 → 게스트 OS 전체 가상화 | 호스트 위 Docker 엔진 → 애플리케이션 단위 실행 |
| 장점 | 완전 독립적 OS/SW 사용 가능 | 라이브러리/바이너리 공유 가능, 성능 손실 거의 없음 |
| 단점 | 무겁고 느림 (오버헤드 큼) | - |
| 기동 시간 | 느림 | **1~2초, 늦어도 10초 내외** |

> 참고: Docker 이전에도 리눅스 자체 가상화 기술(KVM, 프로세스 분리형 컨테이너 등)이 있었으며, Docker의 컨테이너 기술은 **리눅스 컨테이너 기술을 기반**으로 함

### 알파인 리눅스
- 초경량 리눅스 이미지, 용량이 **MB 단위가 아니라 거의 KB~수 MB 수준**으로 매우 작음 (예: MP3 한 곡 정도의 크기)
- IoT 등 소형 디바이스에도 탑재 가능할 만큼 가벼움

## Docker 엔진 구조
- **Docker Daemon**: 서버 역할
- **Docker Client**: 명령어 전달 (build, push, pull, run 등)
- 클라이언트-서버 통신은 **REST API** 기반
- Docker 서버가 관리하는 대상: **네트워크, 컨테이너, 이미지, 볼륨**

## 핵심 용어 정리
| 용어 | 의미 |
|---|---|
| **Docker** | 가상화 기술 자체 |
| **컨테이너(Container)** | 이미지를 실행시킨 인스턴스(실체화된 상태), 상태(state)를 가짐 |
| **이미지(Image)** | 컨테이너 실행에 필요한 파일/설정을 담은 상태 없는(stateless) 템플릿 (비유: CD/DVD) |
| **컨테이너화** | 이미지를 실행 가능한 상태(컨테이너)로 만드는 과정 (비유: 프로그램 설치) |
| **볼륨(Volume)** | 컨테이너가 사용하는 데이터 저장소 |
| **레지스트리(Registry)** | 이미지 저장소. 퍼블릭(Docker Hub 등) 또는 프라이빗 레지스트리 사용 가능 |

## 동작 흐름 (Client → Host → Registry)

<img width="770px" alt="docker-workflow" src="https://github.com/user-attachments/assets/778d5c33-031d-4e9b-928f-094fa04d939d" />

1. 클라이언트가 `build`, `push`, `pull`, `run` 등 명령어를 Docker 데몬에 전달
2. 데몬은 레지스트리에서 필요한 이미지를 다운로드 (pull)
3. 다운로드된 이미지를 기반으로 **컨테이너(인스턴스)를 기동**
4. 레지스트리에는 Linux, Redis, MariaDB, Java, Python 등 다양한 이미지 존재

## 한 줄 정리
> Docker = 클라이언트가 REST API로 데몬에 명령을 내려 레지스트리의 이미지를 받아와 컨테이너로 실행시키는 구조. 컨테이너 가상화 기술 중 압도적 표준 위치를 차지하고 있음
