# 9장 호스트, 실행, 핸들러 사용자 정의 

## 호스트 지정 패턴 
|Action| Example|
|---|---|
|모든 호스트|all or *|
|합집합|dev:staging|
|교집합|staging:&database|
|배제|dev:!queuee|
|와일드카드|*.example.com|
|범위|web[5:10]|
|정규식| ~web\d+\.example\.(com\|org)|

- Usage
  1. 플레이북 파일 내 `hosts` 필드값으로 명시 
  2. ansible-playbook 명령 실행시 `-l` 파라미터 사용
     ```bash
     $ ansible-playbook -l 'staging:&database' playbook.yml
     $ ansible-playbook --limit 'staging:&database' playbook.yml
     ```

- local_action : 제어 머신에서 특정 태스크 실행하기 
- delegate_to : 다른 호스트에서 특정 태스크 실행하기
  - ex) lb 서버에서 특정 호스트를 LB에 등록하려고 할때 
- wait_for 모듈 : SSH 서버거 연결을 수락할때까지 기다리기
- 