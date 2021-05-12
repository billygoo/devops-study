# 51 date 명령어로 두 날짜를 비교하고 날짜차를 구하기
```bash
#!/bin/sh

# 비교할 두 날짜를 변수로 정의
day1="2012/04/01 10:49:41"
day2="2012/03/30 08:31:52"

# 날짜에서 epoch 초를 얻으려면 +%s 사용(리눅스)
# -d 옵션은 FreeBSD/Mac에서 사용 불가 
day1_epoch=$(date -d "$day1" '+%s')
day2_epoch=$(date -d "$day2" '+%s')

echo "day1($day1): $day1_epoch"
echo "day2($day2): $day2_epoch"

# 두 날짜의 epoch초끼리 뺀 값을 
# 하루 = 24시간 = 1440분 = 86400초로 나누면 날짜 계산 가능 
echo "day interval: " 
## Fix typo : 결과값을 변수에 할당
day_interval=$(expr \( $day1_epoch - $day2_epoch \) / 86400)
echo $day_interval
```
- 유닉시 시간 : Unix epoch로 1970년 1월 1일 0시 0분 0초 기준으로 경과한 초를 나타내는 값



# 052 오늘이 마일인지 판별하기 
```bash
#!/bin/sh

tomorrow=$(date "+%d" -d '1 day') 

if [ "$tomorrow" = "01" ]; then 
    # 오늘이 말일이라면 월별 리포트를 작성하는 외부 스크립트 실행 
    ./monthly_report.sh
fi
```
- `date` 명령어는 `-d` 옵션을 써서 현재 시각의 전후를 지정할 수 있음
  - mac에서는 `-v` 옵션을 사용해야 함

# 053 한달 전에 만든 로그 파일을 일괄 아카이브하기 
```bash
#!/bin/sh

logdir="/var/log/myapp"

# 이번달 15일 날짜 취득 
thismonth=$(date '+%Y/%m/15 00:00:00')

# 지난달 날짜를 YYYYMM으로 취득
# 1 month ago는 지난달의 같은 '날(일)'이 되므로 말일을 피하도록 
# 변수 thismonth에 15일을 지정함 
last_YYYYMM=$(date -d "$thismonth -1 month ago" '+%Y%m')

# 지난달 로그 파일을 아카이브
tar cvf ${last_YYYYMM}.tar ${logdir}/access.log-${last_YYYYMM}*
```
- 날짜 연산(`date -d`)을 이용하면 필요한 월 또는 년으로 작업을 진행할 수 있음

# 054 윤년인지 확인하기 
```bash
#!/bin/sh

# 네 자리 년도 얻기 
year=$(date '+%Y')

logfile="/var/log/myapp/access.log-"

# 년도를 나눈 나머지 계산
mod1=$(expr $year % 4)
mod2=$(expr $year % 100)
mod3=$(expr $year % 400)

# 윤년인지 판정 
if [ $mod1 -eq 0 -a $mod2 -ne 0 -o $mod3 -eq 0 ]; then
  echo "leap year:$year"
  ls "${logfile}${year}0229"
else
  echo "not leap year:$year"
  ls "${logfile}${year}0228"
fi
```
- 윤년 기준 
  - 서력이 4로 나눠 떨어지면 윤년
  - 단, 100으로 나눠 떨어지면 안됨
  - 단, 400으로 나눠 떨어지면 윤년
- test 비교 연산자 
    | 표기 | 산술식 |
    |---|---|
    | `var1 -eq var2` | `var1 == var2` |
    | `var1 -ne var2` | `var1 != var2` |
    | `var1 -lt var2` | `var1 < var2`  |
    | `var1 -le var2` | `var1 <= var2` |
    | `var1 -gt var2` | `var1 > var2`  |
    | `var1 -ge var2` | `var1 >= var2` |
- test 논리 연산자
    | 논리 연산자 | 표기법 | 우선순위 | 기타 |
    |---|---|---|---|
    | and | `-a` | 1 | |
    | or | `-o` | 2 | and 보다 우선순위가 낮기 때문에 우선순위를 높이러면 괄호('`(`', '`)`')를 써야 한다. 또한 쉘 해석을 피하기 위해 역슬래쉬('`\`')를 같이 써야한다. |

