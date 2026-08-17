# Docker 컨테이너 백업 & 복원 (commit / save / load) 레퍼런스

---

## 1. 왜 필요한가 — 이미지 vs 컨테이너 복습

- **이미지**: 데이터를 저장할 수 없는 상태(불변, read-only 템플릿)
- **컨테이너**: 이미지를 실행해서 만든 **실체화된 상태** — 여기에 사용자 데이터, 설정 변경 등이 쌓인다

컨테이너를 계속 쓰다 보면 그 안에 쌓인 작업 내용을 **새로운 버전의 이미지로 보존**하고 싶을 때가 있다. 이때 쓰는 것이 `commit`. 그리고 그렇게 만든 이미지를 파일로 내보내 다른 곳(다른 PC, 다른 Docker Host)으로 옮기고 싶을 때 쓰는 것이 `save` / `load`.

---

## 2. 전체 흐름

<img width="858" height="300" alt="image" src="https://github.com/user-attachments/assets/851802e9-8e16-4390-b289-f48e6bbd2367" />

| 단계 | 명령어 | 입력 | 출력 |
|---|---|---|---|
| ① | `docker commit` | **컨테이너**(실행 중, 데이터 있는 상태) | 새 이미지 |
| ② | `docker save` | 이미지 | `.tar` 파일 |
| ③ | (파일 이동) | — | USB / 이메일 / 네트워크 드라이브 등으로 전달 |
| ④ | `docker load` | `.tar` 파일 | 이미지 (다른 Docker Host에 등록) |

---

## 3. 명령어 레퍼런스

| 명령어 | 문법 | 설명 |
|---|---|---|
| `docker commit` | `docker commit <컨테이너ID/이름> <이미지명>:<태그>` | 실행 중(또는 존재하는) **컨테이너**를 새 이미지로 저장. ⚠️ 이미지가 아니라 반드시 **컨테이너**를 대상으로 지정해야 함 |
| `docker save` | `docker save --output <파일명>.tar <이미지명>:<태그>` | 이미지를 tar 파일로 export |
| `docker load` | `docker load --input <파일명>.tar` | tar 파일을 다시 이미지로 import |
| `docker images` | `docker images` | 현재 보유한 이미지 목록 확인 (필터링: `docker images \| grep <키워드>`, Windows는 `findstr`) |
| `docker rmi` | `docker rmi <이미지ID>` | 이미지 삭제 |

---

## 4. 실습 예시

### 4.1 실행 중인 컨테이너를 새 이미지로 commit

```bash
# 현재 실행 중인 컨테이너 확인
docker ps

# manager 컨테이너의 현재 상태를 새 이미지로 커밋
docker commit manager <내계정>/docker-server:manager

# 잘 만들어졌는지 확인 (필터링)
docker images | grep docker-server
# Windows: docker images | findstr docker-server
```

> **흔한 실수**: `commit`의 대상은 **이미지가 아니라 컨테이너**다. 컨테이너 이름/ID를 정확히 지정해야 하며, 이미지 이름을 넣으면 오류가 난다.

**용량 주의**: 원본 베이스 이미지(예: 1.24GB)보다 커밋된 이미지가 더 커질 수 있다(예: 1.96GB). 그동안 컨테이너 안에서 작업(Swarm 클러스터 구성, 서비스 배포 등)한 내용이 레이어로 쌓이기 때문.

### 4.2 이미지를 tar 파일로 저장 (export)

```bash
docker save --output docker-server-manager.tar <내계정>/docker-server:manager
```

- 용량이 클수록(예: 2GB) 시간이 오래 걸림.
- 결과물인 `.tar` 파일은 USB, 이메일, 네트워크 파일 드라이브 등으로 자유롭게 전달 가능.

### 4.3 tar 파일을 다시 이미지로 복원 (import)

```bash
# (테스트를 위해) 기존 이미지를 먼저 삭제해도 됨
docker rmi <이미지ID>

# tar 파일로부터 이미지 복원
docker load --input docker-server-manager.tar

# 복원 확인
docker images | grep docker-server
```

`load` 완료 후 다시 이미지 목록에 나타나면 정상 복원된 것 — 이 이미지를 그대로 다른 Docker Host에서 `docker run`으로 사용할 수 있다.

---

## 5. 핵심 정리 (체크리스트)

- [ ] `commit`은 **컨테이너 → 이미지**, `save`/`load`는 **이미지 ↔ tar 파일**이라는 방향을 헷갈리지 말 것
- [ ] `commit` 대상은 반드시 컨테이너(ID/이름)이며, 이미지 이름을 넣지 않도록 주의
- [ ] commit으로 만든 이미지는 그동안의 작업 이력 때문에 베이스 이미지보다 용량이 커질 수 있음
- [ ] `save --output <파일>.tar <이미지>`로 내보내고, `load --input <파일>.tar`로 다시 불러온다
- [ ] tar 파일은 USB/이메일/네트워크 드라이브 등으로 자유롭게 옮겨 다른 Docker Host에서 그대로 사용 가능

---

## 오케스트레이션 섹션 마무리

이번 문서를 끝으로 Docker Swarm 오케스트레이션 시리즈(클러스터 구성 → Service → Stack → Rolling Update → Rollback → 이미지 백업/복원)가 마무리.
