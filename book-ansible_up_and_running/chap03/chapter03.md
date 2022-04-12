# 인벤토리: 서버 설명 
- 앤서블은 여러 호스트를 관리한다. 이를 관리하기 위한 호스트 집합을 인벤토리(inventory)라고 한다.


## 인벤토리 파일 
- 앤서블에서 호스트를 설명하는 기본 방법 
- 앤서블이 SSH 설정 파일(`~/.ssh`)을 인식하기 때문에 호스트 명에 Alias 이름을 사용할 수 있다. 
  (단, Paramiko 플러그인 사용시는 인지 못함. 이유는 당연히 별도 프로그래밍 언어 사용하기 때문)
- INI 파일 포맷과 YAML 파일 포맷을 지원
```ini
# INI Format
mail.example.com

[webservers]
foo.example.com
bar.example.com

[dbservers]
one.example.com
two.example.com
three.example.com
```

```yaml
# YAML format
all:
  hosts:
    mail.example.com:
  children:
    webservers:
      hosts:
        foo.example.com:
        bar.example.com:
    dbservers:
      hosts:
        one.example.com:
        two.example.com:
        three.example.com:
```

## 동적 매개변수 

|이름 | 기본값 | 설명 |
|---|---|---|
|ansible_host | 호스트 이름 | SSH로 연결할 호스트 이름 또는 IP 주소 |
|ansible_port | 22 | SSH로 연결할 포트 | 
|ansible_user | root | SSH로 연결할 사용자 | 
|ansible_password | N/A | SSH 인증에 사용될 패스워드 |
|ansible_connection | smart | 호스트에 연결하는 방법 | 
|ansible_private_key_file | N/A | SSH인증에 사용될 SSH Private key | 
|ansible_shell_type | sh | Command가 실행되는 Shell | 
|ansible_python_interprter | /usr/bin/python | 호스트의 파이썬 인터프리터 경로 | 
|ansible_*_interpreter | N/A | 다른 언어의 인터프리터 지정할 필요가 있을때 사용 |

- `ansible_connection` : SSH 클라이언트의 Control Persist 기능 지원여부를 체크하며,
  해당 기능을 지원한다면 SSH 클라이언트를 사용하고, 그렇지 않다면 Paramiko를 사용해 SSH
  접속 한다. 
    - SSH의 `ControlPersist` 옵션 : `ControlMaster` 옵션과 함께 사용되며, Mutiplexing 
    기능을 위한 옵션이다. 
- ansible.cfg 파일의 defaults 섹션에 기본값을 정의할 수 있다. 
  | 매개변수 | ansible.cfg | 
  |---|---|
  | ansible_port | remote_port |
  | ansible_user | remote_user |
  | ansible_private_key_file | private_key_file |
  | ansible_shell_type | 확인 필요 !! |


## 그룹 
- 형식 1
  ```ini
  # Host를 나열하고 나중에 그룹핑 하는 방식
  delaware.example.com
  georgia.example.com
  maryland.example.com
  newhampshire.example.com
  newjersey.example.com
  ontario.example.com
  quebec.example.com

  [production]
  delaware.example.com
  georgia.example.com
  maryland.example.com

  [staging]
  ontario.example.com
  quebec.example.com

  # ~~~~~
  ```
- 형식 2
  ```ini
  # 바로 그룹핑 하는 방식
  [production]
  delaware.example.com
  georgia.example.com
  maryland.example.com
  newhampshire.example.com
  newjersey.example.com
  newyork.example.com
  northcarolina.example.com
  pennsylvania.example.com
  rhodeisland.example.com
  virginia.example.com

  [staging]
  ontario.example.com
  quebec.example.com

  [lb]
  delaware.example.com

  [web]
  georgia.example.com
  newhampshire.example.com
  newjersey.example.com
  ontario.example.com
  vagrant1

  # ~~~~~
  ```

### 특징
- `all`(또는 `*`)이라는 그룹이 자동으로 정의됨
- 인벤토리 파일에서 자체 그룹을 정의하며 `.ini` 파일 포맷을 사용 
  - [`.ini` 파일 포맷](https://en.wikipedia.org/wiki/INI_file) : Section이 Group과 매핑됨
- 인벤토리 파일에서 호스트의 동작 인벤토리 매개변수를 사용할때는 한 번만 지정해야 한다.
- 그룹의 그룹을 사용할 수 있으며, `:children`이라는 접미사를 사용해야만 한다.
  ```ini
  [atlanta]
  host1
  host2

  [raleigh]
  host2
  host3

  [southeast:children]   # here
  atlanta
  raleigh
  ```
- 호스트를 패턴으로 지정해 사용할 수 있다.
  ```ini
  web[1:20].example.com       # web1 ~ web20
  
  web[01:20].example.com      # web01 ~ web20

  web-[a-t].example.com       # web-a ~ web-t
  ```

## 호스트 변수와 그룹 변수
- 시스템의 로그인 암호, 또는 web과 db를 연결할때 필요한 정보를 전달하기 위해 변수를 사용한다. 
- 변수를 사용하기 위해서는 `호스트 변수`와 `그룹 변수`를 지정해 사용할 수 있다. 

### 호스트 변수
- 호스트 변수는 하나의 호스트에 사용할 변수를 지정하는 방식이다. 
- 인벤토리 파일 호스트 선언 할때나 또는 별도의 파일(`host_vars/[호스트명]`)로 분리해 사용할 수 있다. 


### 그룹 변수
그룹 변수를 지정하여 dev/staging/production과 같은 스테이지 정보로 활용 할 수 있다. 지정 방법은 다음과 같다. 

1. 인벤토리 내에 그룹 변수를 지정 : `:vars`라는 접미사가 붙은 `<그룹명>:vars`의 형식을 사용
  ```ini
  [all:vars]
  ntp_server=ntp.ubuntu.com

  [production:vars]
  db_primary_host: rhodeisland.example.com
  db_replica_host: virginia.example.com
  db_name: widget_production
  db_user: widgetuser
  db_password: pFmMxcyD;Fc6)6
  rabbitmq_host: pennsylvania.example.com

  [staging:vars]
  db_primary_host: rhodeisland.example.com
  db_replica_host: virginia.example.com
  db_name: widget_production
  db_user: widgetuser
  db_password: pFmMxcyD;Fc6)6
  rabbitmq_host: pennsylvania.example.com

  # ~~~~~
  ```
2. 별도의 파일로 그룹 변수를 지정 : `group_vars/<그룹명>`의 형태로 별도의 파일을 작성한다.
  - 디렉토리 구조
    ```bash
    root_path
    ├── Vagrantfile
    ├── ansible.cfg
    ├── host_vars
    ├── group_vars
    │   ├── staging
    │   │   └── db        # staging그룹의 db 그룹 (nested group)
    │   └── production     # production 그룹 변수
    └── inventory
         └── hosts
    ```
    - `group_vars/production` file 
      ```yaml
      ---
      db_primary_host: rhodeisland.example.com
      db_replica_host: virginia.example.com
      db_name: widget_production
      db_user: widgetuser
      db_password: pFmMxcyD;Fc6)6
      rabbitmq_host: pennsylvania.example.com
      ```
    - `group_vars/staging/db` file : `YAML`
      ```yaml
      db:
        user: widgetuser
        password: pFmMxcyD;Fc6)6
        name:  widget_production
        primary:
          host: rhodeisland.example.com
          port: 5432
        replica:
          host: virginia.example.com
          port: 5432
      ```
    - `YAML`형태로 작성해야 하고, 파일 확장자는 삭제 가능하다
    - 그룹을 하위 그룹 파일을 나눌 수 있음
  - 다음과 같은 형태로 변수를 접근 할 수 있다.
    ```bash
    {{ 변수명 }}

    {{ db_primary_host }}  # yaml에 표현 형식에 따라 접근 방식이 달라짐
    {{ db.primary.host }}
    ```

## 동적 인벤토리
- 동적으로 호스트를 생성할 때 사용한다. 
- 예를 들어 Cloud에서 사용 가능한 자원 목록 대상으로 Provisioning 할때 동적으로 인벤토리 목록 생성할 때 활용 가능

### 사용 방법 
- `inventory` 하위 디렉토리에 실행 가능한 권한을 가지고 있는 파일의 경우 Ansible에서 읽기 대신 실행 처리 
- 실행시 `--host=<호스트명>`, `--list` 매개변수를 처리 할 수 있어야 한다.
  - `--host=<호스트명>` : 지정된 host의 정보만 표시한다. 동작 매개변수를 포함해야한다. 
    ```json
    {
      "testserver": {
        "ansible_host": "127.0.0.1",
        "ansible_port": "2222",
        "ansible_python_interpreter": "/usr/bin/python3"
      }
    }
    ```
  - `--list` : host 정보를 출력한다. 
    ```json
    {
      "production": [
        "delaware.example.com",
        "georgia.example.com",
        "maryland.example.com",
        "newhampshire.example.com",
        "newjersey.example.com",
        "newyork.example.com",
        "northcarolina.example.com",
        "pennsylvania.example.com",
        "rhodeisland.example.com",
        "virginia.example.com"
      ],
      "staging": [
        "ontario.example.com",
        "quebec.example.com"
      ],
      "lb": [
        "delaware.example.com"
      ],
      "web": [
        "georgia.example.com",
        "newhampshire.example.com",
        "newjersey.example.com",
        "ontario.example.com",
        "vagrant1"
      ]
    }
    ```
- 출력은 포맷으로 기존 `ini`나 `yaml`로 작성된 데이터를 `json` 형태로 출력해야한다.
- 기존 인벤토리 파일과 동적 인벤토리 파일을 사용할 수 있다. 
  1. `ansible.cfg` 파일에 `inventory` 매개변수에 디렉토리 지정하기 
  2. 커맨드라인 실행시 `-i` 옵션 사용해 지정하기

## 런타임에 add_host와 group_by를 사용해 항목 추가하기 
### `add_host` 모듈
- 클라우드 내 새로운 VM을 추가하는 경우 해당 모듈을 이용해 host를 추가할 수 있다. 
- 사용할 때 가상 머신을 생성(provisioning)을 하는 절차와 설정(configuration)을 나누어서 활용한다.
  - 동적 스크립트를 이용해 서버 목록을 읽어올때 플레이북이 실행되기전 동적 스크립트가 먼저 실행된다.
  - 즉, 추가로 새로운 호스트가 추가되었어도, 이미 동적 스크립트가 실행되기 때문에 새로운 것을 인식 못하게 된다. 

### `group_by` 모듈 
- 플레이북 실행 중에 새로운 그룹을 생성할 수 있다. 
- 주로 팩트(`fact`)와 연계해서, 호스트들을 배포된 리눅스 타입에 따라 동적으로 그룹을 지정할 수 있다.
  - 즉, 리눅스 배포 그룹 마다 설치 모듈을 알맞게 지정해 사용 가능하다.
