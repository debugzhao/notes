### Ansible相关文件

#### 配置文件

- /etc/ansible/ansible.cfg  主配置文件，用来配置ansible的工作特性
- /etc/ansible/host   主机清单文件，用来配置需要通过ansible来管理的主机
- /etc/ansible/roles/  存放角色的目录

#### Ansible相关工具

- /usr/bin/ansible 主程序，临时命令执行工具
- /usr/bin/ansible-doc 查看配置文档，模块功能查看工具
- /usr/bin/ansible-galaxy 下载/上传优秀代码或Roles模块的官网平台
- /usr/bin/ansible-playbook 定制自动化任务，编排剧本工具
- /usr/bin/ansible-pull 远程执行命令的工具
- /usr/bin/ansible-vault 文件加密工具
- /usr/bin/ansible-console 基于Console界面与用户交互的执行工具

##### Ansible

此工具通过ssh协议，实现对远程主机的配置管理、应用部署、任务执行等功能

建议：在使用此工具之前，首先配置ansible主控端能基于秘钥认证的方式联系被管理的各个节点

范例：通过sshpass实现批量基于key的验证

```shell
# 生成公钥
ssh-keygen

# 拷贝公钥到某一台服务器上
ssh-copy-id 172.20.38.71
```

**Ansible命令的选项说明**

```shell
--version              # 显示版本
-m moudle              # 显示指定模块
-v                     # 详细过程 -vv -vvv更详细过程
--list-hosts           # 显示被控端主机列表
-K, --ask-pass         # 提示输入ssh连接密码，默认key验证
-C, --check            # 检查，并不执行
-T, --timeout=TIMEOUT  # 执行命令超时时间，默认10s
-u, --user=REMOTE_USER # 执行远程执行的用户
-b, --become           # 替代旧版的sudo 切换
-K, --ask-become-pass  # 提示输入sudo时的口令
```

**Ping命令使用**

```shell
[root@ansible .ssh]# ansible all -m ping
172.20.38.17 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
172.20.38.71 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
172.20.18.165 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```

**查看ansible正在管理的机器**

```shell
[root@ansible .ssh]# ansible all --list-hosts
  hosts (3):
    172.20.18.165
    172.20.38.71
    172.20.38.17
[root@ansible .ssh]# ansible appservers  --list-hosts
  hosts (3):
    172.20.38.71
    172.20.38.17
    172.20.18.165
```

**通配符**

```shell
ansible "*" -m ping
```

**或关系**

```shell
ansible "webservers:appservers" -m ping
```

**逻辑与**

```shell
# 在webservers清单中并且还在appservers清单中的主机
ansible "webservers:&appservers" -m ping
```

**逻辑非**

```shell
# 在webservers清单中但是不在appservers清单中的主机
# 注意：此处用单引号
ansible 'webservers:&appservers' -m ping
```

##### ansible-galaxy

此工具会从http://galaxy.ansible.com下载相应的roles

```shell
# 列出所有已经安装的galaxy
ansible-galaxy list
# 安装galaxy
ansible-galaxy install geerlingguy
# 删除galaxy
ansible-galaxy remove geerlingguy
```

##### ansible-pull

此工具会推送ansible命令至远程，效率无限提升， 但是对运维要求较高

##### ansible-playbook

此工具用于编写playbool任务

**yml文件示例：**

```yaml
---
- hosts: webservers
  remote_user: root
  tasks:
    - name: hello word
      command: /usr/bin/wall hello world
```

调用yml文件：

```shell
ansible-playbook hello.yml
```

##### ansible-vault

此工具用于加密/解密yml文件

```shell
# 加密文件
ansible-vault encrypt hello.yml
# 解密文件
ansible-vault decrypt hello.yml
```

##### ansible-console

此工具可以通过交互方式执行命令

```shell
[root@ansible ~]# ansible-console
Welcome to the ansible console.
Type help or ? to list commands.

root@all (3)[f:5]$ list
172.20.18.165
172.20.38.71
172.20.38.17d
root@appservers (3)[f:5]$ cd dbservers
root@dbservers (1)[f:5]$ list
172.20.18.165
root@dbservers (1)[f:5]$ forks
Usage: forks <number>
root@dbservers (1)[f:5]$ forks 10
```

#### Ansible常用模块详解

##### Command模块

在远程主机执行命令，command为默认模块，可以忽略-m选项

```shell
# 查看webservers主机列表的操作系统信息
ansible webservers -m command -a 'cat /etc/centos-release'

# 先切换目录，再查看操作系统版本信息
ansible webservers -m command -a 'chdir=/etc cat centos-release'
```

##### Shell模块

shell模块和command模块功能类似，支持一些特殊字符

```shell
[root@ansible ~]# ansible webservers -m shell -a 'echo $HOSTNAME'
172.20.38.71 | CHANGED | rc=0 >>
71instance
172.20.18.165 | CHANGED | rc=0 >>
localhost.localdomain
```

##### Script模块

在远程主机上运行ansible服务器上的脚本文件

test.sh脚本文件

```shell
#!/bin/bash
echo myhostname is `hostname`
```

```shell
[root@ansible data]# ansible webservers  -m script -a '/data/test.sh'
172.20.18.165 | CHANGED => {
    "changed": true,
    "rc": 0,
    "stderr": "Shared connection to 172.20.18.165 closed.\r\n",
    "stderr_lines": [
        "Shared connection to 172.20.18.165 closed."
    ],
    "stdout": "myhostname is localhost.localdomain\r\n",
    "stdout_lines": [
        "myhostname is localhost.localdomain"
    ]
}
172.20.38.71 | CHANGED => {
    "changed": true,
    "rc": 0,
    "stderr": "Shared connection to 172.20.38.71 closed.\r\n",
    "stderr_lines": [
        "Shared connection to 172.20.38.71 closed."
    ],
    "stdout": "myhostname is 71instance\r\n",
    "stdout_lines": [
        "myhostname is 71instance"
    ]
}
```

##### Copy模块

从ansible主控端复制文件到远程主机

##### Fetch模块

从远程主机提取文件到主控端，目前不支持目录

```shell
ansible all -m fetch -a "src=/etc/redhat-release dest=/data/os"

[root@ansible os]# tree
.
├── 172.20.18.165
│   └── etc
│       └── redhat-release
├── 172.20.38.17
│   └── etc
│       └── redhat-release
└── 172.20.38.71
    └── etc
        └── redhat-release

6 directories, 3 files
```

##### File模块

设置文件属性

```shell
# 创建文件
ansible webservers -m file -a "path=/data/test.txt state=touch"

# 更改权限和所属组
ansible webservers -m file -a "path=/data/test.txt group=bin mode=600"

# 查看文件详情
ansible webservers  -a "ls -l /data/test.txt"
```

##### Unarchive模块

**功能：**压缩包解压缩

**实现方式：**

1. 将ansible主机上的压缩包上传至远程主机的指定目录，然后解压缩，设置copy=yes
2. 将远程主机的某个压缩包解压到指定目录，设置copy=no

```shell
ansible webservers  -m unarchive -a 'src=/data/etc.tar.gz dest=/data copy=yes'
```

##### Archive模块

**功能：**压缩文件

##### Hostname模块

**功能：**管路目标主机的主机名

```shell
ansible 172.20.38.17  -m hostname -a 'name=17instance
```

##### Cron模块

![image-20210606102502583](https://i.loli.net/2021/06/06/M3t2S4YnNHwgeyb.png)

##### Yum模块

**功能：**安装软件

```shell
# 软件安装
ansible all  -m yum -a 'name=httpd'
# 软件卸载
ansible all  -m yum -a 'name=httpd state=absent'
```

##### Service模块

**功能：** 启动/关闭服务

```shell
# 查看主机打开了哪些端口
ansible all -a 'ss -ntl'

# 启动httpd服务
ansible all -m service -a 'name=httpd state=started' 

# 停止httpd服务
ansible all -m service -a 'name=httpd state=stopped' 

# 开机启动httpd服务
ansible all -m service -a 'name=httpd state=started enabled=yes' 
```

##### User模块

**功能：**管理用户

```shell
# 创建用户
ansible all -m user -a 'name=nginx uid=89 groups="nginx" system=yes shell=/sbin/nologin create_home=no home=/data/nginx non_unique=yes'

# 删除用户以及家目录等数据
ansible all -m user -a 'name=nginx state=absent remove=yes'
```

##### Group模块

**功能：**管理组

```shell
# 创建组
ansible all -m group -a 'name=nginx gid=88 system=yes'

# 删除组
ansible all -m group -a 'name=nginx state=absent'
```

##### Lineinfile模块

**功能：**该功能sed命令类似，可以通过正则表达式来替换文件内容

##### Replace模块

**功能：**该功能sed命令类似，可以通过正则表达式来替换文件内容

##### Setup模块

**功能：**收集远程主机信息

```shell
ansible 172.20.18.165 -m setup
```

```shell
# 过滤特定键
[root@ansible data]# ansible all -m setup -a 'filter=ansible_distribution_major_version'
172.20.38.71 | SUCCESS => {
    "ansible_facts": {
        "ansible_distribution_major_version": "7",
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false
}
172.20.18.165 | SUCCESS => {
    "ansible_facts": {
        "ansible_distribution_major_version": "7",
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false
}
172.20.38.17 | SUCCESS => {
    "ansible_facts": {
        "ansible_distribution_major_version": "7",
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false
}
```

#### PlayBook

##### playbook介绍

![image-20210606135515598](https://i.loli.net/2021/06/06/U17cV6esakvGJuP.png)

##### playbook核心组件

- hosts：远程操作的主机列表
- task：任务集
- variables：playbook中调用的内置变量或者自定义的变量
- templates：模板
- handlers：和notify结合使用，在特定条件下才触发的操作
- tags：标签 指定某条任务执行，用以选择运行执行playbook中的部分代码

##### hosts远程主机列表组件

```yaml
- hosts: webservers:appservers
```

##### remote_user组件

remote_user可以用在host和task标签中，用来表示通过什么用户来执行指令

```yaml
- hosts: webservers:appservers
  remote_user: root
```

##### task列表和action组件

```yaml
---
- hosts: webservers
  remote_user: root
  tasks:
    - name: install httpd # 安装httpd服务(该键值对为描述性语言，没有实际意义)
      yum: name=httpd
    - name: start httpd   # 启动httpd服务
      service: name=httpd state=started enabled=yes
```

##### playbook命令

格式

```shell
ansible-playbook <filename.yml> ... [options]
```

常见选项

```yaml
-C --check      # 只检查可能会发生的改变，但是不执行真正的操作
--list-hosts    # 列出运行任务的主机
--list-tags     # 列出tag
--list-task     # 列出task
--limit 主机列表 # 只针对主机列表中的主机执行任务
-v -vv -vvv     # 显示过程
```

范例

```shell
ansible-playbook mysql_user.yml --check   # 只做检查
ansible-playbook mysql_user.yml --limit  172.20.18.164
```

#### PlayBook实战

##### 利用playbook创建mysql账户

```shell
---
# create mysql user and group

- hosts: appservers
  remote_user: root
  gather_facts: no

  tasks:
    - name: create group
      group: name=mysql system=yes gid=306
    - name: create user
      user: name=mysql system=yes group=mysql shell=/sbin/nologin create_home=no home=/data/mysql uid=306
```

```shell
# 预检查yml文件是否有语法错误
ansible-playbook -C mysql_user.yml

# 执行yml task
ansible-playbook mysql_user.yml
```

##### 利用playbook安装并启动nginx

```yaml
---
# install nginx
- hosts: localservers
  remote_user: root
  tasks:
    - name: add group nginx
      user: name=nginx state=present
    - name: add user nginx
      user: name=nginx state=present group=nginx
    - name: Install Nginx
      yum: name=nginx state=present
    - name: change index page
      copy: src=index_page/index.html dest=/usr/share/nginx/html/index.html
    - name: Start Nginx
      service: name=nginx state=started enabled=yes
```

##### playbook中使用handlers和notify

handlers的本质是task list，类似于MySQL触发器触发事件的行为，其中task主要关注当资源发生变化时，才会采取一定的操作。

而notify中定义的action可以用在在每个play最后才被触发，这样可以避免又多次发生改变时，每次都会触发handlers，仅在所有变化完成后一次性地执行操作。

在notify中列出来的操作被称为handler，即notify调用handler中的操作

**httpd.yml配置文件**

```shell
# 修改files/httpd.conf配置文件，实现触发handler的功能

---
# http service yml

- hosts: webservers
  remote_user: root
  gather_facts: no

  tasks:
    - name: install httpd
      yum: name=httpd state=present
    - name: install configure file
      copy: src=files/httpd.conf dest=/etc/httpd/conf
      notify: restart httpd service
    - name: ensure apache is running
      service: name=httpd state=started enabled=yes

  handlers:
    - name: restart httpd service
      service: name=httpd state=restarted
```

**执行yml文件**

```shell
# 执行yml文件
ansible-playbook  httpd.yml

# 查看httpd开放的端口
ansible webservers -a 'ss -ntl | grep 8888'
```

##### playbook中使用tags组件

在playbook文件中，可以利用tags组件，为特定的task指定标签。当在执行playbook时，可以只执行特定的tags，而非整个playbook文件

```yaml
---
# http service yml

- hosts: webservers
  remote_user: root
  gather_facts: no

  tasks:
    - name: install httpd
      yum: name=httpd state=present
    - name: install configure file
      copy: src=files/httpd.conf dest=/etc/httpd/conf
      tags: conf
    - name: start httpd service
      tags: service
      service: name=httpd state=started enabled=yes
```

**选择指定标签执行**

```shell
ansible-playbook -t conf,service httpd_tags.yml
```

#### PlayBook中变量使用

##### 使用setup模块中的变量

```yaml
---
- hosts: webservers
  remote_user: root
  gather_facts: no

  tasks:
    - name: create log file
      file: name=/var/log/{{ansible_distribution}}.log state=touch
```

##### 在playbook命令行中定义变量

##### 在playbook文件中定义变量

```yaml
[root@ansible playbook]# cat var2.yml
---
- hosts: webservers
  remote_user: root
  gather_facts: no

  vars:
    - username: user1
    - groupname: group1

  tasks:
    - name: create group
      group: name={{ groupname  }} state=present
    - name: create user
      user: name={{ username  }} group={{ groupname  }} state=present
```

##### 使用变量文件

**变量文件**

```yaml
[root@ansible playbook]# cat vars.yml
---
# variables file
package_name: vsftpd
service_name: vsftpd
```

**playbook文件**

```yaml
[root@ansible playbook]# cat install_ftpd.yml
---
# install vsftpd
- hosts: webservers
  remote_user: root

  vars_files:
    - vars.yml

  tasks:
    - name: install package
      yum: name={{ package_name  }}
      tags: install
    - name: start service
      service: name={{ service_name  }} state=started enabled=yes
```

##### 主机清单文件中定义变量

#### template模板

![image-20210607100701033](https://i.loli.net/2021/06/07/wqBP5lC7NmJtKby.png)

##### 





