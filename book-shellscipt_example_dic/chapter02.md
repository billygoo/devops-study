# 011 실행 시 변수값이 비어 있을 때 기본값을 정의해서 설정하기
```bash 
#!/bin/sh

cp largefile.tar.gz ${TMPDIR:=/tmp}
cd $TMPDIR
tar xzf largefile.tar.gz 

echo "Extract files to $TMPDIR."
``` 


# 012 지역 변수를 함수 안에 정의해서 호출한 곳의 변수가 파괴되지 않게 하기 
```bash 
#!/bin/sh

DIR=/var/tmp

ls_home()
{
    #변수 DIR을 함수 내부 변수로 정의
    local DIR

    DIR=~/$1
    echo "directory: $DIR"
    ls $DIR
}

ls_home logdir

echo "directory: $DIR"
ls $DIR
```


# 013 HTML 파일에서 특정 속성값 얻기
```bash 
#!/bin/sh

quote="[\"']"
match="[^\"']*"

while read line
do
    href=$(expr "$line" : ".*href=${quote}\(${match}\)${quote}.*")
    if [ $? -eq 0 ]; then
        echo $href
    fi
done < index.html
```


# 014 값이 정수인지 확인해서 계산하기
```bash
#!/bin/sh

# 인수가 정수인지 확인 
test "$1" -eq 0 2> /dev/null

if [ $? -lt 2 ]; then 
    echo "Argument is Integer."
    expr 10 + $1
else
    echo "Argument is not Integer."
    exit 1
fi 
```


# 015 작은 따옴표 안에서 작은 따옴표 쓰기 
```bash 
#!/bin/sh 

price=100
str='It costs $'$price'? I can'\''t believe it!'
echo $str
```


# 016 변수나 함수를 외부 파일로 작성하기 
```bash 
#!/bin/sh 

. ./env.sh

nowtime
cp -i -v large-file.tar.gz "$WORK_DIR"
nowtime 
``` 

```bash
# env.sh 
WORK_DIR=/var/tmp/myapp/

nowtime() {
    date +%X
}
```


# 017 문장 속 공백문자를 포함한 문자열 변수를 인수로 쓰기
```bash
#!/bin/sh 

result="invalid value"

if [ "$result" = "invalid value" ]; then
    echo "ERROR: $result" 1>&2
    exit 1
fi
```


# 018 HTML 파일에서 태그 속에 적힌 주석을 추출해서 그대로 실행하기 
```bash
#!/bin/sh

filename="myapp.log"
eval $(sed -n "s/<code>\(.*\)<\/code>/\1/p" command.htm)
```

```html
<!--command.htm-->
<html>
<head><title>Code List</title></head>

<body>
<p>This is a sample code.</p>
<code>date; ls -l $filename</code>

</body>
</html>
```

# 019 언더스코어 등을 포함한 문자열에서 변수를 명확하게 구분하기 
```bash
#!/bin/sh

today="20150125"
# 쉘 변수가 today가 확장됨
wc -l ${today}_log
```


# 20 명령어 출력 결과를 파일명에 포함해서 그 파일명을 대상으로 명령어를 실행할 때 보기 쉽게 하기 
```bash
#!/bin/sh

err_count=$(grep -c "ERROR" /var/log/myapp($hostname).log)
echo "Error counts: $err_count"
```

# 021 미정의 변수를 에러로 처리해서 실수 방지하기
```bash
#!/bin/sh
set -u 

COPY_DIR=/myapp/work

# COPY_DIR이 아니라 COP_DIR이라고 실수 했다. 
cp myapp.log $COP_DIR
``` 

- `set -u`를 사용하면 스크립트 내부에서 정의되지 않은 변수를 참조하려고 할 때 에러가 발생해서 스크립트 실행을 막게 됩니다. 
  - 이 옵션 사용시 오류가 있기 직전의 스크립트는 실행이 되고, 오류가 있는 라인에서 멈추게 된다. 
  - 단, 동적으로 변수가 할당되는 경우에는 의도하지 않은 결과를 만들어 낼 수 있기 때문에 주의해서 써야한다


# 022 히어 도규먼트에서 변수 확장하지 않고 그대로 $str처럼 표시하기 
```bash
#!/bin/sh

# 이 변수는 확장되지 않으므로 실제로는 사용되지 않음
str="Dummy"

cat << 'EOT'
여기는 히어 도큐먼트 본체입니다. 
이 부분에 적힌 문자열은 명령어 표준 출력에 
직접 리다이렉트 됩니다. 

종료 문자열을 작은 따옴표 기호로 감싸면
$str이라고 적어도 변수 확장되지 않으며
`echo abc`도 명령어로 치환되지 않습니다. 
EOT
```

- 히어 도큐먼트(Here Document, a.k.a. heredoc)는 멀티 라인 문자열을 스크립트 내부 명령어 표준 입력으로 사용하는 기능입니다. 
- 종료 문자열은 어떤 문자든 상관이 없으나 문자열에 포함되어 있으면 오류가 발생할 수 있어서, `EOF`에서 언더바를 사용해서 `__EOF__`와 같이 사용하기도 합니다. 
- heredoc에는 명령어 치환이 되는데 작은 따옴표(`'`)를 사용하면 명령어 치환을 막을 수 있습니다. 
- 작은 따옴푤르 사용하지 않고 명령어 치환을 막으려면 문자열 내부에 역슬래쉬(`\`)를 사용하면 됩니다.  
- 같이 읽어보기
  - https://www.baeldung.com/linux/heredoc-herestring