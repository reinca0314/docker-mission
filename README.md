# 🚀 개발 워크스테이션 구축 미션 결과보고서

## 1. 실행 환경
- **OS**: macOS (OrbStack 환경)
- **Shell**: zsh (Terminal)
- **Docker**: Docker version 28.5.2, build ecc6942
- **Git**: git version 2.53.0

---

## 2. 수행 체크리스트
- [x] 터미널 기본 조작 및 폴더 구성
- [x] 권한 변경 실습 (파일/디렉토리)
- [x] Docker 설치/점검 (OrbStack 활용)
- [x] hello-world 및 ubuntu 컨테이너 실습
- [x] Dockerfile 커스텀 빌드/실행
- [x] 포트 매핑 접속 증거 (8080, 8081 포트 활용)
- [x] 바인드 마운트 실시간 반영 검증
- [x] Docker 볼륨 데이터 영속성 증명
- [x] Git 사용자 설정 및 VSCode GitHub 연동 완료

---

## 3. 수행 로그 (Terminal Operations)

## 3.1 디렉토리 및 파일 관리
```bash
asas69876975$ pwd
/Users/asas69876975/my-mission

asas69876975$ mkdir app
asas69876975$ ls -la
total 16
drwxr-xr-x   4 asas69876975  staff   128  3 31 17:30 .
drwxr-xr-x  10 asas69876975  staff   320  3 31 17:25 ..
-rw-r--r--   1 asas69876975  staff    56  3 31 17:30 Dockerfile
drwxr-xr-x   3 asas69876975  staff    96  3 31 17:28 app
```

## 3.2 권한 변경 실습

### 파일 권한 변경 (644 -> 600)
```bash
asas69876975$ touch permission_test.txt
asas69876975$ ls -l permission_test.txt
-rw-r--r--  1 asas69876975  staff  0  3 31 17:35 permission_test.txt
asas69876975$ chmod 600 permission_test.txt
asas69876975$ ls -l permission_test.txt
-rw-------  1 asas69876975  staff  0  3 31 17:35 permission_test.txt
```

### 디렉토리 권한 변경 (755 -> 700)
```bash
asas69876975$ mkdir permission_dir
asas69876975$ chmod 700 permission_dir
asas69876975$ ls -ld permission_dir
drwx------  2 asas69876975  staff  64  3 31 17:36 permission_dir
```

## 3.3 Docker 서비스 구축 및 검증
### (1) Dockerfile 커스텀 이미지
```bash
FROM nginx:alpine
LABEL org.opencontainers.image.title="my-mission-nginx"
COPY app/ /usr/share/nginx/html/
```

## (2) 포트 매핑 및 커스텀 결과 확인 (Port: 8080)
### 1. 이미지 빌드 및 실행
```bash
asas69876975$ docker build -t my-custom-web .
asas69876975$ docker run -d -p 8080:80 --name mission-web my-custom-web
```

### 2. 접속 확인 (브라우저 스크린샷 대체 가능)
```bash
asas69876975$ curl http://localhost:8080
<h1>Hello My Mission!</h1>
```

## (3) 바인드 마운트 실시간 반영 (Port: 8081)
### 호스트 app 폴더와 컨테이너 경로 동기화
```bash
asas69876975$ docker run -d -p 8081:80 --name dev-web -v $(pwd)/app:/usr/share/nginx/html nginx:alpine
```

### (4) 호스트 index.html 수정 후 즉시 반영 확인
```bash
asas69876975$ echo "Update Test" > app/index.html
asas69876975$ curl http://localhost:8081
Update Test
```

## 3.4 볼륨 영속성 검증
### (1) 볼륨 생성 및 데이터 저장
```bash
asas69876975$ docker volume create my-mission-data
asas69876975$ docker run -d --name data-worker -v my-mission-data:/app/data alpine tail -f /dev/null
asas69876975$ docker exec data-worker sh -c "echo 'This data survives' > /app/data/test.txt"
```

### (2) 컨테이너 삭제 후 재생성하여 데이터 유지 확인
```bash
asas69876975$ docker rm -f data-worker
asas69876975$ docker run -d --name data-checker -v my-mission-data:/app/data alpine tail -f /dev/null
asas69876975$ docker exec data-checker cat /app/data/test.txt
This data survives
```

## Git 설정 확인
```bash
asas69876975$ git config --list
user.name=reinca0314
user.email=*** (개인정보 보호를 위한 마스킹)
core.repositoryformatversion=0
...
```

## 4. 트러블 슈팅 기록
🚩 Case 1: sudo 권한 제한 및 Docker 실행 문제

문제: 보안 정책상 sudo 권한 사용 제한으로 일반적인 Docker Desktop 설치 불가.
원인 가설: 시스템 루트 권한 없이 컨테이너를 구동할 수 있는 런타임 필요.
해결: OrbStack을 활용하여 사용자 권한 내에서 Docker 데몬을 정상 구동함.

🚩 Case 2: Shell 특수문자('!') 인식 오류

문제: 볼륨 데이터 입력 시 bash: !: event not found 에러 발생.
원인 가설: 큰따옴표 내의 느낌표(!)가 쉘의 History 이벤트 지시자로 인식됨.
해결: 텍스트에서 느낌표를 제거하거나 작은따옴표(' ')를 사용하여 리터럴 문자로 처리함.

🚩 Case 3: GitHub Push (403 Forbidden) 및 인증 실패

문제: git push 시 403 Forbidden 발생 및 비밀번호 입력(열쇠 모양) 시 불통.
원인 가설: GitHub의 ID/PW 기반 인증 중단 및 로컬 키체인 정보 충돌.
해결: VSCode의 GitHub 계정 연동 기능 및 PAT(Personal Access Token)을 사용하여 재인증 완료.
