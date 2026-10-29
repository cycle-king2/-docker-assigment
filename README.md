Docker 개발 워크스테이션 구축 과제

1. 프로젝트 개요

Windows 환경에서 WSL 2와 Docker Desktop을 이용해 컨테이너 개발 환경을 구축하고, NGINX 웹 서버 이미지 빌드, 포트 매핑, 바인드 마운트, Docker 볼륨의 데이터 영속성을 실습했습니다.

2. 개발 환경

항목

환경

운영체제

Windows

컨테이너 실행 환경

Docker Desktop / WSL 2

기본 WSL 배포판

Ubuntu

Docker Client

29.9.2

Docker Engine

29.9.2

Git

2.56.0.windows.4

프로젝트 경로

C:\docker-assigment

3. 실습 체크리스트

WSL 2 설치 및 상태 확인

Docker Desktop 설치 및 실행

Docker 버전 및 엔진 정보 확인

hello-world 컨테이너 실행

Ubuntu 컨테이너에서 Linux 명령어 실행

Docker 이미지 목록 확인

Dockerfile 작성 및 사용자 정의 이미지 빌드

NGINX 컨테이너 실행 및 포트 매핑

호스트 파일과 컨테이너 간 바인드 마운트

호스트 파일 수정 사항의 웹 페이지 반영 확인

Docker 볼륨 생성 및 데이터 영속성 검증

컨테이너 로그 및 리소스 사용량 확인

Git 설치 및 사용자 정보 설정

Git 저장소 초기화

4. Docker 기본 명령어 실습

Docker 버전 확인

docker --version
docker version
docker info

Docker Client와 Server 버전 및 엔진 정보를 확인했습니다.

hello-world 실행

docker run hello-world

Hello from Docker! 메시지를 확인해 Docker 컨테이너 실행을 검증했습니다.

Ubuntu 컨테이너 실습

docker run -it --name ubuntu-practice ubuntu bash

컨테이너 내부에서 ls, echo, cat /etc/os-release 명령어를 실행했습니다.

docker start ubuntu-practice
docker exec ubuntu-practice ls /
docker exec ubuntu-practice echo "exec practice"
docker ps -a

docker start: 기존 컨테이너를 시작합니다.

docker exec: 실행 중인 컨테이너에서 추가 명령어를 실행합니다.

docker ps -a: 종료된 컨테이너를 포함해 목록을 확인합니다.

5. 사용자 정의 NGINX 이미지

Dockerfile

FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80

이미지 빌드 및 컨테이너 실행

docker build -t my-nginx-site:v1 .
docker run -d --name my-nginx-container -p 8080:80 my-nginx-site:v1

브라우저 또는 curl로 http://localhost:8080에 접속해 사용자 정의 웹 페이지가 출력되는 것을 확인했습니다.

curl.exe http://localhost:8080

호스트의 8080 포트를 컨테이너의 80 포트에 연결했습니다.

6. 바인드 마운트

실행 명령어

docker run -d --name nginx-bind -p 8081:80 --mount "type=bind,source=C:\docker-assigment\bind-site,target=/usr/share/nginx/html,readonly" nginx:alpine

호스트의 bind-site 디렉터리를 컨테이너의 NGINX 웹 루트에 읽기 전용으로 연결했습니다.

검증

curl.exe http://localhost:8081

호스트 PC의 bind-site\index.html 파일에서 다음 문구를 수정했습니다.

수정 전: 수정 전 페이지입니다.

수정 후: 호스트에서 수정한 내용이 반영되었습니다.

컨테이너를 재생성하지 않고도 curl 응답에 수정된 문구가 나타나는 것을 확인했습니다.

7. Docker 볼륨과 데이터 영속성

볼륨 생성 및 데이터 기록

docker volume create site-data
docker run --name volume-writer --mount source=site-data,target=/data alpine sh -c "echo 'Docker volume persistence test' > /data/test.txt"

데이터 확인

docker run --rm --mount source=site-data,target=/data alpine cat /data/test.txt

Docker volume persistence test 출력으로 데이터 기록을 확인했습니다.

컨테이너 삭제 후 재검증

docker rm volume-writer
docker run --rm --mount source=site-data,target=/data alpine cat /data/test.txt
docker volume ls

volume-writer 컨테이너를 삭제한 뒤에도 테스트 파일의 내용이 유지되고 site-data 볼륨이 남아 있는 것을 확인했습니다.

8. 로그 및 리소스 모니터링

docker logs --tail 10 nginx-bind
docker stats --no-stream

NGINX의 요청 로그에서 HTTP 200 응답을 확인했습니다. docker stats --no-stream으로 실행 중인 컨테이너의 CPU, 메모리, 네트워크 I/O 등의 사용량을 확인했습니다.

9. Git 설정 및 저장소 초기화

git --version
git config --global user.name
git config --global user.email
git config --global init.defaultBranch
git init
git status

Git 설치 버전을 확인하고 사용자 정보 및 기본 브랜치를 설정했습니다. 프로젝트 폴더에서 Git 저장소를 초기화했으며, 이후 파일 추적 및 GitHub 원격 저장소 연결을 진행할 예정입니다.

10. 트러블슈팅

사례 1. Docker 명령어를 찾을 수 없음

증상: PowerShell에서 docker 명령어를 인식하지 못했습니다.

원인: Docker 실행 파일의 경로가 사용자 PATH 환경 변수에 등록되어 있지 않았습니다.

해결: Docker Desktop의 실행 파일 경로를 사용자 PATH에 추가한 뒤 PowerShell에서 버전을 확인했습니다.

사례 2. Docker 엔진 연결 오류

증상: Docker CLI가 dockerDesktopLinuxEngine 파이프에 연결할 수 없다는 오류가 발생했습니다.

원인: Docker CLI가 Docker Desktop의 Linux 엔진과 통신하지 못하는 상태였습니다.

해결: WSL 상태를 확인하고 Docker Desktop 엔진 연결을 복구한 뒤 docker version에서 Client와 Server 정보가 모두 출력되는 것을 확인했습니다.

사례 3. 바인드 마운트 웹 페이지 접속 실패

증상: localhost:8081에 접속할 수 없었습니다.

해결: Docker 엔진 연결을 복구한 후 바인드 마운트 컨테이너를 실행했습니다. 컨테이너 상태와 curl 응답을 확인했고, 호스트 파일 수정 사항이 웹 페이지에 반영되는 것까지 검증했습니다.

11. 향후 보완 사항

Git 파일 추적 및 첫 커밋

GitHub 원격 저장소 생성 및 연결

GitHub에 프로젝트 업로드

GitHub와 VS Code 연동 확인

실습 화면 캡처 및 증거 자료 첨부