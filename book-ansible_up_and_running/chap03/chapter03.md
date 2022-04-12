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
  - 

## 그룹 


## 호스트와 그룹 변수 


## 호스트 변수와 그룹 변수 


## 동적 인벤토리

## 여러 파일로 인벤토리 분할 

## 런타임에 add_host와 group_by를 사용해 항목 추가하기 