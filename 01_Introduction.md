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

## Inventory

### Hosts and Groups

기본 설정

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

SSH 접속 관련 파라미터 설정  
http://docs.ansible.com/ansible/intro_inventory.html#list-of-behavioral-inventory-parameters

```
jumper ansible_port=5555 ansible_host=192.0.2.50
```

범위 설정

```
[webservers]
www[01:50].example.com

[databases]
db-[a:f].example.com
```

### Host Variables

```
[real]
aaa.example.com           project=web phase=real
bbb.example.com           project=web phase=real
ccc.example.com           project=web phase=real
```

### Group Variables

```
[real]
aaa.example.com
bbb.example.com
ccc.example.com

[real:vars]
project=web
phase=alpha
```

퀴즈. 현재 선물하기 ansible 설정에서 group으로 변수 지정을 하지 않는 이유는?

### Groups of Groups, and Group Variables

키워드  
***:children***, ***:vars***

```
[web-1]
a1.example.com
a2.example.com
a3.example.com

[web-2]
b1.example.com
b2.example.com
b3.example.com

[result]
c1.example.com
c2.example.com

[real:children]
web-1
web-2
result
```

## 실행

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

사용 예

```shell
# webapp real 서버군에서 ERROR 로그 검색
ansible all -i hosts/webapp -l real -m shell -a "grep ERROR /home/ansible/logs/webapp/application.log" -f 10
```

## 참고자료

http://deview.kr/2014/session?seq=15
