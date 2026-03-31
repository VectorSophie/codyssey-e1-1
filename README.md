# AI/SW 개발 워크스테이션 구축 미션 보고서

## 1. 실행 환경
- **OS**: Microsoft Windows 11
- **Shell**: PowerShell 5.1.
- **Docker**: version 29.2.1
- **Git**: git version 2.47

## 2. 체크리스트
- [x] 터미널 기본 조작 (mkdir, cd, pwd, ls, echo, cat, cp, mv, rm)
- [x] 권한 변경 실습 및 전/후 비교
- [x] Docker 설치 및 데몬 동작 점검
- [x] hello-world 및 ubuntu 컨테이너 실행 및 로그 확인
- [x] Dockerfile 기반 Nginx 커스텀 웹 서버 빌드 및 실행
- [x] 포트 매핑(8080:80) 및 브라우저/curl 접속 확인
- [x] Docker 볼륨을 이용한 데이터 영속성 검증
- [x] Git 사용자 정보 설정 및 로컬 저장소 연동

## 3. 결과

### 3.1 터미널 로그
```powershell
# 디렉토리 생성 및 이동
mkdir practice; cd practice; pwd
# 파일 생성 및 내용 확인
echo "Hello, Terminal!" > hello.txt; cat hello.txt
# 복사 및 이름 변경
cp hello.txt hello_copy.txt; mv hello_copy.txt renamed.txt
# 삭제 및 목록 확인
rm renamed.txt; ls
```

### 3.2 Docker 기본 점검 및 운영
```bash
# Docker 버전 및 상태 확인
$ docker --version
Docker version 29.2.1, build a5c7197

# hello-world 실행
$ docker run hello-world
Hello from Docker! (성공 메시지 확인)

# ubuntu 컨테이너 내부 명령 실행
$ docker run --name ubuntu-test ubuntu ls
bin, boot, dev, etc, home, lib...
```

### 3.3 커스텀 Dockerfile 및 포트 매핑
- **Dockerfile**:
```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
```
- **빌드 및 실행**:
```bash
$ docker build -t my-custom-nginx:1.0 .
$ docker run -d -p 8080:80 --name my-web-8080 my-custom-nginx:1.0
```
- **접속 검증**:
```powershell
$ Invoke-WebRequest -Uri http://localhost:8080 -UseBasicParsing
Content : <h1>Hello from My Custom Nginx!</h1>
```

### 3.4 Docker 볼륨 영속성 검증
```bash
# 볼륨 생성 및 데이터 쓰기
$ docker volume create my-data
$ docker run -d --name vol-test -v my-data:/app ubuntu sleep infinity
$ docker exec vol-test sh -c "echo 'Persistence Test' > /app/data.txt"

# 컨테이너 삭제 후 새로운 컨테이너에서 데이터 확인
$ docker rm -f vol-test
$ docker run -d --name vol-test-2 -v my-data:/app ubuntu sleep infinity
$ docker exec vol-test-2 cat /app/data.txt
# 결과: Persistence Test (데이터 유지 확인)
```

### 3.5 Git 설정 및 연동
```bash
$ git config --global user.name "AI-Assistant"
$ git config --global user.email "assistant@example.com"
$ git init
$ git add .
$ git commit -m "Initial commit: Workspace setup"
```

## 4. 트러블슈팅 (Troubleshooting)

### 뮨제 1: PowerShell에서의 `ls -la` 및 `export` 명령어 호환
- **문제**: Linux 스타일의 `ls -la` 및 `export` 명령어를 PowerShell에서 사용 시 에러 발생.
- **원인 가설**: PowerShell은 Alias로 `ls`를 `Get-ChildItem`으로 매핑하지만 `-la` 같은 Linux 전용 옵션은 인식하지 못함. `export`는 PowerShell에 존재하지 않는 명령어임.
- **해결/대안**: PowerShell 표준 명령어(`Get-ChildItem -Force`, `$env:VAR = value`)를 사용하거나, `ls` 명령어의 경우 매개변수 없이 사용함.

### 문제 2: Docker 컨테이너 한글 인코딩
- **문제**: `Invoke-WebRequest` 결과에서 HTML 태그 앞부분에 깨진 문자가 포함됨.
- **원인 가설**: PowerShell `Out-File` 기본 인코딩(UTF-16 LE)과 Nginx(UTF-8) 간의 인코딩 불일치.
- **해결/대안**: `Out-File` 시 `-Encoding utf8` 옵션을 명시적으로 부여하여 index.html을 생성함.