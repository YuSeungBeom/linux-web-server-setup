# Linux 웹 서버 구축 및 운영

VirtualBox 환경의 Ubuntu 서버에서 Nginx 웹 서버를 설치하고 가상 호스트를 구성하여 여러 웹 사이트를 운영하는 실습 프로젝트입니다.

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
- VirtualBox
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

## Nginx 로그 확인

- `/var/log/nginx/access.log`에서 Firefox 접속 로그 확인
- 첫 접속 시 `200 OK` 확인
- 새로고침 시 `304 Not Modified` 확인
- `304`는 페이지가 변경되지 않아 브라우저 캐시를 사용하는 정상 응답

## 2026-09-22 업데이트 - SSH 원격 접속 설정 완료

### 구현 내용
- Ubuntu VM의 OpenSSH Server 서비스 실행 상태 확인
- UFW에서 `OpenSSH` 규칙을 통해 SSH 포트(`22/tcp`) 허용
- VirtualBox NAT 환경에서 SSH 포트 포워딩 설정
  - Host Port: `2222`
  - Guest Port: `22`
- Windows PowerShell에서 Ubuntu VM 원격 SSH 접속 성공

### SSH 접속 명령어
```powershell
ssh vboxuser@127.0.0.1 -p 2222
```

> `사용자명`에는 Ubuntu 터미널에서 `whoami` 명령으로 확인한 실제 계정명을 입력합니다.

### 접속 확인
- 원격 SSH 세션에서 `whoami` 명령으로 로그인 계정 확인
- `hostname` 명령으로 접속 대상 Ubuntu 서버 확인
- `pwd` 명령으로 현재 작업 디렉터리 확인

## 2026-09-22 - SSH 키 기반 인증 설정

### 구현 내용
- Windows PowerShell에서 ED25519 SSH 키 쌍 생성
- 공개키를 Ubuntu 서버의 `~/.ssh/authorized_keys`에 등록
- `.ssh` 디렉터리 권한 `700`, `authorized_keys` 파일 권한 `600` 적용
- Windows 개인키를 사용한 SSH 접속 성공 확인
- Ubuntu 계정 비밀번호 입력 없이 키 기반으로 원격 접속 확인

### 보안 원칙
- 개인키(`id_ed25519`)는 Windows PC에만 보관
- 공개키(`id_ed25519.pub`)만 Ubuntu 서버에 등록
- 개인키와 서버 비밀번호는 GitHub에 업로드하지 않음

## 2026-09-22 업데이트 - SSH 비밀번호 인증 비활성화 완료

### 구현 내용
- SSH 공개키 인증을 유지한 상태에서 비밀번호 기반 SSH 로그인을 비활성화
- 키보드 대화형(Keyboard-Interactive) 인증도 비활성화
- SSH 설정을 별도 Drop-in 설정 파일로 관리
- SSH 설정 문법 검사 후 서비스를 재시작하여 변경 사항 적용
- Windows PowerShell에서 공개키 로그인 및 비밀번호 인증 차단 테스트 완료

### SSH 보안 설정 파일

설정 파일:

```text
/etc/ssh/sshd_config.d/01-key-only.conf
```

설정 내용:

```text
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
```

- `PubkeyAuthentication yes`: 등록된 SSH 공개키를 이용한 인증 허용
- `PasswordAuthentication no`: 계정 비밀번호를 사용하는 SSH 인증 차단
- `KbdInteractiveAuthentication no`: 키보드 대화형 인증 경로 차단

Ubuntu에서는 `/etc/ssh/sshd_config.d/` 디렉터리의 Drop-in 설정 파일을 통해 SSH 설정을 분리하여 관리할 수 있다. 기존 기본 설정을 직접 수정하는 대신 별도 설정 파일을 사용해 키 기반 인증 정책을 적용하였다.

### 설정 적용

설정 파일 작성 후 문법 오류를 먼저 검사하였다.

```bash
sudo sshd -t
```

출력이 없으면 SSH 설정 문법에 문제가 없음을 의미한다.

문법 검사 후 SSH 서비스를 재시작하여 설정을 반영하였다.

```bash
sudo systemctl restart ssh
```

### 실제 적용 설정 확인

`sshd -T` 명령으로 현재 SSH 데몬에 실제 적용된 설정값을 확인하였다.

```bash
sudo sshd -T | grep -E 'passwordauthentication|pubkeyauthentication|kbdinteractiveauthentication'
```

확인 결과:

```text
pubkeyauthentication yes
passwordauthentication no
kbdinteractiveauthentication no
```

### SSH 접속 및 차단 테스트

Windows PowerShell에서 SSH 키 기반 접속을 확인하였다.

```powershell
ssh vboxuser@127.0.0.1 -p 2222
```

키 기반 로그인 성공 후, 비밀번호 인증만 사용하도록 강제하여 비밀번호 로그인이 차단되는지 확인하였다.

```powershell
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no vboxuser@127.0.0.1 -p 2222
```

다음과 같은 결과가 출력되면 비밀번호 인증 차단이 정상적으로 적용된 것이다.

```text
vboxuser@127.0.0.1: Permission denied (publickey).
```

`Permission denied (publickey)` 메시지는 오류가 아니라, 서버가 비밀번호 인증 요청을 거부하고 공개키 인증만 허용한다는 의미이다.

### 최종 SSH 보안 구성

| 항목 | 설정 상태 |
|---|---|
| SSH 서비스 포트 | Ubuntu VM `22/tcp` |
| Windows 접속 주소 | `127.0.0.1:2222` |
| VirtualBox NAT 포트 포워딩 | Host `2222` → Guest `22` |
| 공개키 인증 | 활성화 |
| 비밀번호 인증 | 비활성화 |
| 키보드 대화형 인증 | 비활성화 |
| 개인키 보관 위치 | Windows 호스트 PC |
| 서버에 저장되는 키 | 공개키만 저장 |
