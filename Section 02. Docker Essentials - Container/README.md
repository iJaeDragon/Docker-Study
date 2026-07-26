# Docker 아키텍처 (요약)

## Docker 개요
- **2014년** Docker 1.0 발표, 컨테이너 기반 오픈소스 가상화 플랫폼
- 로컬 PC, AWS/Azure/GCP 등 어디서나 사용 가능
- OS뿐 아니라 백엔드/프론트엔드/DB/메시지 큐 등도 컨테이너화 가능
- 컨테이너 가상화 기술 중 사실상 표준 위치

## 기존 가상화 vs Docker 컨테이너
| 구분 | 서버 가상화 | Docker 컨테이너 |
|---|---|---|
| 방식 | 하이퍼바이저 → 게스트 OS 전체 가상화 | Docker 엔진 → 애플리케이션 단위 실행 |
| 장점 | 완전 독립적 OS/SW 사용 | 라이브러리/바이너리 공유, 성능 손실 거의 없음 |
| 단점 | 무겁고 느림 | - |
| 기동 시간 | 느림 | **1~2초, 늦어도 10초 내외** |

> Docker의 컨테이너 기술은 **리눅스 컨테이너 기술을 기반**으로 함

## Docker 엔진 구조
- **Docker Daemon**: 서버 역할
- **Docker Client**: 명령어 전달 (build, push, pull, run 등)
- 클라이언트-서버 통신: **REST API**
- 관리 대상: **네트워크, 컨테이너, 이미지, 볼륨**

## 핵심 용어
| 용어 | 의미 |
|---|---|
| **Docker** | 가상화 기술 자체 |
| **컨테이너(Container)** | 이미지를 실행시킨 인스턴스, 상태(state)를 가짐 |
| **이미지(Image)** | 컨테이너 실행에 필요한 파일/설정을 담은 stateless 템플릿 (비유: CD/DVD) |
| **컨테이너화** | 이미지를 컨테이너로 만드는 과정 (비유: 프로그램 설치) |
| **볼륨(Volume)** | 컨테이너가 사용하는 데이터 저장소 |
| **레지스트리(Registry)** | 이미지 저장소. 퍼블릭(Docker Hub) 또는 프라이빗 |

## 동작 흐름 (Client → Host → Registry)

<img width="770px" alt="docker-workflow" src="https://github.com/user-attachments/assets/778d5c33-031d-4e9b-928f-094fa04d939d" />

1. 클라이언트가 `build`, `push`, `pull`, `run` 등 명령어를 Docker 데몬에 전달
2. 데몬은 레지스트리에서 필요한 이미지를 다운로드 (pull)
3. 다운로드된 이미지를 기반으로 **컨테이너(인스턴스)를 기동**
4. 레지스트리에는 Linux, Redis, MariaDB, Java, Python 등 다양한 이미지 존재

## 한 줄 정리
> Docker = 클라이언트가 REST API로 데몬에 명령을 내려 레지스트리의 이미지를 받아와 컨테이너로 실행시키는 구조
