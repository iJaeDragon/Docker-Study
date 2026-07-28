# Docker Essentials - Image

## Docker 이미지란?
- 컨테이너를 만들기 위한 **읽기 전용 템플릿**
- 컨테이너 실행에 필요한 파일/설정을 담고 있지만 **상태(state)는 가지지 않음**
- 비유: 이미지 = CD/DVD(설치용 매체), 컨테이너 = 설치되어 실행 중인 상태
  → 컨테이너가 삭제돼도 원본 이미지는 유지되며, 이미지로 언제든 새 컨테이너를 다시 만들 수 있음

## 이미지 관련 용어
| 용어 | 의미 |
|---|---|
| **레지스트리(Registry)** | 이미지가 저장/제공되는 서비스 공간 (예: Docker Hub) |
| **레파지토리(Repository)** | 레지스트리 내에서 이미지별로 저장되는 개별 저장소 |
| **로컬 레지스트리** | 레지스트리에서 받아와 내 PC에 저장해둔 이미지 (원격 레지스트리와 구분) |
| **Docker CLI(클라이언트)** | 도커 데몬에 명령을 내리는 인터페이스. 이미지 다운로드/컨테이너 생성·중지/이미지 업로드 등을 여기서 수행 |

## 전체 흐름
```
Registry → (pull) → Local Registry(내 PC) → (run) → Container
```
- `docker run`: 이미지가 로컬에 없으면 레지스트리에서 다운로드(pull) 후 `create` + `start`까지 한번에 처리

## Dockerfile (이미지를 만드는 DSL)
- 이미지를 생성하기 위한 **전용 언어(Domain Specific Language, DSL)**
- 파일명은 기본적으로 `Dockerfile` (확장자 없음). 다른 이름 사용 가능 (예: `Dockerfile_test`)

### 기본 구조 예시
```dockerfile
FROM ubuntu:16.04
```
- **`FROM`절**: 이미지의 **베이스(base)**를 지정하는 필수 구문
  - 베이스 이미지 위에 필요한 것들을 추가로 설치해서 최종 이미지를 완성
  - 예: 자바 이미지를 만들 때 → JDK가 이미 설치된 이미지를 베이스로 쓰거나, 순수 리눅스 베이스에 JDK를 직접 설치
  - 예: DB 이미지를 만들 때 → OS 베이스 위에 MySQL/MariaDB 설치

## 이미지 빌드 명령어
```
docker build -t [이미지명]:[태그명] .
```
- `build`는 이미지 전용 명령이라 `docker image build` → `docker build`로 생략 가능
- `-t`, `--tag`: 만들 이미지의 이름:태그 지정 (태그 생략 시 자동 `latest`)
- 마지막 `.` : 현재 디렉토리에서 `Dockerfile`을 찾아 빌드하라는 의미 (오타 아님)
- 파일명이 `Dockerfile`이 아니면 별도 옵션으로 파일명을 명시해야 함 (안 하면 오류 발생)
- 빌드 과정에서 Dockerfile의 명령어 단위로 **레이어**가 나뉘어 생성됨

## 빌드 후 확인
```
docker image ls   (또는 docker images)
```
- 새로 만든 이미지가 목록에 있는지 확인

## 한 줄 정리
> Docker 이미지는 컨테이너를 만들기 위한 읽기 전용 템플릿이며, `Dockerfile`에 `FROM`(베이스)부터 필요한 설치 과정을 명시한 뒤 `docker build -t 이름:태그 .`로 빌드해서 만든다.
