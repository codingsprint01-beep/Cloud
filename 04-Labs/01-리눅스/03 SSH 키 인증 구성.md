# SSH 키 인증 구성

## 1. 실습 목표

SSH key pair (키 쌍) 생성, 서버 공개키 등록, 키 기반 원격 접속과 반복 접속 설정 검증

## 2. 실습 환경

- Ubuntu Linux VM
- Windows 로컬 PC
- OpenSSH

## 3. 구성

```mermaid
flowchart LR
    C["클라이언트<br/>개인키 보관"] -->|"공개키 생성·등록"| S["Ubuntu SSH 서버<br/>authorized_keys"]
    C -->|"개인키로 인증"| S
```

## 4. 핵심 구현

```bash
ssh-keygen                                      # SSH 키 쌍 생성
ssh-copy-id -i ~/.ssh/<공개키> <사용자>@<서버IP> # Linux 환경에서 공개키 등록
ssh -i ~/.ssh/<개인키> <사용자>@<서버IP>         # 개인키 지정 접속
```

Windows에서는 공개키 내용을 SSH 표준 입력으로 전달해 서버의 `~/.ssh/authorized_keys`에 추가하는 방식 사용.

클라이언트 `~/.ssh/config`:

```text
Host <별칭>
    HostName <서버IP>
    User <사용자명>
    IdentityFile <개인키 경로>
    IdentitiesOnly yes
```

이후:

```bash
ssh <별칭>
```

서버 `/etc/ssh/sshd_config`의 키·비밀번호 인증 관련 설정도 확인.

## 5. 검증

- 키 쌍 생성 확인
- 서버 `authorized_keys`에 공개키 등록
- 개인키 지정 SSH 접속 성공
- `~/.ssh/config` 별칭 접속 성공
- 개인키 passphrase (개인키 보호 암호) 설정 시 접속 과정에서 입력 확인

## 6. Troubleshooting

- Windows에서 SSH 설정 파일 이름 오류로 클라이언트 설정 미적용
- 정확한 위치와 이름이 `~/.ssh/config`임을 확인
- `sshd_config`의 주석 상태와 실제 적용 설정의 차이 확인

## 7. 배운점

- SSH 키 인증: 클라이언트 개인키 보관 + 서버 공개키 등록
- 개인키 passphrase와 서버 계정 비밀번호의 목적 구분
- `ssh-copy-id`: 공개키 등록
- `ssh -i`: 개인키 지정
- `~/.ssh/config`: 반복 접속 정보 관리
- `~/.ssh`와 `authorized_keys` 권한 설정의 중요성 확인

## 관련 기록

