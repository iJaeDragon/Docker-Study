```
                                  사용자
                                    │
                                    │ HTTP / HTTPS
                                    ▼
 ┌──────────────────────────────────────────────────────────────────────┐
 │ Nginx                                                                │
 │                                                                      │
 │ Web Server     : HTTP/HTTPS 요청을 받아주는 앞단 서버                  │
 │ Reverse Proxy  : 사용자의 요청을 WAS로 전달                            │  
 │ Load Balancer  : 여러 WAS 중 어디로 요청을 보낼지 결정                  │
 │ Static File    : CSS, JS, 이미지 등 정적 파일을 직접 제공할 수도 있음    │
 └───────────────────────────────┬───────────────────────────────────────┘
                                 │
                                 │ 요청 분배
                                 ▼
               ┌─────────────────┼─────────────────┐
               │                 │                 │
               ▼                 ▼                 ▼
 ┌────────────────────────────────────────────────────────────────────┐
 │   WAS Cluster : 서버를 여러 대 묶어서 하나의 서비스처럼 운영하는 구조  │
 │┌────────────────────┐ ┌────────────────────┐ ┌────────────────────┐│
 ││ Tomcat / WAS 1     │ │ Tomcat / WAS 2     │ │ Tomcat / WAS 3     ││
 ││                    │ │                    │ │                    ││
 ││ WAS                │ │ WAS                │ │ WAS                ││
 ││ : Spring 실행      │ │ : Spring 실행       │ │ : Spring 실행      ││
 ││                    │ │                    │ │                    ││
 ││ Controller         │ │ Controller         │ │ Controller         ││
 ││ Service            │ │ Service            │ │ Service            ││
 ││ Repository         │ │ Repository         │ │ Repository         ││
 │└────────────────────┘ └────────────────────┘ └────────────────────┘│
 └────────────────────────────────────────────────────────────────────┘
                                  │                      
                                  │                      
                                  │
                                  │
                                  │
                    ┌─────────────┴─────────────┐
                    │                           │
                    ▼                           ▼
          ┌────────────────────┐     ┌─────────────────────────────────────────────────────────────────────────┐
          │        DB          │     │    Docker Volume / Mount (컨테이너의 파일 저장 경로와 실제 저장공간 연결)   │
          │                    │     │                                                                         │
          │ DB                 │     │ Docker Volume : 컨테이너가 사용할 저장공간을 정의/관리                     │
          │ : 데이터 저장       │     │ Mount : Volume과 컨테이너의 경로를 연결                                   │
          │ 사용자              │     └─────────────────────────────────────────────────────────────────────────┘
          │ 게시글              │                           │
          │ 주문                │                          │     
          │ 파일 메타데이터      │                          │
          └────────────────────┘                           ▼
                                              ┌────────────────────────┐
                                              │    공용 파일 저장소     │
                                              │                        │
                                              │ 첨부파일 / 이미지 / 문서 │
                                              │ : 실제 파일 저장        │
                                              │                        │
                                              │ NFS / NAS / S3 등      │
                                              │                        │
                                              └────────────────────────┘
```

# 1. Web Server와 WAS 관계

### Web Server
##### 웹 서버는 보통 `nginx` 혹은 `webtob`를 사용함.
#### 역할
```
Nginx
├── HTTP / HTTPS 요청 수신
├── Reverse Proxy
├── Load Balancing
└── 정적 파일 제공
```
즉 사용자와 WAS 사이에서 요청을 받아주는 앞단 서버.


### WAS
##### 대표적으로 Tomcat을 사용
#### 역할
```
Tomcat
└── Spring MVC
    ├── Controller
    ├── Service
    └── Repository
```
Tomcat에서 Spring 애플리케이션을 실행하며, 실제 비즈니스 로직은 Spring 애플리케이션이 처리.

### 흐름
```
Nginx
→ 요청을 받아 전달

Tomcat
→ Spring 애플리케이션 실행

Spring
→ 실제 서비스 로직 처리
```
---

# 2. Nginx와 Tomcat 연결

Nginx는 Tomcat의 주소를 알고 있어야 함.

만약 Tomcat 주소가 `192.168.0.101:8080` 로 되어 있다고하면 Nginx에서 해당 주소로 요청을 전달할 수 있음.

### Nginx 설정:
```
server {
    listen 80;

    location / {
        proxy_pass http://192.168.0.101:8080;
    }
}
```

### 흐름
```
사용자
  │
  │ http://example.com
  ▼
Nginx : 80
  │
  │ proxy_pass
  ▼
Tomcat : 8080
```
---

# 3. WAS가 여러개 일때 Load Balancing

WAS가 여러개 일떄 Nginx Load Balancing을 통해 분산할 수 있음.
```
                    Nginx
                      │
               Load Balancing
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
       Tomcat 1    Tomcat 2    Tomcat 3
```
Nginx가 여러 WAS 중 하나를 선택해서 요청을 전달함.

Nginx에서는 `upstream`을 사용함.
```
upstream was_cluster {
    server 192.168.0.101:8080;
    server 192.168.0.102:8080;
    server 192.168.0.103:8080;
}

server {
    listen 80;

    location / {
        proxy_pass http://was_cluster;
    }
}
```
---

# 4. WAS Cluster

2대 이상의 웹 애플리케이션 서버를 하나로 묶어 마치 하나의 서버처럼 동작하게 만드는 기술임.

```
                         WAS Cluster
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   ┌────────────┐     ┌────────────┐     ┌────────────┐      │
│   │   WAS 1    │     │   WAS 2    │     │   WAS 3    │      │
│   │  Tomcat    │     │  Tomcat    │     │  Tomcat    │      │
│   │  Spring    │     │  Spring    │     │  Spring    │      │
│   └────────────┘     └────────────┘     └────────────┘      │
│                                                             │
│           같은 애플리케이션을 실행                            │
│           하나의 서비스처럼 운영                              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 전체 흐름

```text
사용자
  │
  ▼
Nginx
  │
  │ Load Balancing
  ▼
WAS Cluster
  │
  ├── WAS 1
  ├── WAS 2
  └── WAS 3
```
---

# 5. WAS Cluster와 DB 관계

여러 WAS가 하나의 DB를 사용할 수 있음.

```text
                         WAS Cluster
              ┌────────────┼────────────┐
              │            │            │
              ▼            ▼            ▼
            WAS 1        WAS 2        WAS 3
              │            │            │
              └────────────┼────────────┘
                           │
                           ▼
                    ┌──────────────┐
                    │      DB      │
                    │              │
                    │ 사용자       │
                    │ 게시글       │
                    │ 주문         │
                    │ 파일 정보    │
                    └──────────────┘
```
---

# 6. Docker Volume

Docker Volume은 **컨테이너가 사용할 데이터를 저장하기 위한 저장공간**임.

```text
Docker Volume
└── upload-data
```

Volume은 컨테이너와 분리되어 있기 때문에 컨테이너가 삭제되어도 Volume이 유지된다면 데이터도 유지할 수 있음.

```text
Container
└── /app/uploads
          │
          │ Mount
          ▼
Docker Volume
└── upload-data
```

---

# 7. Mount

Mount는 **Volume과 컨테이너 내부의 경로를 연결하는 것**.

```text
Docker Volume
└── upload-data
        │
        │ Mount
        ▼
Container
└── /app/uploads
```

간단하게 정리하면:

```text
Docker Volume
→ 저장공간

Mount
→ 저장공간과 컨테이너 경로를 연결
```

또는:

```text
Volume
= 어디에 데이터를 저장할 것인가

Mount
= 컨테이너의 어디에서 그 저장공간을 사용할 것인가
```

---

# 8. Docker Compose Volume / Mount 설정

예를 들어 다음과 같이 설정할 수 있음.

```yaml
services:

  was1:
    image: my-spring-app
    volumes:
      - upload-data:/app/uploads

  was2:
    image: my-spring-app
    volumes:
      - upload-data:/app/uploads

  was3:
    image: my-spring-app
    volumes:
      - upload-data:/app/uploads

volumes:
  upload-data:
```

여기서:

```yaml
volumes:
  upload-data:
```

는 Docker Volume을 정의하는 부분.

그리고:

```yaml
- upload-data:/app/uploads
```

는 해당 Volume을 컨테이너의 `/app/uploads`에 Mount하는 부분.

---

# 9. 여러 WAS가 하나의 Volume 사용

위와 같이 설정하면:

```text
                         upload-data
                         Docker Volume
                              │
              ┌───────────────┼───────────────┐
              │               │               │
            Mount           Mount           Mount
              │               │               │
              ▼               ▼               ▼
           WAS 1           WAS 2           WAS 3
       /app/uploads    /app/uploads    /app/uploads
```

WAS 1, WAS 2, WAS 3이 같은 Volume을 사용하는 구조가 됨.

예를 들어 WAS 1에서:

```text
/app/uploads/test.pdf
```

를 저장하면 같은 Volume을 사용하는 WAS 2와 WAS 3에서도 해당 파일을 볼 수 있음.

```text
WAS 1
/app/uploads/test.pdf
        │
        ▼
upload-data
        ▲
        │
        ├── WAS 2
        │   /app/uploads/test.pdf
        │
        └── WAS 3
            /app/uploads/test.pdf
```

---

# 10. Spring에서 파일 저장

Spring 애플리케이션에서는 Docker Volume을 직접 알 필요가 없음.

Spring은 단순히 컨테이너 내부 경로에 파일을 저장함.

예:

```java
String uploadPath = "/app/uploads";
```

파일 저장:

```java
Path path = Paths.get(
    uploadPath,
    file.getOriginalFilename()
);

Files.copy(
    file.getInputStream(),
    path,
    StandardCopyOption.REPLACE_EXISTING
);
```

Spring 입장에서는 단순히:

```text
/app/uploads/test.pdf
```

라는 경로에 파일을 저장하는 것.

---

# 11. Spring의 경로와 Docker Mount의 관계

Docker:

```yaml
volumes:
  - upload-data:/app/uploads
```

Spring:

```java
String uploadPath = "/app/uploads";
```

이 둘이 연결되어 있음.

```text
Spring
  │
  │ /app/uploads/test.pdf
  ▼
Container
  │
  │ Mount
  ▼
Docker Volume
  │
  │ upload-data
  ▼
실제 저장공간
```

따라서 Spring에서 `/app/uploads`에 파일을 저장하면 **자동으로 Docker Volume에 저장됨**.

Spring이 Docker Volume에 저장하는 것이 아니라:

```text
Spring
→ /app/uploads에 저장

Docker
→ /app/uploads를 Volume에 Mount

결과
→ Volume에 저장
```

이라는 구조임.

---

# 12. 경로를 잘못 입력하면 어떻게 되는가?

Docker 설정:

```yaml
volumes:
  - upload-data:/app/uploads
```

이렇게 되어 있는데 Spring에서:

```java
String uploadPath = "/app/uploadss";
```

라고 잘못 입력했다고 가정.

그러면:

```text
Container
│
├── /app/uploads
│      │
│      └── upload-data Volume
│
└── /app/uploadss
       │
       └── Volume과 연결되지 않은 경로
```

따라서:

```text
/app/uploads/test.pdf
```

에 저장하면:

```text
→ Docker Volume에 저장
```

되지만,

```text
/app/uploadss/test.pdf
```

에 저장하면:

```text
→ Docker Volume에 저장되지 않음
→ 일반적인 경우 컨테이너의 writable layer에 저장
```

될 수 있음.

---

# 13. Spring 설정파일로 경로 관리

파일 저장 경로를 Java 코드에 직접 작성하는 것보다 설정파일로 관리하는 것이 좋음.

## application.yml

```yaml
file:
  upload-path: /app/uploads
```

## Java

```java
@Value("${file.upload-path}")
private String uploadPath;
```

파일 저장:

```java
Path path = Paths.get(
    uploadPath,
    file.getOriginalFilename()
);
```

이렇게 하면:

```text
application.yml
      │
      │ /app/uploads
      ▼
Spring
      │
      ▼
Docker Mount
      │
      ▼
Docker Volume
```

구조가 됨.

---

# 14. 같은 서버에서 여러 WAS가 실행되는 경우

하나의 Docker Host에서 여러 WAS 컨테이너를 실행하는 경우:

```text
Docker Host
│
├── WAS 1
│   └── /app/uploads ──┐
│                     │
├── WAS 2              │
│   └── /app/uploads ──┼──→ upload-data Volume
│                     │
└── WAS 3              │
    └── /app/uploads ──┘
```

이 경우 같은 Volume을 여러 컨테이너가 사용할 수 있음.

---

# 15. 여러 서버로 구성된 클러스터에서는 주의

WAS Cluster가 여러 서버에 걸쳐 있는 경우에는 주의해야 함.

예:

```text
서버 A                         서버 B
┌─────────────────┐            ┌─────────────────┐
│ Docker          │            │ Docker          │
│                 │            │                 │
│ WAS 1           │            │ WAS 2           │
│                 │            │                 │
│ Volume A        │            │ Volume B        │
└─────────────────┘            └─────────────────┘
```

이 경우:

```text
Volume A ≠ Volume B
```

임.

따라서 WAS 1에서 Volume A에 저장한 파일을 WAS 2가 자동으로 볼 수 없음.

---
