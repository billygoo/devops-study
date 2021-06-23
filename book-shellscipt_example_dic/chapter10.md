> bash 

# 124 쉘 변수를 정수값 같은 속정으로 선언하기
```bash
#!/bin/bash

url_template="http://www.example.org/download/img_%03d.jpg"

# 카운터 변수 count를 정수형으로 선언
declare -i count=0

while [ $count -le 10 ]
do
  url=$(printf "$url_template" $count)
  curl -O "$url"

  # expr 명령어 없이 산술 계산 가능 
  count=count+1
done
```
- `declare` : 쉘 변수의 속성값을 설정하게 해준다. 용도는 다음과 같다. 
  - 변수 속성을 설정하기 
  - 현재 정의한 변수, 함수명을 표시하기 
  - 옵션
    - -a : 변수를 배열로
    - -A : 변수를 해시로(bash 4.0 이상)
    - -i : 변수를 정수로 
    - -r : 변수를 읽기 전용으로(like `readonly`)
    - -x : 변수를 환경 변수로(like `export`)
    - -f [함수명] : 현재 정의한 쉘 함수와 내용을 표현
    - -F [함수명] : 현재 정의한 쉘 함수 이름만 표시
    - -p [변수명] : 현재 정의한 변수명과 그 내용을 표시 
- 일반적으로 변수는 모두 문자열로 다루어 진다.
- bash에서 제공하는 산술 확산을 이용하면 expr 없이도 사용 가능함 

# 125 for 반복문을 간단히 작성하기 
```bash
#!/bin/bash

# bash 브레이스 확장 {}으로 IP 주소 목록 작성 
for ipaddr in 192.168.2.{1..5}
do
  ping -c 1 "$ipaddr" > /dev/null 2>&1

  if [ $? -eq 0 ]; then
    echo "[OK] Ping -> $ipaddr"
  else
    echo "[NG] Ping -> $ipaddr"
  fi
done
```
- bash 브레이스(`{}`) 확장 : {}로 감싼 문자열로 쉼표(`,`)로 문자를 나열하거나, 점 두 개(`..`)로 범위를 지정할 수 있음
  - a.{php,html} -> a.php a.html
  - a{1..3}.html -> a1.html a2.html a3.html

# 126 사칙 연산을 간단히 작성하기
```bash
#!/bin/bash

# 브레이스 확장으로 1에서 100까지 숫자 목록 생성 
for i in {1..100}
do
  echo $((i * 3)) > ${i}.txt
done
```
- bash 산술 확장 : `$((산술식))` 포맷으로 산술식을 계산할 수 있다. 
  - 산술 확장 사용시 변수의 $ 기호를 생략 가능하다. 
  - 단 위치 파라미터의 경우 $를 삭제할 수 없다. 

# 127 변수 내부 문자열을 n 번째부터 m번째까지 추출하기 
```bash
#!/bin/bash

while read id status
do
  if [ "${id:0:2}" = "AC" ]; then
    echo "$id $status"
  fi
done < "$1"
```
- bash 파라미터 확장 : substr 같이 필요한 문자열을 추출할 수 있음
  - `${변수명:오프셋:문자수}` : 오프셋 위치부터 지정한 문자수 만큼 추출
  - `${변수명:오프셋}` : 오프셋 위치부터 끝까지 모든 문자열 추출 
  - 파라미터 확장어 오프셋은 0부터 시작된다.

# 128 변수 내부 문자열 일부를 치환하기 
```bash
#!/bin/bash

# 조사할 명령어 얻기
command="$1"

# 인수 확인
if [ -z "$command" ]; then
  echo "에러: 조사 대상 명령어를 지정하세요." >&2
  exit 1
fi

# 환경변수 PATH의 값에서 :을 스페이스로 치환해서 for루프 실행
for dir in ${PATH//:/ }
do 
  if [ -f "${dir}/${command}" ]; then
    echo "${dir}/${command}"
  fi
done
```
- bash 파라미터 확장 : 문자열을 치환 할 수 있음
  - `${변수명/패턴/치환문자열}` : 마지막에 일치한 패턴만 치환된다.
  - `${변수명//패턴/치환문자열}` : 일치한 모든 패턴이 치환된다.

# 129 중간 파일 없이 명령어 출력을 파일처럼 다루기 
```bash
#!/bin/bash

dir1="/var/tmp/backup1"
dir2="/var/tmp/backup2"

# comm 명령어로 출력을 비교. 중간 파일을 만들지 않아도 프로세스 치환으로 처리 가능
common <(ls "$dir1") <(ls "$dir2")
```
- bash 프로세스 치환(Process Subsitution) : 명령어 입력과 출력을 FIFO와 파일 디스크립터를 나타내는 /dev/fd 이하의 디바이스 파일에 접속해서 실행하는 기능
  - `명령어1 <(명령어2)` or  `명령어1 >(명령어2)`
  - 명령어 치환 시 괄호 앞에 스페이스를 쓰면 안된다. 

# 130 파이프 처리로 각 명령어 종료 상태값 조사하기 
```bash
#!/bin/bash

# 다음과 같이 여러가지 명령어가 실행 된다. 
./script.sh | ./sort-data.sh | ./calc.sh > output.txt 

# 실행 결과를 저장 
pipe_status = ("${PIPESTATUS[@]}")

if [ "${pipe_status[1]}" -ne 0 ]; then
  echo "[ERROR] sort-data.sh에 실패했습니다." >&2
fi
```
- 기존 상태 변수(`$?`)에서는 마지막 실행 결과만 저장한다. 
- bash 내장 변수 `PIPESTATUS` : 파이프라인 처리의 모든 종료 상태값을 취득할 수 있다. 0부터 시작 
  - 어떤 명령어를 실행할 때마다 변하기 때문에 평가할 명령어 바로 뒤에 변수 값을 다른 변수에 복사해서 확인해야 한다.
  
# 131 간단한 메뉴를 표시해서 사용자가 선택할 수 있게 하기 
```bash
#!/bin/bash 

PS3='Menu: '

# 메뉴 표시 정의. 메뉴 각 항목은 in에 목록으로 지정 
# $item은 선택한 목록 문자열이, $REPLY에는 입력한 숫자가 대입됨 
select item in "list file" "current directory" "exit"
do
  case "$REPLAY" in 
    1)
      ls
      ;;
    2)
      pwd
      ;;
    3)
      exit
      ;;
    *)
      echo "Error: Unknown Command"
      ;;
  esac

  echo
done
```
- bash select문 : 간단한 메뉴를 만드는 기능 
```
PS3=Prompt 구문
select <변수명> in <리스트>
do
  <명령어>
done
```
- `PS3` : select문이 이용하는 bash 쉘 변수로 메뉴 프로프트로 표시
- `REPLY`: 입력한 숫자가 저장되는 변수

# 132 정수값으로 난수 얻기 
```bash
#!/bin/bash

ipaddr="192.168.2.1"
port=80

waittime$((RANDOM % 10 + 1))

nc -w 5 -zv $ipaddr $port
echo "Wait: $waittime sec."
sleep $waittime
nc -w 5 -zv $ipaddr $port
```
- `RAMDOM` : bash 쉘 변수로 0~32767 사이의 정수값 난수를 돌려준다. 
  - `sh`에서는 `/dev/uramdom` 값을 써서 난수를 조합해야 함 
