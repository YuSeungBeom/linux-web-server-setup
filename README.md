# Linux 웹 서버 구축 및 운영

VMware 환경의 Ubuntu 서버에서 Nginx 웹 서버를 설치하고 가상 호스트를 구성하여 여러 웹 사이트를 운영하는 실습 프로젝트입니다.

## 2026-09-21 업데이트 - Nginx 가상 호스트 설정 완료

### 구현 내용
- Nginx 웹 서버 설치 및 서비스 실행
- Nginx 가상 호스트 (Virtual Host) 설정
- 'site1.example.com', 'site2.example.com' 운영
-사이트별 독립적인 웹 루트 디렉터리 구성
  - '/var/www/site1/html'
  - '/var/www/site2/html'
- 'sites-available' 설정 파일과 sites-enabled 심볼릭 링크를 이용해 사이트를 활성화
- 'nginx -t Nginx'를 통한 설정 문법 검증
- 'Firefox에서 각 가상 호스트 접속 확인

### 환경
- VMware
- Ubuntu Linux
- Nginx
- Firefox

### 테스트 주소
- 'http://site1.example.com'
- 'http://site2.example.com'

> 위 도메인은 VMware 내부 실습 환경에서 /etc/hosts로 연결한 로컬 테스트용 도메인입니다.

### 트러블슈팅

- 심볼릭 링크 생성 실패
    - 원인: 'sites-enabled' 경로 또는 대상 설정 파일 경로가 올바르지 않음
    - 해결: '/etc/nginx/sites-available/site1' 및 'site2' 파일을 확인한 뒤 절대 경로로 심볼릭 링크를 다시 생성
      
- 'nginx -t' 설정 검사 에러
  - 원인: '/etc/nginx/sites-enabled/site1' 링크가 없거나 끊어진 상태
  - 해결: 'sites-available'의 설정 파일 존재 여부를 확인하고 'ln -s' 명령으로 링크를 재생성한 뒤 `sudo nginx -t` 재실행

- Let's Encrypt SSL 인증서 발급 실패
  - 원인: `site1.example.com`은 실습용 예시 도메인이고 VMware 내부 서버는 공개 DNS 및 공인 IP 환경이 아님
  - 해결: 현재는 HTTP 기반 로컬 실습을 유지하고 실제 도메인·공인 IP·외부 접근 환경을 구성한 후 HTTPS를 적용할 예정


## 2026-09-21 업데이트 - UFW 방화벽 설정 완료

### 구현 내용
- UFW(Uncomplicated Firewall) 방화벽 활성화
- 기본 정책 설정
  - Inbound: 기본 차단
  - Outbound: 기본 허용
- SSH 원격 관리용 `OpenSSH` 규칙 허용
- Nginx HTTP 웹 서비스용 `Nginx HTTP` 규칙 허용
- `sudo ufw status verbose`로 규칙 적용 상태 확인

### 확인 결과
- UFW 활성 상태에서 `http://site1.example.com` 접속 성공
- 허용된 서비스: SSH(22/tcp), HTTP(80/tcp)


## 2026-09-21 업데이트 - Nginx 운영 로그 확인

### 확인 내용
- `/var/log/nginx/access.log`에서 사이트 접속 요청 확인
- Firefox에서 `site1.example.com`, `site2.example.com` 접속 후 HTTP 상태 코드 `200` 확인
- `/var/log/nginx/error.log`에서 최근 오류 여부 확인
- `systemctl status nginx`로 Nginx 서비스 실행 상태 확인
