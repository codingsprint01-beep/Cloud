# RAID 장애 분석 및 복구

## 1. 문제 현상

`mdadm` 기반 RAID 0·1·5에서 장애 상태를 재현하고 RAID 레벨별 장애 허용 차이 확인.

RAID 5에서는 멤버 1개가 `FAULTY`인 상황에서 정상 디스크 1개까지 추가 제거되어 배열이 `inactive` (비활성) 상태로 전환.

```text
md5 : inactive
Not enough devices to start the array.
```

## 2. 영향 / 범위

- RAID 0: 디스크 1개 손실 시 배열 사용 불가
- RAID 1: 1개 디스크 장애 시 `degraded` 상태로 운영 가능
- RAID 5: 1개 디스크 장애까지 허용, 2개 사용 불가 시 정상 시작 불가
- 복구 대상: `/dev/md5`

## 3. 원인 분석

```bash
lsblk -f
cat /proc/mdstat
sudo mdadm --detail /dev/mdX
```

상태 표기:

```text
[UU]   → 2개 정상
[U_]   → 2개 중 1개 장애
[UUU]  → 3개 정상
[UU_]  → 3개 중 1개 장애
```

RAID 5에서 허용 범위를 넘는 2개 디스크가 사용 불가 상태가 되었고,
남은 멤버의 기존 `FAULTY` 메타데이터 때문에 일반 `assemble`로 시작되지 않는 상태 확인.

## 4. 원인

RAID 5의 장애 허용 범위는 1개 디스크이지만 실제 사용 불가 디스크가 2개로 증가해 배열이 `inactive` 상태로 전환.

## 5. 해결

```bash
sudo mdadm --assemble --force --run /dev/md5 /dev/sdg1 /dev/sdh1
sudo mdadm /dev/md5 --add /dev/sdf1
cat /proc/mdstat
```

복구 흐름:

```mermaid
flowchart LR
    A["inactive"] --> B["[3/2] [_UU]<br>2개 멤버로 배열 시작"]
    B --> C["recovery<br>복구 진행"]
    C --> D["[3/3] [UUU]<br>정상 복구"]
```

추가로 RAID 멤버 제거와 디스크 내부 RAID metadata 제거의 차이 확인.

```bash
sudo mdadm /dev/md1 --remove /dev/sdX1
sudo mdadm --zero-superblock /dev/sdX1
```

## 6. 검증

```bash
cat /proc/mdstat
sudo mdadm --detail /dev/md5
```

확인 결과:

```text
[3/3] [UUU]
State : clean
Active Devices : 3
Working Devices : 3
Failed Devices : 0
```

## 7. 재발 방지 / 배운점

- `/proc/mdstat`, `mdadm --detail` 기반 상태 분석
- RAID 0은 장애 허용과 기존 데이터 `rebuild` 불가
- RAID 1은 미러링 기반 1개 장애 허용
- RAID 5는 패리티 기반 1개 장애 허용
- `inactive`와 `degraded` 상태 구분
- RAID 배열 멤버 제거와 superblock 제거는 별도 작업
- RAID는 백업을 대체하지 않음

## 관련 기록

- [RAID 구성 및 검증](<../../04-Labs/01-리눅스/05 RAID 구성 및 검증.md>)
