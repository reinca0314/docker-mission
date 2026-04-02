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

### 3.1 디렉토리 및 파일 관리
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

### 3.2 권한 변경 실습
```bash
# 파일 권한 변경 로그
asas69876975$ touch permission_test.txt
asas69876975$ chmod 600 permission_test.txt
asas69876975$ ls -l permission_test.txt
-rw-------  1 asas69876975  staff  0  3 31 17:35 permission_test.txt

# 디렉토리 권한 변경 로그
asas69876975$ mkdir permission_dir
asas69876975$ chmod 700 permission_dir
asas69876975$ ls -ld permission_dir
drwx------  2 asas69876975  staff  64  3 31 17:36 permission_dir
```

### 3.3 hello-world 실행 결과 확인
```bash
asas69876975$ docker run hello-world
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

### 3.4 Docker 서비스 구축 및 검증
#### (1) Dockerfile 커스텀 이미지
```bash
FROM nginx:alpine
LABEL org.opencontainers.image.title="my-mission-nginx"
COPY app/ /usr/share/nginx/html/
```

#### (2) 포트 매핑 및 커스텀 결과 확인 (Port: 8080)
```bash
# 이미지 빌드 및 실행
asas69876975$ docker build -t my-custom-web .
asas69876975$ docker run -d -p 8080:80 --name mission-web my-custom-web

# 내부 파일 내용 확인 (curl 검증)
asas69876975$ curl http://localhost:8080
<h1>Hello My Mission!</h1>
```

#### (3) 바인드 마운트 실시간 반영 (Port: 8081)
```bash
# 바인드 마운트로 컨테이너 실행
asas69876975$ docker run -d -p 8081:80 --name dev-web -v $(pwd)/app:/usr/share/nginx/html nginx:alpine

# 호스트 파일 수정 후 반영 확인
asas69876975$ echo "Update Test" > app/index.html
asas69876975$ curl http://localhost:8081
Update Test
```

### 3.5 볼륨 영속성 검증
#### (1) 볼륨 생성 및 데이터 저장
```bash
# 1. 볼륨 생성 및 데이터 쓰기
asas69876975$ docker volume create my-mission-data
asas69876975$ docker run -d --name data-worker -v my-mission-data:/app/data alpine tail -f /dev/null
asas69876975$ docker exec data-worker sh -c "echo 'This data survives' > /app/data/test.txt"

# 2. 컨테이너 삭제 및 재생성 후 데이터 확인
asas69876975$ docker rm -f data-worker
asas69876975$ docker run -d --name data-checker -v my-mission-data:/app/data alpine tail -f /dev/null
asas69876975$ docker exec data-checker cat /app/data/test.txt
This data survives
```

---

## 4. Git 설정 및 저장소 연동
```bash
# Git 설정 확인
asas69876975$ git config --list
user.name=reinca0314
user.email=*** (개인정보 마스킹)
core.repositoryformatversion=0
...
```

---

## 5. 트러블 슈팅 기록
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

---

## 6. 핵심 개념 요약 정리
### (1) 파일 및 경로 제어 (기초)
절대 경로 vs 상대 경로 선택 기준

절대 경로: 최상위 루트(/)부터 전체 주소를 기입합니다. 실행 위치에 상관없이 항상 동일한 파일을 가리켜야 하는 설정 파일이나 시스템 스크립트 작성 시 사용합니다.

상대 경로: 현재 위치(./)를 기준으로 경로를 지정합니다. 프로젝트 폴더 자체가 이동해도 내부 파일 간의 연결 관계가 유지되어야 하는 소스 코드 개발 시 주로 사용합니다.

파일 권한 숫자 표기 규칙 (755, 644)
권한은 **읽기(4), 쓰기(2), 실행(1)**의 합으로 계산됩니다.
세 자리 숫자는 순서대로 **[소유자 / 그룹 / 기타 사용자]**를 의미합니다.
755: 소유자는 모든 권한(4+2+1), 나머지는 읽고 실행(4+1)만 가능합니다. 보통 실행이 필요한 디렉토리에 사용합니다.
644: 소유자는 읽고 쓰기(4+2), 나머지는 읽기(4)만 가능합니다. 일반적인 텍스트 파일에 사용합니다.

### (2) Docker 핵심 기술 원리
이미지 vs 컨테이너 (빌드/실행/변경)

이미지 (Build): 앱 실행에 필요한 모든 환경이 압축된 읽기 전용(Immutable) 설계도입니다.
컨테이너 (Run): 이미지를 기반으로 격리된 프로세스를 띄운 실행 상태입니다.
변경 (Change): 컨테이너 내부에서 파일을 바꿔도 원본 이미지는 변하지 않으며, 컨테이너를 삭제하면 변경 사항도 사라집니다.

포트 매핑이 필요한 이유
컨테이너는 호스트와 격리된 가상 네트워크를 사용하므로 외부에서 내부 포트로 직접 접속이 불가능합니다.
외부 요청을 컨테이너 내부로 전달하기 위해 호스트의 포트와 컨테이너 포트를 연결하는 통로를 만들어야 합니다.

호스트 포트 충돌 시 진단 순서
lsof -i :포트번호 명령어로 해당 포트를 점유 중인 프로세스 ID(PID)를 확인합니다.
불필요한 프로세스라면 kill -9 [PID]로 종료합니다.
종료가 어렵다면 Docker 실행 시 호스트 포트를 다른 번호(예: 8080 → 8082)로 변경하여 매핑합니다.

### (3) 데이터 및 구조 설계
디렉토리 구조 구성 기준

관심사 분리: 소스 코드(app/)와 인프라 설정(Dockerfile)을 분리하여 관리 효율성을 높였습니다.
재현성: 최상위 경로에 설정 파일과 문서(README.md)를 배치하여 누구나 즉시 환경을 파악하고 실행할 수 있도록 구성했습니다.
데이터 유실 방지 대안 (Docker Volume)
컨테이너는 삭제 시 내부 데이터가 모두 사라지는 휘발성 특징이 있습니다.
이를 방지하기 위해 데이터를 호스트의 특정 영역에 저장하는 **볼륨(Volume)**을 생성하여, 컨테이너가 교체되어도 데이터가 영구적으로 보존되도록 설계해야 합니다.

재현 가능한 설정 정리 방식
현재는 개별 docker run 명령어를 사용했으나, 실무에서는 docker-compose.yml 파일에 포트, 볼륨, 이미지 설정을 명시하여 단일 명령어로 전체 환경을 재현 가능하게 관리합니다.
