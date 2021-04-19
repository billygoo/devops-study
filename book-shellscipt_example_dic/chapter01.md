# 001 명령어 옵션 처리하기 

```bash
#!/bin/sh

# -a 옵션이 있는지 플래그 변수 a_flag와 
# -p 옵션의 구분자를 정의하기
a_flag=0
separator=""
while getopts "ap:" option
do
    case $option in 
        a)
            a_flag=1
            ;;
        p)
            separator="$OPTARG"
            ;;
        \?)
            echo "Usage: getopt.sh [-a] [-p separator] target_dir" 1>&2
            exit 1
            ;;
    esac
done 

# 옵션 지정을 위치 파라미터에서 삭제하기
shift $(expr $OPTIND - 1)
path="$1"

# -a 옵션이 지정되었는지 쉘 변수 a_flag값으로 판단하기
if [ $a_flag -eq 1 ]; then
    ls -a -- "$path"
else
    ls -- "$path"
fi

if [ -n "$separator" ]; then
    echo "$separator"
fi
```

- `getopts`를 쓸 때 `while`문 조건식으로 `getopts`쓰고 `while` 루프 내부의 `case` 문으로 판단한다.
- `getopts` 명령어로 옵션 해석이 끝난 다음에 쉘 변수 `OPTIND`는 '다음에 처리할 위치 파라미터 번호'를 나타낸다.  
- 같이 읽어보기 
  - https://mug896.github.io/bash-shell/getopts.html


# 002 키보드에서 Ctrl+C 를 입력했을 때 현재 상태를 출력하며 종료하기 
```bash
#!/bin/sh

count=0
trap ' echo
    echo "Try count: $count"
    exit ' INT 

while : 
do
    curl -o /dev/null $1
    count=$(expr $count + 1)
    sleep 1
done
```

- `trap` 명령어 서식은 '' 안에 하고 싶은 처리, 그 다음에 제어하고 싶은 시그널명을 씁니다. 
```bash 
trap ' date; exit ' INT
----  -----------   ---
명령   실행 코드      시그널
```

- 다음은 프로그래밍 하면서 자주 만나는 시그널 들입니다.

|Signal Name|Signal Number|Description|
|---|---|---|
|SIGHUP|1|Hang up detected on controlling terminal or death of controlling process|
|SIGINT|2|Issued if the user sends an interrupt signal (Ctrl + C)|
|SIGQUIT|3|Issued if the user sends a quit signal (Ctrl + D)|
|SIGFPE|8|Issued if an illegal mathematical operation is attempted|
|SIGKILL|9|If a process gets this signal it must quit immediately and will not perform any clean-up operations|
|SIGALRM|14|Alarm clock signal (used for timers)|
|SIGTERM|15|Software termination signal (sent by kill by default)|
https://www.tutorialspoint.com/unix/unix-signals-traps.htm


# 003 키보드에서 사용자 키 입력을 획득해서 변수값으로 이용하기 
```bash 
#!/bin/sh

echo -n "Enter your ID: "
read id

echo "Now your ID is $id"
```


# 004 암호 입력 시 사용자 키 입력을 표시하지 않기 
```bash
#!/bin/sh

username=guest
hostname=localhost

echo -n "Password: "
# 에코백 OFF(-echo)
stty -echo 
read password
# 에코백 ON(echo)
stty echo 

echo 

# 입력한 암호로 내려받기 
wget -q -password="$password" "ftp://${username}@${hostname}/filename.txt
``` 

- `stty`는 터미널 인터페이스의 옵션을 설정 할 수 있는 유틸리티 입니다. 
- `stty` 명령어로 `-echo`를 지정하면 echo back이 되지 않아서 입력한 문자가 화면에 표시되지 않습니다. (_** Echo back : 반향, 메아리_)


# 005 사용자 키 입력을 한 글자만 받기(Enter 키 불필요)
```bash
#!/bin/sh

echo -n "Type Your Answer [y/n]: "

# 현재 터미널 설정을 쉘 변수 tty_state에 백업하고 
# 터미널을 raw 설정함
tty_state=$(stty -g)
stty raw
#키보드에서 문자 하나 읽기 
char=$(dd bs=1 count=1 2> /dev/null)
# 터미널 설정을 원래대로 돌림
stty "$tty_state"

echo 

# 입력된 문자에 따라 처리 분기
case "$char" in 
    [yY])
        echo "Input: YES"
        ;;
    [nN])
        echo "Input: NO"
        ;;
    *)
        echo "Input: What?"
        ;;
esac
``` 

- `stty` 명령어로 터미널을 `raw` 모드로 하면 키 버퍼를 처리하지 않으므로 문자마다 처리할 수 있습니다. 
- 단, `stty` 명령어로 옵션을 변경하면 기존의 옵션이 지워지므로 나중에 설정을 원래대로 돌릴 수 있도록 현재 상태를 저장할 필요가 있습니다. 
- `dd` 명령어를 이용해서 스탠다드 인풋을 스탠다드 아웃풋으로 복사할 수 있습니다. 
  - https://linuxconfig.org/how-dd-command-works-in-linux-with-examples
    - 주로 master boot record를 백업할 때 사용한다.
    - 또한 복사할 때 특정 블록을 스킵해 복사하기 위해 사용하기 한다. (ex. grub bootloader)


# 006 파일을 읽어서 처리할 때 키보드에서 입력받기 
```bash 
#!/bin/sh

tty=`tty`
while read question
do 
    echo $question
    read dir < $tty
    echo "Command: ls $dir"
    ls $dir
done < question.txt
``` 
- `read` 명령어를 활용해 파일을 읽는 도중에 `read` 명령어를 활용해 키보드 입력을 처리할 수 있다. 
- `read` 명령어는 표준 입력(키보드에서 입력)을 읽어서 그 값을 쉘 변수에 대입하지만 파일 내용을 한 줄씩 읽어서 쉘 변수에 대입할 수도 있습니다. 
- 같이 읽어보기 
  - 명령어 치환 : https://wiki.kldp.org/HOWTO/html/Adv-Bash-Scr-HOWTO/commandsub.html


# 007 선택식 메뉴를 표시해서 입력된 숫자 값 처리하기
```bash 
#!/bin/sh 

while : 
do 
    echo "Menu:"
    echo "1) list file"
    echo "2) current directory"
    echo "3) exit"

    read number
    case $number in 
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

- `while` 문을 활용해 menu 처리 스크립트를 작성할 수 있다. 
- case문 마지막에 `*`를 쓰면 지금까지의 조건에 일치하지 않는 값을 처리합니다. 스크립트가 의도하지 않은 동작을 하지 않도록 이런 에러 처리를 잊지 맙시다. 


# 008 표시 문자 색 변경하기 
```bash
#!/bin/sh

echo "Script Start."

# 배경을 회식(47), 문자색을 빨강(31)으로 바꿈
echo -e "\033[47;31m Important Message \033[0m"

echo "Script End"
```

- echo 명령어의 `-e` 옵션을 지정해서 이스케이프 시퀀스라는 제어 코드로 표시할 문자에 색을 입힐 수 있습니다. 
-  파라미터를 여러 개 지정할 때는 `;`으로 나열합니다. 
- Format 
```
  \033[파마리터m 문자열 \033[0m
```
- 같이 읽어보기
  - Escape Sequences : https://mug896.github.io/bash-shell/escape_sequences.html


# 009 달력을 이용해 특정 날짜의 로그 파일 삭제하기 
```bash 
#!/bin/sh

LOG_DIR=/myapp/ap1/log 

# diglog 명령어로 달력 출력
# 선택한 날짜는 표준 에러 출력이므로 임시 파일에 리다이렉트 
dialog --calendar "Select Date" 2 60 2> cal.tmp

# 달력 기능은 일/월/년 형식으로 출력되므로
# 년월일로 바꿈
date_str=$(awk -F / '{print $3$2$1}' cal.tmp)

# 취소되면 임시 파일을 삭제하고 종료
if [ -z "$date_str" ]; then
    rm -f cal.tmp
    exit
fi 

rm -i ${LOG_DIR}/app_log.$date_str

# 임시 파일 삭제 
rm -f cal.tmp 
```

- `dialog`를 이용해서 Console에 창을 표시할 수 있다. 
 

# 010 파일 압축 시 실행 상태를 표시하는 진행바 표시하기
```bash
#!/bin/sh

DATA_DIR=/myapp/datadir

cd $DATA_DIR
tar cf - bigfile1.dat bigfile2.dat | pv | gzip > archive.tar.gz
```

- pv 명령어에 모니터링하고 싶은 출력을 파이프(`|`)를 이용해 진생상황을 표시할 수 있다. 
```
* Syntax pv command
pv fileName
pv OPTIONS fileName
pv fileName > outputFileName
pv OPTIONS | command > outputFileName
command1 | pv | command2
```
- 같이 읽어보기 
  - pv : https://www.geeksforgeeks.org/pv-command-in-linux-with-examples/
