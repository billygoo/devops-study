# 13장 베이그런트 
## 포트 포워딩 설정
```ruby
Vagrant.configure("2") do |config|
    config.vm.box = "ubuntu/bionic64"
    config.vm.network :forwarded_port, host: 8000, guest: 80
end
```
- 로컬 머신의 포트(8000)과 가상 머신의 포트(80)로 연결한다. 
  
## 사설 IP 지정
```ruby
Vagrant.configure("2") do |config|
    config.vm.box = "ubuntu/bionic64"
    config.vm.network "private_network", ip: "192.168.33.10"
end
```
- 로컬에서만 동작하며, 로컬에서 지정한 IP로 서버에 접근할 수 있다. 


## Agent 포워딩 사용하기 
- Agent 포워딩은 SSH 인증시 키를 서버에 놓지 않고 로컬키를 활용해 인증하 SSH를 연결하는 방식을 말한다. 
- 예를 들어 가상 머신에서 github repository를 로컬키를 활용해 인증해 clone할 때 활용할 수있다. 이때 인증키는 서버에 남지 않는 장점이 생긴다.  
```ruby
Vagrant.configure("2") do |config|
    config.vm.box = "ubuntu/bionic64"
    config.ssh.forward_agent = true
end
```

## 베이그런트 프로비저너
- 베이그런트가 시작한 후 가상 머신을 설정할 때 사용되는 외부 프로비저닝 툴을 지정할 수 있는데 이를 베이그런트 프로비저너라고 말한다. 
- 여기에는 Chef, Puppet, Salt, Docker, Ansible 등을 지정할 수 있다. 

### Ansible을 이용한 베이그런트 설정
```ruby
Vagrant.configure("2") do |config|
    config.vm.box = "ubuntu/bionic64"
    config.vm.provision "ansible" do |ansible|
        ansible.playbook = "playbook.yml"
    end
end
```
- 프로비저너는 한번 실행되면, 다음에 실행되지 않는다. 
- 추가로 실행하고 싶을 경우 다음과 같이 명령어를 사용할 수 있다. 
    ```bash
    $ vagrant reload --provision

    $ vagrant up --provision
    ```
- 베이그런트가 실행되면 앤서블 인벤토리 파일이 생성된다. 
  - 경로: `.vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory`
  - 베이그런트 설정에 있는 시스템 이름과 인벤토리의 host명과 일치한다.
- `ansible.limit` 설정을 이용해 베이그런트 가상 머신이 모두 실행 된 이후 한번에 프로비저너를 실행할 수 있다. 
    ```ruby
    VAGRANTFILE_API_VERSION = "2"

    Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
        # Use the same key for each machine
        config.ssh.insert_key = false

        config.vm.define "vagrant1" do |vagrant1|
            vagrant1.vm.box = "ubuntu/bionic64"
        end
        config.vm.define "vagrant2" do |vagrant2|
            vagrant2.vm.box = "ubuntu/bionic64"
        end
        config.vm.define "vagrant3" do |vagrant3|
            vagrant3.vm.box = "ubuntu/bionic64"
            vagrant3.vm.provision "ansible" do |ansible|
                ansible.limit = 'all'
                ansible.playbook = "playbook.yml"
                ansible.groups = {
                    "web"  =>  ["vagrant1"],
                    "task" =>  ["vagrant2"],
                    "redis" => ["vagrant3"]
                }
            end
        end
    end
    ```
    - `ansible.groups`를 이용하면 그룹을 지정하면 인벤토리 파일도 그룹이 설정되어 생성된다. 
- 로컬 앤서블 프로비저너 : 앤서블 설치 없이 가상 머신에서 앤서블을 실행할 수 있는 `ansible_local`을 이용할 수 있다. 
    ```ruby
    Vagrant.configure("2") do |config|
        config.vm.box = "ubuntu/bionic64"
        config.vm.provision "ansible_local" do |ansible|
            ansible.playbook = "playbook.yml"
        end
    end
    ```

# References
- [Using SSH agent forwarding](https://docs.github.com/en/developers/overview/using-ssh-agent-forwarding)