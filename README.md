# AI/SW 개발 워크스테이션 구축

---

## 1. 프로젝트 개요

본 프로젝트는 터미널, Docker, Git을 활용하여
재현 가능한 개발 환경을 구축하는 것을 목표로 한다.

개발 환경은 특정 개인 PC에 종속되지 않고,
누구나 동일한 방식으로 실행·배포·검증할 수 있어야 한다.

이를 위해 다음을 수행하였다:

* 터미널 기반 파일 및 권한 관리
* Docker 컨테이너 실행 및 관리
* Dockerfile 기반 커스텀 이미지 제작
* 포트 매핑, 바인드 마운트, 볼륨을 통한 실행 구조 검증
* Git을 통한 버전 관리 및 GitHub 연동

---

## 2. 실행 환경

* OS: macOS (OrbStack)
* Shell: zsh
* Docker: 28.5.2
* Git: 2.53.0

---

## 3. 프로젝트 디렉토리 구조

```text
Building_AI&SW_Development_Workstations/
├── README.md
├── images/
│   ├── port.png
│   ├── bind.png
│   └── volume.png
├── practice/
│   └── file.txt
└── my-web/
    ├── Dockerfile
    └── app/
        └── index.html
```

### 구조 설계 기준

* practice/: 터미널 및 권한 실습
* my-web/: Docker 기반 웹 서버 구성
* images/: 실행 결과 증거
* README.md: 전체 과정 문서화

---

## 4. 수행 체크리스트

* [x] 터미널 기본 조작
* [x] 파일 권한 실습
* [x] Docker 설치 및 점검
* [x] hello-world 실행
* [x] Ubuntu 컨테이너 실행
* [x] Dockerfile 기반 이미지 빌드
* [x] 포트 매핑 접속 확인
* [x] 바인드 마운트 검증
* [x] Docker 볼륨 영속성 검증
* [x] Git 설정
* [x] GitHub 연동

---

## 5. 터미널 조작 로그

```bash
$ cd ~/Desktop/Building_AI\&SW_Development_Workstations
$ mkdir practice
$ cd practice

$ touch file.txt
$ echo "hello" > file.txt
$ cat file.txt

$ cp file.txt copy.txt
$ mv copy.txt moved.txt
$ rm moved.txt
```

---

## 6. 파일 권한 실습

```bash
$ ls -l file.txt
$ chmod 644 file.txt
$ chmod 755 .
$ ls -la
```

### 권한 개념

* r: 읽기 / w: 쓰기 / x: 실행
* 644 = rw- r-- r--
* 755 = rwx r-x r-x

---

## 7. Docker 설치 및 점검

```bash
$ docker --version
$ docker info
```

---

## 8. Docker 운영 명령

```bash
$ docker images
$ docker ps
$ docker ps -a
$ docker stats --no-stream
```

```text
CONTAINER ID   IMAGE             STATUS
500944b02a77   ubuntu            Up
d24b945cc3c6   nginx:alpine      Up
e346f1aa3bb3   mission-web:1.0   Up
ad4dc53b48b3   ubuntu            Exited
a594660d9f0a   hello-world       Exited
```

---

## 9. hello-world 실행

```bash
$ docker run hello-world
```

```text
Hello from Docker!
```

---

## 10. Ubuntu 컨테이너 실행

```bash
$ docker run -it ubuntu bash
# echo "inside container"
# exit
```

---

## 11. Dockerfile 기반 웹 서버 구축

### Dockerfile

```dockerfile
FROM nginx:alpine
LABEL org.opencontainers.image.title="mission-web"
ENV APP_ENV=dev
COPY ./app /usr/share/nginx/html
```

### 커스텀 포인트

* nginx:alpine 기반 경량 웹 서버 사용
* LABEL로 이미지 메타데이터 정의
* ENV로 환경 변수 설정
* COPY로 정적 웹 파일 배포

---

### 빌드 및 실행

```bash
$ docker build -t mission-web:1.0 .
$ docker run -d -p 8080:80 --name mission-web-8080 mission-web:1.0
$ curl http://localhost:8080
```

---

## 12. 포트 매핑 검증

```bash
$ curl http://localhost:8080
```

포트 매핑은 컨테이너 내부 서비스에 외부에서 접근하기 위해 필요하며,
호스트 포트와 컨테이너 포트를 연결하여 웹 서버 접근을 가능하게 한다.

![포트매핑](./images/port.png)

---

## 13. 바인드 마운트 검증

```bash
$ docker run -d -p 8081:80 --name mission-bind-8081 \
-v "/Users/wlgjs060614351/Desktop/Building_AI&SW_Development_Workstations/my-web/app:/usr/share/nginx/html" \
nginx:alpine
```

```bash
$ curl http://localhost:8081
```

파일 수정 후:

```bash
$ curl http://localhost:8081
```

바인드 마운트는 호스트와 컨테이너를 연결하여
파일 변경이 즉시 반영되도록 한다.

![바인드마운트](./images/bind.png)

---

## 14. Docker 볼륨 영속성 검증

```bash
$ docker volume create mission-data

$ docker exec -it mission-volume-1 bash -lc "echo hello-volume > /data/test.txt && cat /data/test.txt"
hello-volume

$ docker rm -f mission-volume-1

$ docker run -d --name mission-volume-2 \
-v mission-data:/data ubuntu sleep infinity

$ docker exec -it mission-volume-2 bash -lc "cat /data/test.txt"
hello-volume
```

### 수행 과정 설명

1. 볼륨 생성

```bash
$ docker volume create mission-data
```

* 컨테이너와 독립적인 저장 공간 생성

2. 데이터 생성

```bash
$ docker exec -it mission-volume-1 bash -lc "echo hello-volume > /data/test.txt && cat /data/test.txt"
hello-volume
```

* `/data` 경로에 파일 생성 (볼륨과 연결됨)

3. 컨테이너 삭제

```bash
$ docker rm -f mission-volume-1
```

* 컨테이너는 삭제되지만 데이터는 유지됨

4. 새 컨테이너 실행

```bash
$ docker run -d --name mission-volume-2 \
-v mission-data:/data ubuntu sleep infinity
```

5. 데이터 확인

```bash
$ docker exec -it mission-volume-2 bash -lc "cat /data/test.txt"
hello-volume
```

### 핵심 개념

* Docker 볼륨은 컨테이너와 분리된 저장소
* 컨테이너 삭제 후에도 데이터 유지
* 여러 컨테이너에서 동일 데이터 사용 가능

![볼륨](./images/volume.png)

---

## 15. Docker 로그 확인

```bash
$ docker logs mission-web-8080
```

웹 요청이 정상적으로 처리되고 로그가 기록됨을 확인하였다.

---

## 16. 이미지와 컨테이너의 차이

| 구분 | 이미지    | 컨테이너    |
| -- | ------ | ------- |
| 개념 | 실행 템플릿 | 실행 인스턴스 |
| 상태 | 불변     | 변경 가능   |
| 역할 | 환경 정의  | 실제 실행   |

* 빌드: Dockerfile → 이미지
* 실행: 이미지 → 컨테이너
* 변경: 컨테이너 변경은 이미지에 반영되지 않음

---

## 17. 경로 개념

* 절대 경로: /Users/...
* 상대 경로: ./app

Docker에서는 절대 경로 사용이 안정적이다.

---

## 18. 재현 가능한 실행 방법

```bash
docker build -t mission-web:1.0 .
docker run -d -p 8080:80 mission-web:1.0
docker run -d -p 8081:80 -v "/Users/.../app:/usr/share/nginx/html" nginx:alpine
docker volume create mission-data
```

---

## 19. 포트 충돌 트러블슈팅

```bash
docker ps
lsof -i :8080
docker stop <container>
docker run -p 8081:80
```

---

## 20. 데이터 영속성 문제와 해결

컨테이너 삭제 시 데이터는 사라진다.
이를 해결하기 위해:

* 바인드 마운트: 실시간 반영
* Docker 볼륨: 데이터 유지

---

## 21. 트러블슈팅 사례

invalid mode 오류 발생
→ 원인: 경로에 ":" 포함
→ 해결: 폴더명 변경

---

## 22. Git 설정 및 GitHub 연동

```bash
$ git config --global user.name "이지헌"
$ git config --global user.email "wlgjs06061@naver.com"
$ git config --list
```

Git은 로컬 버전 관리, GitHub는 원격 협업 플랫폼이다.

---

## 23. 결론

Docker를 통해 실행 환경을 표준화할 수 있으며
재현 가능한 환경 구성이 협업의 핵심임을 확인하였다.
