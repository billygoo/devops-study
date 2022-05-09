# 12장 사용자 정의 모듈
복잡한 태스크를 수행할 필요가 있지만 적합한 모듈이 없을 경우 사용자 정의 모듈을 작성할 수 있다. 

## Ansible이 모듈을 호출하면 어떻게 동작하나?
1. 매개변수를 사용해 Standalone 파이썬 스크립트 생성(if python module) 
2. 호스트에 모듈 복사 
3. 호스트에 매개변수 파일 생성(if not python module)
4. 호스트에서 매개변수 파일을 전달해 모듈 스크립트 실행
5. 표준 출력으로 생성된 모듈 실행 결과를 파싱

## 사용자 정의 모듈 작성 규칙
### 사용자 정의 모듈 저장 위치 
- 모든 플레이북 및 롤 : 환경변수 `ANSIBLE_LIBRARY`로 지정된 디렉토리 하위
  - ansible.config에서 DEFAULT_MODULE_PATH 값으로 조정가능 : `~/.ansible/plugins/modules:/usr/share/ansible/plugins/modules`
- 특정 플레이북 및 싱글 롤에서 사용할 때 : 플레이북의 library 디렉토리 

### 사용자 정의 호출 결과 
- json 형태로 반환해야 한다. 
```json
{'changed': false, 'failed': true, 'msg': 'could not reach the host'}
```
  - `changed` : 상태가 변경 됐는지 여부를 반환
  - `failed` : 모듈이 실패 여부를 반환 
  - `msg` : 모듈이 실패했을 경우의 메시지

## python을 이용한 사용자 정의 모듈 구현 
- `AnsibleModule` 파이선 클래스를 활용
```python
#!/usr/bin/python
from ansible.module_utils.basic import AnsibleModule

# 실제 동작을 처리하는 함수
def can_reach(module, host, port, timeout):
    nc_path = module.get_bin_path('nc', required=True)
    args = [nc_path, "-z", "-w", str(timeout),
            host, str(port)]
    (rc, stdout, stderr) = module.run_command(args)
    return rc == 0

def main():
    # 사용자 정의 모듈이 입력받을 아규먼트를 정의
    module = AnsibleModule(
        argument_spec=dict(
            host=dict(required=True),
            port=dict(required=True, type='int'),
            timeout=dict(required=False, type='int', default=3)
        ),
        supports_check_mode=True
    )

    # check_module은 시스템 상태를 절대로 변경하지 않기 때문에 
    # check_mode를 사용하면 특별히 할 일은 없다. 
    # 따라서 changed=False를 리턴만 하면 된다.
    if module.check_mode:
        module.exit_json(changed=False)

    host = module.params['host']
    port = module.params['port']
    timeout = module.params['timeout']

    if can_reach(module, host, port, timeout):
        module.exit_json(changed=False)
    else:
        msg = "Could not reach %s:%s" % (host, port)
        module.fail_json(msg=msg)

if __name__ == "__main__":
    main()
```
- `AnsibleModule`을 이용해 모듈에서 입력이 필요한 파라미터 및 옵션 설정
  - `AnsibleModule` 매개변수
    |매개변수                | default | 설명|
    |---|---|---|
    |argument_spec          | N/A   | 매개변수에 대한 정보를 포함하는 dict |
    |bypass_checks          | False | 매개변수 제약 조건 확인 여부 |
    |no_log                 | False | 모듈 동작 로깅 여부로, 로깅에 민감한 정보가 있을 경우 활성화함 |
    |check_invaild_arguments| True  | 적절하지 않은 매개변수 사용시 에러 발생 |
    |mutually_exclusive     | N/A   | 같이 사용하면 안되는 매개변수 지정, list로 값 지정|
    |required_together      | N/A   | 같이 사용해야 하는 매개변수 지정, list로 값 지정 |
    |required_one_of        | N/A   | 적어도 하나는 필수인 매개변수 |
    |add_file_common_args   | False | file 모듈 매개변수 지원 여부, 사용자 모듈 + file 모듈(ex. 권한, 사용자 변경)과 같이 동작함 |
    |supports_check_mode    | False | Check mode 지원 여부 |
    - `Check mode` : dry-run 모드와 비슷하게 실제 실행은 하지 않고 전체 동작을 점검하는 모드 
  - 매개변수 설정(`argument_spec`)에 활용되는 옵션
    |옵션|설명|
    |---|---|
    |required| 매개변수 필수 여부 |
    |default | 매개변수 기본값 |
    |choices| 매개변수에서 사용할 수 있는 값 리스트|
    |aliases| 매개변수에 대한 별칭들로 리스트로 값을 지정한다 |
    |type| 매개변수 타입으로 다음을 지정 가능. `str`, `list`, `dict`, `bool`, `int`, `float` |
- `AnsibleModule.params`(dict type)을 이용해 입력된 파라미터를 읽음
- 실제 실행할 함수를 호출 후 `AnsibleModule.exit_json()` 또는 `AnsibleModule.fail_json()`를 이용해 결과 출력
- `AnsibleModule.run_command()` : 외부 프로그램을 호출할 수 있는 함수

### 모듈 문서화
https://docs.ansible.com/ansible/latest/dev_guide/developing_modules_documenting.html
- 모듈 스크립트 내에 `DOCUMENTATION`과 `EXAMPLE`, 그리고 `RETURN` 변수를 사용해 문서화
- 변수의 설명 값은 YAML기반 문법을 사용한다. 


### 사용자 정의 모듈 디버깅하기 
- Ansible에서 제공하는 `test-module` 프로그램을 이용해 모듈을 테스트 할 수 있다.
  - 
```bash
$ git clone https://github.com/ansible/ansible.git --recursive
$ source ansible/hacking/env-setup
$ ansible/hacking/test-module -m [module file path] -a "[arguments]"
```

## Bash를 이용한 사용자 정의 모듈 구현 
```bash
#!/bin/bash
# WANT_JSON

# 파일에서 변수를 읽는다
host=`jq -r .host < $1`
port=`jq -r .port < $1`
timeout=`jq -r .timeout < $1`

# 기본 타임아웃은 3이다
if [[ $timeout = null ]]; then
    timeout=3
fi

# 호스트에 연결할 수 있는지 확인한다
nc -z -w $timeout $host $port

# 성공 또는 실패를 기반으로 출력
if [ $? -eq 0 ]; then
    echo '{"changed": false}'
else
    echo "{\"failed\": true, \"msg\": \"could not reach $host:$port\"}"
fi
```
- bash를 이용할때 사용하는 유틸리티는 실행전에 호스트에 설치되어 있어야 한다.
- `# WANT_JSON` 주석을 이용하면 입력을 ansible에서 아규먼트를 json 형태로 전달한다.
- `shebang` 사용시에 env를 호출 하면 안된다. 
  - ansible에서 파일에 지정된 shebang을 확인 후 존재하지 않으면 `ansible_*_interpreter`로 사용해 프로그램을 호출한다.



# Reference 
- https://docs.ansible.com/ansible/latest/dev_guide/developing_locally.html#adding-a-module-outside-of-a-collection
- [Modules and plugins: what is the difference?](https://docs.ansible.com/ansible/latest/dev_guide/developing_locally.html#adding-a-module-outside-of-a-collection)