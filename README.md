# Learning Ansible
> 아주 쉽게 정리한 Ansible @jacob

# Introduction

## 개요

- 개발언어 : python
- 정의  : YAML
- Agent/SSH여부 : SSH
- 통신방법 : json
- 멱등성 보장
  - 여러 번 적용해도 결과는 바뀌지 않는다.
  - Ansible 모듈 대부분 멱등성을 제공
  - 멱등성을 제공하지 않는 모듈
    - shell, command, file
- Documentation
  - http://docs.ansible.com/



## 타 대표 제품과의 비교

![비교](http://networknuts-web.biz/wp-content/uploads/2015/11/ansible-chef-puppet.png)

## 설치

```sh
$ brew install ansible
$ sudo yum install ansible
$ sudo apt-get ansible
```



## 환경 파일

ansible 프로젝트 홈 밑에 아래 이름으로 생성하면 알아서 읽어드림  
참조 : http://docs.ansible.com/ansible/intro_configuration.html

```sh
$ANSIBLE_HOME/ansible.cfg
$ANSIBLE_HOME/.ansible.cfg
```





## Your first commands

### 홈 디렉토리 생성

> Ansible 홈 생성. 앞으로 이 디렉토리를 활용한다.

```Bash
$ mkdir learning_ansible
```

### 호스트 등록

```Bash
$ cd learning_ansible
$ mkdir -p hosts
$ vi ./hosts/admin
```

```
[web]
ansible-test-web01
ansible-test-web02

[db]
ansible-test-db01
```

### 명령어를 날려보자

```bash
# ping
$ ansible all -i hosts/admin -m ping -u deploy
# setup
$ ansible all -i hosts/admin -m setup -u deploy
```

### 자주 사용하는 것들은 ansible.cfg 설정

> -u jacob 은 매번 적기 귀찮으니 기본으로 설정되도록 ansible.cfg 에 설정한다.

```bash
$ vi ansible.cfg
```

```Bash
[defaults]
# host_key_checking: ssh 첫 접속 시 yes/no 출력 무시
host_key_checking = False
# SSH settings
remote_user = deploy
remote_port = 22
```

### 다시 명령어를 날려보자

``` sh
# ping
$ ansible all -i hosts/first -m ping

# setup
$ ansible all -i hosts/first -m setup

# web 서버만 ping
$ ansible all -i hosts/first -l web -m ping

# ansible-test-web01 서버만 ping 체크
$ ansible all -i hosts/first -l "ansible-test-web01" -m ping

# ansible-test-web01,02 2대 서버만 ping 체크
$ ansible all -i hosts/first -l "ansible-test-web01,ansible-test-web02" -m ping
$ ansible all -i hosts/first -l "ansible-test-web01 ansible-test-web02" -m ping
$ ansible all -i hosts/first -l "ansible-test-web0[1-2]" -m ping
```





## Inventory

### Hosts and Groups

일반적인 형태

```
mail.example.com

[webservers]
foo.example.com
bar.example.com

[dbservers]
one.example.com
two.example.com
three.example.com
```

별칭 지정 및 터널을 통해 연결 하고자 할 경우

```javascript
jumper ansible_port = 5555 ansible_host = 192.0.2.50
```

그밖의 여러가지 형태

```java
[webservers]
www[01:50].example.com

[databases]
db-[a:f].example.com
  
[targets]
localhost              ansible_connection=local
other1.example.com     ansible_connection=ssh        ansible_user=mpdehaan
other2.example.com     ansible_connection=ssh        ansible_user=mdehaan
```

> 파라미터 종류들
> http://docs.ansible.com/ansible/latest/intro_inventory.html#list-of-behavioral-inventory-parameters

### Host Variables

```java
[atlanta]
host1 http_port=80 maxRequestsPerChild=808
host2 http_port=303 maxRequestsPerChild=909
```

### Grups of Groups, and Group Variables

> 이해하기 쉽게 우리나라 지역 한글명으로 설명한다.

```
[전라남도]
목포
순천

[전라북도]
전주
정읍

[전라도:children]
전라남도
전라북도

[전라도:vars]
some_server=foo.southeast.example.com
halon_system_timeout=30
self_destruct_countdown=60
escape_pods=2

...(중략)...

[대한민국:children]
전라도
경상도
경기도
```





## Ansible 실행 및 옵션

기본적인 형태

```sh
ansible all -i hosts/web -l "alpha,sandbox" -m copy -a "src=/etc/hosts dest=/tmp/hosts" -f 10
```

- -i : INVENTORY
- -l : SUBSET. 그룹 혹은 호스트 지정
- -m : MODULE_NAME. Ansible에서 정의한 모듈을 사용([모듈보기](http://docs.ansible.com/ansible/modules_by_category.html)).지정한 모듈을 사용해야  ***멱등성*** 을 보장 받을 수 있음.
- -a : MODULE_ARGS. 
- -f : FORKS. 병렬 처리할 프로세스 개수 (default : 5)
- -e : EXTRA_VARS. 추가적으로 변수 사용 시





## 실행 예

``` sh
$ ansible atlanta -m copy -a "src=/etc/hosts dest=/tmp/hosts"

# File
$ ansible webservers -m file -a "dest=/srv/foo/a.txt mode=600"
$ ansible webservers -m file -a "dest=/srv/foo/b.txt mode=600 owner=mdehaan group=mdehaan"
$ ansible webservers -m file -a "dest=/path/to/c mode=755 owner=mdehaan group=mdehaan state=directory"
$ ansible webservers -m file -a "dest=/path/to/c state=absent"

# Yum
$ ansible webservers -m yum -a "name=acme state=present"
$ ansible webservers -m yum -a "name=acme-1.5 state=present"
$ ansible webservers -m yum -a "name=acme state=latest"
$ ansible webservers -m yum -a "name=acme state=absent"

# Users and groups
$ ansible all -m user -a "name=foo password=<crypted password here>"
$ ansible all -m user -a "name=foo state=absent"

# Deploying From Source Control
$ ansible webservers -m git -a "repo=git://foo.example.org/repo.git dest=/srv/myapp version=HEAD"

# Managing services
$ ansible webservers -m service -a "name=httpd state=started"
```


# Playbooks

## 미리 준비할 것들

### ansible.cfg

테스트가 용이하게 기본적인 환경설정을 하자.  
http://docs.ansible.com/ansible/intro_configuration.html

```properties
[defaults]
host_key_checking = False

# SSH settings
remote_user = deploy
remote_port = 22

roles_path = ./roles
```

### 디렉토리 구조

> mkdir -p group_vars hosts playbooks roles

```
- group_vars
- hosts
- playbooks
- roles
ansible.cfg
```





## 기본 설정

> playbook을 사용하지 않는 명령어

```Sh
$ ansible all -i hosts/admin -m ping -u deploy
```

playbook 으로 변환

> vi playbooks/basic.yml

```yaml
---
- hosts: all
  tasks:
    - name: test connection
      ping:
```

> 실행

```Sh
$ ansible-playbook playbooks/basic.yml -i hosts/admin -l alpha
```





## 기본 설정 - 좀 더 테스트 해보자

각 서버에 접속하여 `/home/deploy` 에 `{본인이름}.txt touch` 하는 것을 구현해 보자  
touch는 file 모듈을 활용하면 된다.  File 모듈에 대해 자세히 보려면 [여기][1]를 참고하라.

[1]: http://docs.ansible.com/ansible/latest/file_module.html

> vi playbook/touch_files.yml

```Yaml
---
- hosts: all
  tasks:
    - name: make directory
      file:
        path: /home/deploy/touch_files
        state: directory
        
    - name: touch file
      file:
        path: /home/deploy/touch_files/jacob.txt
        state: touch
```

### 변수 활용 (vars, {{변수명}})

위에 설정을 보면 `/home/deploy/touch_files` 이 중복된다. 변수를 활용하면 깔끔하겠죠?  
변수를 사용할 때는 `{{variables}}` 형태로 사용하면 된다.

```yaml
---
- hosts: all
  vars:
    touch_files_path: /home/deploy/touch_files
    id: jacob
  tasks:
    - name: make directory
      file:
        path: "{{touch_files_path}}"
        state: directory
        
    - name: touch file
      file:
        path: "{{touch_files_path}}/{{id}}.txt"
        state: touch
```

### 루프를 이용하여 여러 파일들을 생성해 보자 (item, with_items)

`{본인이름}1~3.txt` 을 생성해 보자

```yaml
---
- hosts: all
  vars:
    touch_files_path: /home/deploy/touch_files
    id: jacob
  tasks:
    - name: make directory
      file:
        path: "{{touch_files_path}}"
        state: directory
        
    - name: touch file
      file:
        path: "{{touch_files_path}}/{{item}}.txt"
        state: touch
      with_items:
        - "{{id}}1"
        - "{{id}}2"
        - "{{id}}3"
```

or

```yaml
---
- hosts: all
  vars:
    touch_files_path: /home/deploy/touch_files
    id: jacob
  tasks:
    - name: make directory
      file:
        path: "{{touch_files_path}}"
        state: directory
        
    - name: touch file
      file:
        path: "{{touch_files_path}}/{{item.id}}{{item.num}}.txt"
        state: touch
      with_items:
        - {id: "{{id}}", num: 1}
        - {id: "{{id}}", num: 2}
        - {id: "{{id}}", num: 3}
```



### 조건문도 설정해볼까 (when)

> http://docs.ansible.com/ansible/latest/playbooks_conditionals.html 

CentOS 이며 버전이 7일 경우에만 실행되게 해보자

```Yaml
---
- hosts: all
  vars:
    touch_files_path: /home/deploy/touch_files
    id: jacob
  tasks:
    - name: make directory
      file:
        path: "{{touch_files_path}}"
        state: directory
        
    - name: touch file
      file:
        path: "{{touch_files_path}}/{{item}}.txt"
        state: touch
      with_items:
        - "{{id}}1"
        - "{{id}}2"
        - "{{id}}3"
      when:
        - ansible_distribution == "CentOS"
        - ansible_distribution_major_version == "7"
        #- (ansible_distribution == "CentOS" and ansible_distribution_major_version == "7")
```

파일이 존재하면 실행하고 없으면 실행되게 해보자

> Stat 모듈 : http://docs.ansible.com/ansible/latest/stat_module.html

```Yaml
---
- hosts: all
  vars:
    touch_files_path: /home/deploy/touch_files
    id: jacob
  tasks:
    - name: make directory
      file:
        path: "{{touch_files_path}}"
        state: directory
        
    - stat:
        path: "{{touch_files_path}}/jacob.txt"
       register: result
    
    - name: touch file
      file:
        path: "{{touch_files_path}}/{{item}}.txt"
        state: touch
      with_items:
        - "{{id}}1"
        - "{{id}}2"
        - "{{id}}3"
      when:
        - not result.stat.exists
        #- result.stat.exists == false
```





## Ansible의 핵심 Role

- 중복 소스 제거
- 자주 사용하는 것들은 함수로
- 미리 레시피를 만들어 놓고 호출만 하면 끝!

### 프로젝트 구조 예

```
site.yml
webservers.yml
fooservers.yml
roles/
   common/
     files/
     templates/
     tasks/
     handlers/
     vars/
     defaults/
     meta/
   webservers/
     files/
     templates/
     tasks/
     handlers/
     vars/
     defaults/
     meta/
```

호출은 이렇게

```yaml
---
- hosts: webservers
  roles:
    - common
    - webservers
```

변수나 조건 설정을 할 경우

```Yaml
---
- hosts: webservers
  roles:
    - {role: touch_files, touch_files_path: '/home/deploy/touch_files'}
    - {role: touch_files, touch_files_path: '/home/deploy/touch_files', when: "ansible_os_family == 'RedHat'"}
```

### Role 생성

위에서 진행했던 touch_files.yml에 있던 tasks를 role로 빼보자

> main.yml 은 index.html 같은 개념

```Sh
$ mkdir -p roles/touch_files/tasks/main.yml
```

> vi roles/touch_files/tasks/main.yml

```Yaml
---
- name: make directory
  file:
    path: "{{touch_files_path}}"
    state: directory
    
- stat:
    path: "{{touch_files_path}}/jacob.txt"
  register: result

- name: touch file
  file:
    path: "{{touch_files_path}}/{{item}}.txt"
    state: touch
  with_items:
    - "{{id}}1"
    - "{{id}}2"
    - "{{id}}3"
  when:
    - result.stat.exists == false
```

> vi playbooks/touch_files_role.yml

```yaml
---
- hosts: all
  vars:
    touch_files_path: /home/deploy/touch_files
    id: jacob
  roles:
    - touch_files
```

or

```yaml
---
- hosts: all
  roles:
    - {role: touch_files, touch_files_path: /home/deploy/touch_files, id: jacob}
```

> 실행

```Sh
$ ansible-playbook playbooks/touch_files_role.yml -i hosts/admin -l alpha
```



## Variables

***{{variables}}*** 형태로 사용

### 변수를 사용하는 방법들

#### 공통 파일로 관리

> group_vars/common.yml

```yaml
git:
  version: git-2.13.0
  download_url: https://www.kernel.org/pub/software/scm/git
nginx:
  version: nginx-1.12.1
  download_url: https://nginx.org/download/nginx-1.12.1.tar.gz
```

> playbooks/install_nginx.yml

```Yaml
---
- hosts: all
  vars_files:
    - ../../group_vars/common.yml
  roles:
  	- install_nginx
```

#### Hosts 에서 관리

> hosts/admin

```json
[web]
ansible-test-web01	nginx_version=nginx-1.12.1
ansible-test-web02  nginx_version=nginx-1.12.1

[db]
ansible-test-db01
```

#### Playbook에서 관리

> playbooks/install_nginx.yml

```yaml
---
- hosts: all
  vars:
    - nginx:
        - version: nginx-1.12.1
        - download_url: https://nginx.org/download/nginx-1.12.1.tar.gz
  roles:
    - install_nginx
```

#### Roles 의 vars 에서 관리

> roles/install_nginx/vars/main.yml

```Yams
- nginx:
  - version: nginx-1.12.1
  - download_url: https://nginx.org/download/nginx-1.12.1.tar.gz
```



## Playbook 안에 Playbook

> playbook/site.yml

```Yaml
---
- include: webservers.yml
- include: dbservers.yml
```

> webservers.yml

```yaml
---
- hosts: webservers
  roles:
    - common
    - webtier
```

> 실행

```Sh
$ ansible-playbook site.yml --limit webservers
$ ansible-playbook webservers.yml
```





## 참고 사이트

[Ansible 공식 Documentation][1]

[Ansible examples][2]

[1]: http://docs.ansible.com/ansible/latest/index.html
[2]: https://github.com/ansible/ansible-examples


