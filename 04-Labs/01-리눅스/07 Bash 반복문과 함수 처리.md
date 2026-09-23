# Bash 반복문과 함수 처리

## 1. 실습 목표

함수 호출, `for` 반복, 스크립트 위치 매개변수, `while + shift` 동작 흐름 검증

## 2. 실습 환경

- Ubuntu Linux
- Bash
- 작업 디렉터리 내 여러 로그 파일

## 3. 구성

```mermaid
flowchart TD
    A["loop_lab.sh 실행"] --> B["show_file 함수 정의"]
    B --> C["for file in *log"]
    C --> D["현재 파일명을 file에 저장"]
    D --> E["show_file \"$file\" 호출"]
    E --> F["함수 내부 $1에 파일명 전달"]
    F --> G["로그파일 확인 출력"]
    G --> C
    C -->|"반복 종료"| H["스크립트 $1 확인"]
    H --> I["서버 처리 출력"]
    I --> J["shift"]
    J --> H
    H -->|"$1 빈 값"| K["종료"]
```

## 4. 핵심 구현

```bash
#!/bin/bash

show_file() {
    echo "로그파일 확인: $1"
}

for file in *log
do
    show_file "$file"
done

while [ "$1" != "" ]
do
    echo "서버 처리: $1"
    shift
done
```

실행:

```bash
bash loop_lab.sh server1 server2 server3
```

## 5. 검증

실제 로그 파일 반복 출력:

```text
로그파일 확인: app.log
로그파일 확인: db.log
로그파일 확인: linux-lab-2026-09-02.log
로그파일 확인: linux-lab-2026-09-03.log
로그파일 확인: system.log
```

스크립트 실행 인수 순차 처리:

```text
서버 처리: server1
서버 처리: server2
서버 처리: server3
```

정상 확인:
- `*log` 파일 반복
- 함수 인수 → 함수 내부 `$1` 전달
- 스크립트 `$1`, `$2`, `$3` 순차 처리
- `shift`에 따른 위치 매개변수 이동
- `$1`이 빈 값이 되면 `while` 종료

## 6. 배운점

- 함수는 정의 시가 아니라 호출 시 실행
- 함수 종료 후 호출 위치로 복귀
- `show_file "$file"`은 함수 호출, `show_file=$file`은 변수 대입
- `for`의 `file`은 반복 변수이며 `in`은 `for` 문법
- 스크립트 위치 매개변수와 함수 내부 위치 매개변수 구분
- 변수 참조 자체 때문에 공백이 필요한 것은 아님
- `done` 이후 스크립트 처음이 아니라 `while` 조건으로 복귀
- `$(( ))`는 산술 확장, `$()`는 명령 치환
- `shift`로 위치 매개변수 순차 처리

## 관련 기록

