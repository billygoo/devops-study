# 011 실행 시 변수값이 비어 있을 때 기본값을 정의해서 설정하기
```bash 
#!/bin/sh

cp largefile.tar.gz ${TMPDIR:=/tmp}
cd $TMPDIR
tar xzf largefile.tar.gz 

echo "Extract files to $TMPDIR."
``` 

- defalut 값 설명 
```bash
$ man bash | less -p:=
######
       ${parameter:-word}
              Use  Default  Values.  If parameter is unset or null, the expan-
              sion of word is substituted.  Otherwise, the value of  parameter
              is substituted.
       ${parameter:=word}
              Assign  Default  Values.   If  parameter  is  unset or null, the
              expansion of word is assigned to parameter.  The value of param-
              eter  is  then  substituted.   Positional parameters and special
              parameters may not be assigned to in this way.
       ${parameter:?word}
              Display Error if Null or Unset.  If parameter is null or  unset,
              the  expansion  of  word (or a message to that effect if word is
              not present) is written to the standard error and the shell,  if
              it is not interactive, exits.  Otherwise, the value of parameter
              is substituted.
       ${parameter:+word}
              Use Alternate Value.  If parameter is null or unset, nothing  is
              substituted, otherwise the expansion of word is substituted.
###### 

$ echo ${MINUS:-value}
value
$ echo ${MINUS}
# null value

$ echo ${EQUAL:=value}
value
$ echo ${EQUAL}
value
$ echo ${EQUAL:=new_value}
value

$ echo ${QUESTION:?There is no value. Please check} 
zsh: QUESTION: There is no value. Please check

$ echo ${PLUS:+1} 
# null value
$ PLUS=xxxx
$ echo ${PLUS:+1} 
1
```
  - `${parameter:-word}`: 값이 설정되어 있으면 그대로 사용하고, 설정 안되어 있으면 기본값으로 대체하나 값은 실행 종료 후 변수에 반영 하지 않음
  - `${parameter:=word}`: 값이 설정되어 있으면 그대로 사용하고, 설정 안되어 있으면 기본값으로 대체하나 값은 실행 종료 후 변수에 반영
  - `${parameter:?word}`: 값이 설정되어 있지 않으면 표준 에러에 word를 메시지로 출력하고 종료
  - `${parameter:+word}`: 값이 설정되어 있으면 word를 전달해 주고, 설정 안되어 있으면 null을 반환. 


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
- 변수 앞에 `local`을 붙여서 지역 변수를 선언할 수 있다. 
- 쉘 스크립트는 `기본적으로 변수를 모두 전역 변수로 취급` 한다.


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

- `expr` 명령어를 이용해 정규식 패턴으로 검색할 수 있다. 
- `expr` 명령어는 4가지 용도로 활용할 수 있다. 
  - 산술 연산 : +, -, *, /, %
  - 논리 연산 : `|`(or) , `&`(and)
  - 관계 연산 : =, >, >=, <=, <, !=
    - `expr "값1" < "값2"` -> 참이면 1, 거짓이면 0
  - 문자열 패턴 매칭 : `:`
    - `expr "문자열" : "Regexp 패턴"`


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

- `test expression` 명령어는 `[ expression ]`를 이용해 사용할 수 있다. 
- `man test`를 이용해 다양한 옵션을 알 수 있다. 


# 015 작은 따옴표 안에서 작은 따옴표 쓰기 
```bash 
#!/bin/sh 

price=100
str='It costs $'$price'? I can'\''t believe it!'
echo $str
```

- 큰 따옴표(`"`)는 문자열의 변수 값을 해석해 보여준다. 
- 작은 따옴표(`'`)는 문자열의 변수 값을 해석하지 않고, 전체 문자열이 그대로 보여준다.


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

- `.`(닷 명령어)로 외부 파일을 읽어들일 때는 소스 파일이 그대로 삽입된 것처럼 파일 내부 명령어가 실행된다. 
- 단 외부 파일에는 쉬뱅(`#!/bin/sh`)을 표시하지 않는다. 
- **Tip**
  -  파일이 없으면 오류가 발생하기 때문에 미리 파일 존재 여부를 검사하도록 다음과 같이 코드를 작성한다. 
  ```bash
  [ -f /etc/sysconfig/sshd ] && . /etc/sysconfig/sshd
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

- 문자열에 공백이 추가되어 있을 경우 `test` 명령어에 변수를 넣으면 배열 취급되어 오류가 발생할 수 있다. 
- 그래서 공백이 포함된 문자열을 배열로 취급하지 않는다면, 큰 따옴표(`"`)로 감싸서 처리해야 한다.


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

- `sed`를 이용하면 패턴에 매칭되는 값을 추출할 수 있다. 
- `eval`을 이용하면 문자열을 한번 더 처리해 명령어로 실행 할 수 있다. 
```bash
$ unset bar 
$ cmd="bar=foo" 
$ eval "$cmd" 
$ echo "$bar" 
foo
```
- 주의해야 할 것은 명령어를 실행할 수 있기 때문에 입력에 주의해야 한다. 
- 예를 들어 `rm -rf ~/*`와 같은 잘못된 명령어를 쓸 수 있기 때문에 프로그램을 작성할 때는 `echo`를 활용해 어떤 명령어가 실행되는지 확인한다. 

# 019 언더스코어 등을 포함한 문자열에서 변수를 명확하게 구분하기 
```bash
#!/bin/sh

today="20150125"
# 쉘 변수가 today가 확장됨
wc -l ${today}_log
```

- 쉘에서 변수를 선언할 때 `$변수명`을 사용하는데 이때 띄어쓰기 없이 바로 값을 붙이면 변수로 해설 될 수 있다. 
- 그래서 중괄호(`{}`)로 감싸주면 변수와 문자를 구분해 줄 수 있다. 
```bash
$ declare -a number={"zero", "one", "two"}
$ echo ${number[0]}
zero
$ echo $number[0]
zero[0]
```
- 배열 변수를 다룰 때는 중괄호를 반드시 써줘야 배열을 제대로 인식한다. 중괄호 없이 사용하면 배열 인덱스를 문자열로 인식한다. 


# 20 명령어 출력 결과를 파일명에 포함해서 그 파일명을 대상으로 명령어를 실행할 때 보기 쉽게 하기 
```bash
#!/bin/sh

err_count=$(grep -c "ERROR" /var/log/myapp/$(hostname).log)
echo "Error counts: $err_count"
```

- 명령어 치환(Command Subsitution)을 이용 방법은 다음과 같이 두 가지가 있다. 
  - `$(명령어)` -> `$(hostname)`
  - \`(그레이브)로 감싸기 -> \`hostname\`


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