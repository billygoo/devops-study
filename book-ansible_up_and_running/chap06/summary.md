# 앤서블을 사용해 매자닌 배포하기 
## 암호 변수 관리하기 
- 별도의 변수 파일을 선언해 관리하며, 이 파일의 경우 버전관리 시스템에 넣지 않고 별도로 관리한다.
- 또한, `.gitignore` 파일을 선언해서 실수로 버전관리 시스템에 포함되지 않도록 한다. 

## Task에 with_items를 이용해 반복 실행하기 
- `with_items`을 사용하면 여러 패키지를 설치하거나 할때 반복적으로 실행 시킬 수 있다. 
- 또한, `with_items`을 사용하는 모듈에 따라서 최적화가 적용되어 한 번에 일괄적으로 실행될 수 있음 
```yaml
- name: install Python requirements globally via pip
    pip: name={{ item }} state=latest
    with_items:
    - pip
    - virtualenv
    - virtualenvwrapper
```

## Task의 매개변수가 길 경우 활용 방법
```yaml
- name: create tls certificates
    command: >
    openssl req -new -x509 -nodes -out {{ proj_name }}.crt
    -keyout {{ proj_name }}.key -subj '/CN={{ domains[0] }}' -days 3650
    chdir={{ conf_path }}
    creates={{ conf_path }}/{{ proj_name }}.crt
```
- 파라미터가 너무 길 경우 라인 폴딩 기능을 활용할 수 있다. 

```yaml
# 기존
- name: install required python packages
    pip: name={{ item }} virtualenv={{ venv_path }}

# dict 형태로 전달
- name: install required python packages
    pip: 
        name: "{{ item }}"
        virtualenv: "{{ venv_path }}"
```
- 위 예시와 같이 dict 형태로 변경해 전달할수도 있다. 

