# 🐳 Docker MSSQL 2017 세팅

Docker 컨테이너로 MSSQL Server 2017을 설치하고, 로컬 PC와 포트를 매핑하여 접속 테스트까지 완료하는 가이드입니다.

## 📋 요구사항

- MSSQL **2017** 버전
- 기본 포트(`1433`) ↔ 로컬 PC 기본 포트(`1433`) 매핑
- 기본 계정(`sa`) 설정
- `SELECT 1` 테스트로 정상 동작 확인

## 🔧 사전 준비

- Docker가 설치되어 있어야 합니다. ([Docker 설치 가이드](https://docs.docker.com/get-docker/))
- 로컬 PC의 `1433` 포트가 다른 프로세스에서 사용 중이지 않아야 합니다.

---

## 1. MSSQL 2017 이미지 다운로드

```bash
docker pull mcr.microsoft.com/mssql/server:2017-latest
```

Microsoft에서 공식 제공하는 MSSQL 2017 이미지를 받아옵니다.

---

## 2. 컨테이너 실행 (계정 / 포트 설정 포함)

```bash
docker run -e "ACCEPT_EULA=Y" ^
  -e "SA_PASSWORD=YourStrong@Passw0rd" ^
  -e "MSSQL_PID=Developer" ^
  -p 1433:1433 ^
  --name mssql2017 ^
  --hostname mssql2017 ^
  -v mssql2017-data:/var/opt/mssql ^
  -d mcr.microsoft.com/mssql/server:2017-latest
```

### 옵션 설명

| 옵션 | 설명 |
|---|---|
| `ACCEPT_EULA=Y` | MSSQL 라이선스 동의 (필수, 없으면 컨테이너가 뜨지 않음) |
| `SA_PASSWORD` | 기본 관리자 계정(`sa`)의 비밀번호. 대/소문자, 숫자, 특수문자 포함 8자 이상 필요 |
| `MSSQL_PID=Developer` | 무료로 쓸 수 있는 Developer 에디션 (개발/테스트 전용, 상업적 이용 불가) |
| `-p 1433:1433` | 로컬 PC `1433` 포트 ↔ 컨테이너 `1433` 포트(MSSQL 기본 포트) 매핑 |
| `--name mssql2017` | 컨테이너 이름 지정 (이후 명령어에서 참조하기 위함) |
| `-v mssql2017-data:/var/opt/mssql` | 데이터 볼륨 마운트 (컨테이너 삭제 후에도 데이터 유지) |
| `-d` | 백그라운드(detached) 실행 |

> ⚠️ **주의**: 예시 비밀번호(`YourStrong@Passw0rd`)는 반드시 본인 환경에 맞게 변경하세요.

---

## 3. 컨테이너 초기화 대기

```bash
echo "MSSQL 초기화 대기 중..."
sleep 15
docker logs mssql2017 --tail 20
```

MSSQL 컨테이너는 완전히 기동하는 데 보통 10~20초가 걸립니다. 로그에 아래와 같은 메시지가 보이면 준비가 완료된 것입니다.

```
SQL Server is now ready for client connections
```

메시지가 보이지 않으면 `sleep` 시간을 늘리고 로그를 다시 확인하세요.

---

## 4. 컨테이너 상태 확인

```bash
docker ps --filter "name=mssql2017"
```

- `STATUS`가 `Up`으로 표시되는지 확인
- `PORTS`에 `0.0.0.0:1433->1433/tcp`가 표시되는지 확인

---

## 5. 접속 테스트 (`SELECT 1`)

컨테이너 내부의 `sqlcmd` 도구로 접속을 테스트합니다.

```bash
docker exec -it mssql2017 /opt/mssql-tools/bin/sqlcmd ^
  -S localhost -U sa -P "YourStrong@Passw0rd" ^
  -Q "SELECT 1 AS TestResult"
```

정상 동작 시 아래와 같은 결과가 출력됩니다.

```
TestResult
-----------
          1

(1 rows affected)
```

> 💡 **참고**: 이미지 버전에 따라 `sqlcmd` 경로가 `/opt/mssql-tools18/bin/sqlcmd`일 수 있습니다. 경로가 다르면 아래 명령으로 먼저 찾아보세요.
> ```bash
> docker exec -it mssql2017 find / -iname "sqlcmd" 2>/dev/null
> ```

---

## 6. (선택) 로컬 PC에서 직접 접속 테스트

로컬에 `sqlcmd`나 DB 클라이언트(Azure Data Studio, DBeaver 등)가 설치되어 있다면 컨테이너에 들어가지 않고도 확인할 수 있습니다.

```bash
sqlcmd -S localhost,1433 -U sa -P "YourStrong@Passw0rd" -Q "SELECT 1 AS TestResult"
```

---

## 7. 컨테이너 이미지 저장 후 복제

### 7.1. 볼륨을 지정하지 않은 컨테이너 (-v mssql2017-data:/var/opt/mssql을 지정하지 않았을때)
기존 컨테이너를 이미지로 만들기 전에, 데이터 정합성을 위해 컨테이너를 잠깐 멈춰줍니다. (필수는 아니지만 안전합니다)

```bash
docker stop mssql2017
```

멈춘 컨테이너를 기반으로 새 이미지를 생성합니다.

```bash
docker commit mssql2017 mssql2017-clone-image:v1
```

`mssql2017` 컨테이너의 현재 상태(설치된 프로그램, DB 데이터, 설정 전부 포함)를 `mssql2017-clone-image:v1`이라는 이름의 이미지로 스냅샷 뜨는 명령입니다.


생성한 이미지가 잘 만들어졌는지 확인합니다.

```bash
docker images | grep mssql2017-clone-image
```
혹은 Windows 라면
```
docker images | findstr mssql2017-clone-image
```

이 이미지로 새로운 컨테이너를 실행합니다. 원본과 동일한 데이터를 그대로 가진 채로 뜹니다. 단, 같은 PC에서 두 컨테이너를 동시에 띄우는 것이므로 호스트 포트는 겹치지 않게 다른 포트(예: `1434`)로 매핑해야 합니다.

```bash
docker run ^
  -e "ACCEPT_EULA=Y" ^
  -p 1434:1433 ^
  --name mssql2017-clone ^
  --hostname mssql2017-clone ^
  -d mssql2017-clone-image:v1
```

| 옵션 | 설명 |
|---|---|
| `ACCEPT_EULA=Y` | MSSQL 라이선스 동의 (필수) |
| `-p 1434:1433` | 로컬 PC `1434` 포트 ↔ 컨테이너 `1433` 포트 매핑. 원본(`mssql2017`)이 이미 `1433`을 쓰고 있으므로 겹치지 않게 다른 포트 사용 |
| `--name mssql2017-clone` | 복제 컨테이너 이름 지정 |
| `-v mssql2017-clone-data:/var/opt/mssql` | 복제본 전용 데이터 볼륨. 컨테이너를 삭제해도 데이터 유지 |
| `-d mssql2017-clone-image:v1` | 앞서 commit으로 만든 이미지를 사용해서 컨테이너 실행 |

`SA_PASSWORD`는 이미 이미지 안에 설정이 저장되어 있으므로 다시 지정할 필요가 없습니다.

새 컨테이너가 정상 기동될 때까지 대기합니다.

```bash
sleep 15
docker logs mssql2017-clone --tail 20
```

새 컨테이너에 접속해서 원본 데이터가 그대로 복제되었는지 확인합니다.


만약 다른 PC나 서버로 이미지를 옮겨서 똑같이 복제하고 싶다면, 이미지를 파일로 내보내고(export) 옮긴 뒤 불러오는(load) 방식을 사용합니다.

이미지를 tar 파일로 저장합니다.

```bash
docker save -o mssql2017-clone-image.tar mssql2017-clone-image:v1
```

다른 PC로 `mssql2017-clone-image.tar` 파일을 복사한 뒤, 그곳에서 이미지를 불러옵니다.

```bash
docker load -i mssql2017-clone-image.tar
```

불러온 뒤에는 위와 동일하게 `docker run` 명령으로 컨테이너를 띄우면 됩니다.


이 방식은 스냅샷 방식이라 편리하지만, 운영 환경에서는 `docker commit` 대신 `.bak` 백업/복원이나 볼륨 자체를 복사하는 방식이 더 안전하고 표준적입니다.

### 7.2. `.bak` 백업/복원



---

## 🗑️ 컨테이너 정리 (필요 시)

```bash
docker stop mssql2017
docker rm mssql2017
docker volume rm mssql2017-data   # 데이터까지 완전히 삭제하려는 경우에만 실행
```
