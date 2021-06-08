> 09 서버 관리 

# 104 서버 네트워크 인터페이스와 IP 주소 목록 얻기 
```bash
#!/bin/sh

# ifconfig 명령어로 유효한 인터페이스 표시
# awk 명령어로 인터페이스 명과 IP 주소 추출
# LANG=C /sbin/ifconfig | \
# awk '/^[a-z]/ {print "[" $1 "]"} 
#     /inet / {split($2,arr,":"); print arr[2]}'

# ifconfig 아웃풋 포맷이 변경됨, 추가로 inet6도 표시
LANG=C /sbin/ifconfig | \
awk '/^[a-z]/ {print "[" $1 "]"} 
  /inet[6]? / {print $2}'
```
- `ifconfig` 명령어 출력 결과를 `awk`로 가공하기
- `LANG` 환경변수 : 
- awk 
  - split 함수 : 변수를 구분자로 나누어 저장함
    - `split([나눌 변수], [결과 저장 변수], 구분자)`

# 105 서버 에 작성된 사용자 계정 목록 얻기 
```bash
#!/bin/sh

# 사용자 계정 정보 파일
filename="/etc/passwd"

# 줄 첫 글자가 #인 주석은 제외 하고 cut 명령어로 첫번째 값을 표시
grep -v "^#" "$filename" | cut -f 1 -d ":"
```
- `/etc/passwd` : 사용자 목록 파일로 다음 순서로 값이 저장되어 있다
  - 사용자명 -> 암호 -> UID -> GID -> Comment -> home path -> login shell
  - `/etc/default/useradd` : 사용자가 추가 될때 기본 정보를 수정할 수 있다.
- `grep -v "[제외할 패턴]"` : 필요 없는 문자열을 제외할 때 `-v` 옵션을 사용한다.
