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
