# SSH 접속 장애 분석 및 UFW 방화벽 설정 수정

## 1. 문제 현상

Ubuntu VM의 22/TCP 허용 규칙을 설정했지만 Windows 로컬 PC에서 SSH 접속 실패.

```text
ssh: connect to host 192.168.111.200 port 22: Connection timed out
```

## 2. 영향 / 범위

- 대상: Ubuntu Linux VM
- 접속 경로: Windows 로컬 PC → Ubuntu VM IPv4 SSH
- SSH 서비스와 22/TCP 포트 상태 확인 필요

## 3. 원인 분석

```bash
sudo systemctl status ssh        # SSH 서비스 상태
sudo ss -tulnp | grep ':22'      # 22/TCP LISTEN
sudo ufw status numbered         # UFW 규칙·순서
```

확인:
- SSH 접속은 IPv4 주소 사용
- UFW에 IPv4와 IPv6 규칙이 별도로 존재
- 차단 규칙이 실제 IPv4 SSH 트래픽에 먼저 적용되는 상태 확인

## 4. 원인

동일 22/TCP에 허용·차단 규칙이 충돌하고, IPv4 SSH 트래픽에 차단 규칙이 우선 적용되어 접속 실패.

## 5. 해결

```bash
sudo ufw status numbered
sudo ufw delete <차단규칙번호>
sudo ufw allow 22/tcp            # IPv4 SSH 허용
```

## 6. 검증

Windows 로컬 PC에서 동일 IPv4 주소로 SSH 재접속 후 정상 동작 확인.

## 7. 재발 방지 / 배운점

- SSH 장애 점검: `서비스 → LISTEN 포트 → 방화벽 → 실제 접속`
- UFW IPv4/IPv6 규칙 구분
- 사용하지 않는 방화벽 규칙 정리

## 관련 기록

- [SSH와 Nginx 서버 운영](<../../04-Labs/01-리눅스/04 SSH와 Nginx 서버 운영.md>)
