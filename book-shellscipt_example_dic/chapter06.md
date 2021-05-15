# 068 ID가 적힌 목록 파일에서 ID에서 추출할 때 ID 끝 문자를 정렬하기 
```bash
#!/bin/sh

# 임시 파일 지정
tmpfile="sort.lst"

# 대상 ID 파일 확인 
if [ ! -f "$1" ]; then 
  echo "ID 목록 파일을 지정하세요." >&2
  exit 1;
fi

# ID 끝 문자 숫자로 목록 정렬 
rev "$1" | sort | rev > $tmpfile

# 정렬한 ID 목록으로 리포트 작성 
./report.sh $tmpfile

# 임시 파일 삭제 
rm -f $tmpfile
```

- `id.lst`
```text
PPX0_2
AN29_9
UIA5_3
BA06_7
QXD3_0
```

# 069 텍스트 파일에서 구분자를 지정해서 컬럼 추출하기 
```bash 
#!/bin/sh 

# 미리 설정하지 않으면 에러가 발생하는 환경 변수 정의 
envname="TMPVAR"

# env 명령어로 환경 변수 목록을 표시해서 cut 명령어로 
# * 첫 번째 값을 표시 [ -f 1 ]
# * 분리자는 : [ -d "=" ] 로 표시 
env | cut -f 1 -d "=" > env.lst 

# 확인할 환경 변수명이 env.lst와 같은지로 정의되어 있는지 확인
grep -q "^${envname}$" env.lst

if [ $? -eq 0 ]; then
  # 환경 변수가 설정되어 있으면 start.sh 실행 
  echo "환경 변수 $envname가 설정되어 있습니다."
  ./start.sh
else
  echo "환경 변수 $envname가 설정되어 있지 않습니다."
fi
```

# 070 파일 앞머리의 쉬뱅(shebang, #!/bin/sh 등)을 추출해서 스크립트에 따라 확장자 붙이기 
```bash
#!/bin.sh

# 대상 스크립트 파일이 있는지 확인
if [ ! -f "$1" ]; then 
  echo "지정한 파일을 찾지 못했습니다: $1" >&2
  exit 1
fi

# 파일 첫 줄 읽음
headline=$(head -n 1 "$1")

# 파일 첫 줄에 따라 확장자를 판정해서 부여함
case "$headline" in
  */bin/sh|*bash*)
    mv -v "$1" "${1}.sh"
    ;;
  *perl*)
    mv -v "$1" "${1}.pl"
    ;;
  *ruby*)
    mv -v "$1" "${1}.rb"
    ;;
  *)
    echo "Unknown Type: $1"
esac
```

# 071 입력 파일 해시값을 줄마다 추가해서 출력하기 
```bash
#!/bin/sh

# 해시값을 출력할 임시 파일을 초기화 
tmpfile="hash.txt"
: > $tmpfile

# 쉘 구분자를 줄바꿈만 인식하도록 변경
IFS='
'

# 지정한 텍스트 파일에서 한 줄씩 읽어들임
while read -r line
do
  # MD5 해시 취득
  # 명령어에 파일명이 따라오므로 첫 번째 컬럼만 추출
  echo -n "$line" | md5sum | awk '{print $1}' >> $tmpfile
done < $1

# 원본 텍스트 파일과 해시를 출력한 임시 파일을 쉼표로 연결해서 표시
paste -d, "$1" $tmpfile
```

# 072 CSV 파일에서 지정한 특정 레코드의 컬럼값 얻기 
- `data.csv`
```txt
0001,kim,45
0002,lee,312
0003,park,102
0004,Kang,3
0005,Seo,92
```

```bash
#!/bin/sh 

# CSV 파일 지정 
csvfile="data.csv"

# ID가 지정되지 않으면 종료
if [ -z "$1" ]; then 
  echo "ID를 지정하세요." >&2
  exit 1
fi

# CSV 파일이 존재하지 않으면 종료 
if [ ! -f "$csvfile" ]; then
  echo "CSV 파일이 존재하지 않습니다: $csvfile" >&2
  exit 1
fi

while read line
do
  # cut으로 컬럼 추출
  id=$(echo $line | cut -f 1 -d ',')
  name=$(echo $line | cut -f 2 -d ',')
  score=$(echo $line | cut -f 3 -d ',')

  # ID 컬럼이 인수로 지정한 ID와 일치하면 표시
  if [ "$1" = "$id" ]; then
    echo "$name"
  fi
done < $csvfile
```

# 073 CSV 파일에 ID 목록을 입력해서 대응하는 ID 컬럼값 얻기 
```bash 
#!/bin/sh

filecheck()
{
  if [ ! -f "$1" ]; then
    echo "ERROR: File $1 does not exist." >&2
    exit 1;
  fi
}

# CSV 파일명과 ID 목록 파일명을 지정해서 파일 존재 확인 
csvfile="data.csv"
idlistfile="$1"
filecheck "$csvfile"

filecheck "$idlistfile"

while IFS=, read id name score
do
  grep -xq "$id" "$idlistfile"
  if [ $? -eq 0 ]; then
    echo $name
  fi
done < "$csvfile"
```

# 074 숫자로 된 CSV 파일에서 평균값 계산하기 
```bash
#!/bin/sh

# CSV 파일이 존재하지 않으면 종료
if [ ! -f "$1" ]; then
  echo "대상 CSV 파일이 존재하지 않습니다: $1" >&2
  exit 1
fi

# 확장자를 제외한 파일명 취득
filename=${1%.*}

awk -F, '{sum += $3} END{print sum / NR}' "$1" > ${filename}.avg
```

# 075 숫자값(CSV 파일)에서 "*"를 써서 간단한 텍스트 그래프 출력하기 
```bash
#!/bin/sh

csvfile="data.csv"
GRAPH_WIDTH=50

markprint() {
  local i=0
  while [ $i -lt $1 ]
  do
    echo -n "*"
    i=$(expr $i + 1)
  done
}

# 자료에서 최대값 얻음. 역순 정렬해서 첫 줄 얻음
max=$(awk -F, '{print $3}' "$csvfile" | sort -nr | head -n 1)

# 자료가 모두 0이면 최대값을 1로 지정
if [ $max -eq 0 ]; then
  max=1
fi

# CSV 파일을 읽어서 값마다 그래프 출력 
while IFS=, read id name score
do
  markprint $(expr $GRAPH_WIDTH \* $score / $max)
  echo " [$name]"
done < $csvfile
```


