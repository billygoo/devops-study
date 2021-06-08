> 제어 구문 예제

# 098 변수가 포함된 IP 주소 목록 파일을 읽어서 ping 명령어로 확인하기
```bash
#!/bin/sh

# 명령어 인수 확인
if [ -z "$1" ]; then 
  echo "세 번째 옥텟까지 IP 주소를 인수로 지정하요." >&2
  exit 1
fi

# 대상 IP를 외부 파일 ping_target.lst에서 
# %ADDR_HEAD% 부분을 치환해서 순서대로 얻기
for ipaddr in $(sed "s/%ADDR_HEAD/$1/" ping_target.lst)
do
  # ping 명령어 실행. 출력 결과는 필요없으므로 /dev/null로 리다이렉트
  ping -c 1 $ipaddr > /dev/null 2>&1

  # 종료 스테이터스로 성공 실패 
  if [ $? -eq 0 ]; then 
    echo "[Success] ping -> $ipaddr"
  else
    echo "[Failed ] ping -> $ipaddr"
  fi
done
```
- for 문에 명령어 치환(`$()`)을 이용해 목록으로 판별가능한 값을 대입해 사용할 수 있음

# 099 연속된 파일명을 가진 URL을 자동 생성해서 순서대로 내려받기 
```bash
#!/bin/sh

url_template=http://www.example.org/download/img_%03d.jpg

# seq 명령어로 연속 번호 생성
for i in $(seq 10)
do
  url=$(printf "$url_template" $i)
  curl -O "$url"
done
```
- `seq` : 연속된 숫자를 출력하는 유틸
  - `expr` 명령어를 연속된 숫자를 만들 수도 있다. 단, `seq` 명령어 보다 느리다.

# 100 강제 종료될 때까지 파일 내려받기를 반복해 통신 확인하기
```bash
#!/bin/sh

# 확인 대상 URL
url=http://192.168.22.1/webapl/check

# 무한 반복
while true
do
  # curl명령어에서 테스트 대상 URL 내려받기 
  curl -so /dev/null "$url"

  # curl 명령어 종료 상태 확인 
  if [ $? -eq 0 ]; then 
    echo "[check OK]"
  else
    echo "[check NG]"
  fi

  sleep 5
done
```
- 무한 반복 문 만들기 
  - `while [ 1 ]`
  - `while true`
  - `while :`

# 101 ID 컬럼을 "00001" 처럼 0으로 채운 CSV 파일에서 번호를 지정해서 값을 추출하기 
```bash
#!/bin/sh

# 추출 조건 등 정의 
match_id=1          # 추출할 ID
csvfile="data.csv"  # CSV 파일

if [ ! -f "$csvfile" ]; then
  echo "CSV 파일이 존재하지 않습니다: $csvfile" >&2
  exit 1
fi

while read line 
do
  id=$(echo $line | cut -f 1 -d ',')
  name=$(echo $line | cut -f 2 -d ',')

  if [ "$id" -eq "$match_id" ]; then
    echo "$name"
  fi
done < "$csvfile"
```
- `test`의 산술 비교를 위해서는 `-eq` 연산자를 써야만 한다. 
- `test`의 `=` 연산자를 사용하면 문자열 연산이 된다. 


# 102 스크립트를 수정해서 if문 안이 비더라도 에러가 발생하지 않게 하기
```bash
#!/bin/sh

# 데이터 파일 정의
datafile="/home/user1/myapp/sample.dat"

# 데이터 파일 존재 확인
if [ -f "$datafile" ]; then
  # 용도 변경으로 필요 없으므로 주석 처리 
  # ./myapp "$datafile"

  # 빈 if문을 위해 :(널 명령어 추가)
  :
else
  echo "데이터 파일이 존재하지 않습니다: $datafile" >&2
  exit 1
fi
```
- 쉘 스크립트에서는 `if` 문이 비어 있으면 에러가 발생한다.
- 비어 있는 `if` 문을 작성하기 위해 `:`(널 명령어)를 사용할 수 있다.


# 103 웹 서버에서 파일을 내려받아서 MD5 해시값 계산하기
```bash
#!/bin/sh

# 내려 받을 파일 URL 경로, 파일명 지정
url_path="http://www.example.org/"
filename="sample.dat"

# mac에서는 md5 명령어 활용 
curl -sO "${url_path}${filename}" && md5sum "$filename"

rm -f "$filename"
```
- `&&(AND 연산자)`를 이용해 앞에 명령어가 성공하면 뒤에 명령어를 실행한다. 
- `curl`
  - `-s` : 중간 상황을 표시하지 않음 
  - `-O` : 표준출력이 아닌 파일로 저장