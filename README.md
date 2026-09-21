# Linux 웹 서버 구축 및 운영

## 2026-09-21 업데이트 - Nginx 가상 호스트 설정 완료

### 구현 내용
- Nginx 가상 호스트 (Virtual Host) 설정
- site1.example.com, site2.example.com 운영

### 트러블슈팅
- 심볼릭 링크 생성 실패 → 절대 경로로 다시 생성
- nginx -t 에러 → sites-enabled 폴더 확인
