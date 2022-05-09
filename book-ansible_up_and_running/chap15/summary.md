# 15장 도커
## docker_container 모듈 
- docker 컨테이너의 라이프사이클을 관리
  - https://docs.ansible.com/ansible/2.5/modules/docker_container_module.html
```yaml
- name: Create a data container
  docker_container:
    name: mydata
    image: busybox
    volumes:
      - /data

- name: Re-create a redis container
  docker_container:
    name: myredis
    image: redis
    command: redis-server --appendonly yes
    state: present
    recreate: yes
    exposed_ports:
      - 6379
    volumes_from:
      - mydata

- name: Restart a container
  docker_container:
    name: myapplication
    image: someuser/appimage
    state: started
    restart: yes
    links:
     - "myredis:aliasedredis"
    devices:
     - "/dev/sda:/dev/xvda:rwm"
    ports:
     - "8080:9000"
     - "127.0.0.1:8081:9001/udp"
    env:
        SECRET_KEY: ssssh

- name: Container present
  docker_container:
    name: mycontainer
    state: present
    image: ubuntu:14.04
    command: sleep infinity

- name: Stop a container
  docker_container:
    name: mycontainer
    state: stopped

- name: Start 4 load-balanced containers
  docker_container:
    name: "container{{ item }}"
    recreate: yes
    image: someuser/anotherappimage
    command: sleep 1d
  with_sequence: count=4

- name: remove container
  docker_container:
    name: ohno
    state: absent

- name: Syslogging output
  docker_container:
    name: myservice
    image: busybox
    log_driver: syslog
    log_options:
      syslog-address: tcp://my-syslog-server:514
      syslog-facility: daemon
      # NOTE: in Docker 1.13+ the "syslog-tag" option was renamed to "tag" for
      # older docker installs, use "syslog-tag" instead
      tag: myservice

- name: Create db container and connect to network
  docker_container:
    name: db_test
    image: "postgres:latest"
    networks:
      - name: "{{ docker_network_name }}"

- name: Start container, connect to network and link
  docker_container:
    name: sleeper
    image: ubuntu:14.04
    networks:
      - name: TestingNet
        ipv4_address: "172.1.1.100"
        aliases:
          - sleepyzz
        links:
          - db_test:db
      - name: TestingNet2

- name: Start a container with a command
  docker_container:
    name: sleepy
    image: ubuntu:14.04
    command: ["sleep", "infinity"]

- name: Add container to networks
  docker_container:
    name: sleepy
    networks:
      - name: TestingNet
        ipv4_address: 172.1.1.18
        links:
          - sleeper
      - name: TestingNet2
        ipv4_address: 172.1.10.20

- name: Update network with aliases
  docker_container:
    name: sleepy
    networks:
      - name: TestingNet
        aliases:
          - sleepyz
          - zzzz

- name: Remove container from one network
  docker_container:
    name: sleepy
    networks:
      - name: TestingNet2
    purge_networks: yes

- name: Remove container from all networks
  docker_container:
    name: sleepy
    purge_networks: yes
```

## docker_image 모듈
- docker image를 관리(빌드, Load, Pull, Push)
  - https://docs.ansible.com/ansible/2.5/modules/docker_image_module.html
```yaml
- name: pull an image
  docker_image:
    name: pacur/centos-7

- name: Tag and push to docker hub
  docker_image:
    name: pacur/centos-7
    repository: dcoppenhagan/myimage
    tag: 7.0
    push: yes

- name: Tag and push to local registry
  docker_image:
     name: centos
     repository: localhost:5000/centos
     tag: 7
     push: yes

- name: Remove image
  docker_image:
    state: absent
    name: registry.ansible.com/chouseknecht/sinatra
    tag: v1

- name: Build an image and push it to a private repo
  docker_image:
    path: ./sinatra
    name: registry.ansible.com/chouseknecht/sinatra
    tag: v1
    push: yes

- name: Archive image
  docker_image:
    name: registry.ansible.com/chouseknecht/sinatra
    tag: v1
    archive_path: my_sinatra.tar

- name: Load image from archive and push to a private registry
  docker_image:
    name: localhost:5000/myimages/sinatra
    tag: v1
    push: yes
    load_path: my_sinatra.tar

- name: Build image and with buildargs
  docker_image:
     path: /path/to/build/dir
     name: myimage
     buildargs:
       log_volume: /var/log/myapp
       listen_port: 8080
```


## docker_service 모듈 
- docker-compose.yml을 이용해 서비스를 제어 할 수 있다. 
  - https://docs.ansible.com/ansible/2.5/modules/docker_service_module.html
```yaml
- name: Run using a project directory
  hosts: localhost
  connection: local
  gather_facts: no
  tasks:
    - docker_service:
        project_src: flask
        state: absent

    - docker_service:
        project_src: flask
      register: output

    - debug:
        var: output

    - docker_service:
        project_src: flask
        build: no
      register: output

    - debug:
        var: output

    - assert:
        that: "not output.changed "

    - docker_service:
        project_src: flask
        build: no
        stopped: true
      register: output

    - debug:
        var: output

    - assert:
        that:
          - "not web.flask_web_1.state.running"
          - "not db.flask_db_1.state.running"

    - docker_service:
        project_src: flask
        build: no
        restarted: true
      register: output

    - debug:
        var: output

    - assert:
        that:
          - "web.flask_web_1.state.running"
          - "db.flask_db_1.state.running"
```

## docker_image_facts 모듈
- docker 이미지의 메타데이터를 읽어올 수 있다. 
  - https://docs.ansible.com/ansible/2.5/modules/docker_image_facts_module.html
```yaml
- name: Inspect a single image
  docker_image_facts:
    name: pacur/centos-7

- name: Inspect multiple images
  docker_image_facts:
    name:
      - pacur/centos-7
      - sinatra
```

## Ansible Container 
- 망했다. 
- 컨테이너 이미지를 빌드하기 위해선 [ansible-bender](https://github.com/ansible-community/ansible-bender) 프로젝트를 추천함 
- k8s에 컨테이너를 배포하기 위해선 [ansible-operator](https://www.ansible.com/integrations/containers/operators) 프로젝트를 추천함

