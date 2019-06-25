- 本文作者:  wuXing
- QQ: 1226032602
- E-mail: 1226032602@qq.com

[http://docs.ansible.com/ansible/intro_installation.html](http://docs.ansible.com/ansible/intro_installation.html)
[http://docs.ansible.com/modules_by_category.html](http://docs.ansible.com/modules_by_category.html)
http://docs.ansible.com/ansible/authorized_key_module.html
# 简介
  自动化运维工具ansible
  ansible基于python开发的自动化运维工具(saltstack)
  python语言是运维人员最佳的语言
  其功能实现基于SSH远程连接服务
  批量系统配置、批量软件部署、批量文件拷贝、批量运行命令等功能
# 特点
1、不需要单独安装客户端，基于sshd服务的，sshd就相当于ansible的客户端
2、不需要服务端
3、依靠大量的模块实现
# 管理机安装
```bash
yum install epel-release  -y
yum install ansible -y
```
# 客户端安装（如果selinux开启则安装）
```bash
yum install libselinux-python -y
```
# 查看
```bash
[root@m01 ~]# ansible --version
ansible 2.7.8
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.5 (default, Oct 30 2018, 23:45:53) [GCC 4.8.5 20150623 (Red Hat 4.8.5-36)]
```
# ansible配置文件
```bash
[root@m01 ~]# tree /etc/ansible/
/etc/ansible/
├── ansible.cfg
├── hosts
└── roles

1 directory, 2 files
```
# ansible配置文件读取顺序
* ANSIBLE_CONFIG (如果设置了环境变量)
* ansible.cfg (在当前目录中)
* ~/.ansible.cfg (在家目录中)
* /etc/ansible/ansible.cfg
# 管理机配置
vim  /etc/ansible/hosts
```
[oldboy]
172.16.1.31
172.16.1.41
172.16.1.61
172.16.1.7
172.16.1.8

[oldboy]
172.16.1.31 ansible_ssh_user=oldboy ansible_ssh_pass=123456 ansible_ssh_port=12345
172.16.1.41 ansible_ssh_user=oldboy ansible_ssh_pass=123456 ansible_ssh_port=12345
```
# 查看主机列表
```
ansible web --list-host
```
# ansible配置sudo
ansible及客户端都添加sudo授权用户
```
/etc/ansible/ansible.cfg
deprecation_warnings=False
sudo_user      = oldboy
remote_port      = 52113
```
# 内置变量
```
ansible_connection
连接类型到主机。这可以是任何ansible连接插件的名称。SSH协议类型是smart，ssh或paramiko。默认是智能的。基于非SSH的类型将在下一节中介绍。

ansible_host   要连接的主机的名称，如果与您希望提供给它的别名不同。
ansible_port   ssh端口号，如果不是22
ansible_user   要使用的默认ssh用户名。

ansible_ssh_host：   ansible使用ssh要连接的主机。
ansible_ssh_port：   ssh的端口。默认为22。
ansible_ssh_user：   ssh登录的用户名。默认为root。

ansible_ssh_pass
要使用的ssh密码（永远不要将此变量存储为纯文本;始终使用保险库。请参阅变量和保险柜）

ansible_ssh_private_key_file
ssh使用的私钥文件。如果使用多个密钥并且您不想使用SSH代理，则很有用。

ansible_ssh_common_args
此设置始终附加到sftp，scp和ssh的默认命令行。ProxyCommand用于为特定主机（或组）配置

ansible_sftp_extra_args  此设置始终附加到默认的sftp命令行。
ansible_scp_extra_args  此设置始终附加到默认scp命令行。
ansible_ssh_extra_args  此设置始终附加到默认的ssh命令行。

ansible_ssh_pipelining  确定是否使用SSH流水线。这可以覆盖中的pipelining设置ansible.cfg。

ansible_ssh_executable（在2.2版中添加）
此设置将覆盖使用系统ssh的默认行为。这可以覆盖中的ssh_executable设置ansible.cfg。

ansible_become  等同于ansible_sudo或ansible_su允许强制权限升级
ansible_become_method   允许设置权限提升方法

ansible_become_user  等同于ansible_sudo_user或ansible_su_user允许通过权限提升设置您成为的用户

ansible_become_pass   等效于ansible_sudo_pass或ansible_su_pass允许您设置权限提升密码（永远不要将此变量存储为纯文本;始终使用保管库。请参阅变量和保管库）

ansible_become_exe   等效于ansible_sudo_exe或ansible_su_exe允许您为所选的升级方法设置可执行文件

ansible_become_flags   等效于ansible_sudo_flags或ansible_su_flags允许您设置传递给选定升级方法的标志。这也可以ansible.cfg在sudo_flags选项中全局设置

ansible_shell_type
目标系统的shell类型。除非已将ansible_shell_executable设置为非Bourne（sh）兼容shell，否则不应使用此设置 。默认情况下，命令使用sh-style语法格式化。将此设置为csh或fish将导致在目标系统上执行的命令遵循这些shell的语法。

ansible_python_interpreter
目标主机python路径。这对于具有多个Python或不位于/ usr / bin / python的系统（如* BSD）或/ usr / bin / python 不是2.X系列Python的系统非常有用。我们不使用/ usr / bin / env机制，因为它需要设置远程用户的路径，并假设python可执行文件名为python，其中可执行文件可能被命名为python2.6。

ansible_shell_executable
这将设置ansible控制器将目标机器上使用的壳，将覆盖executable在ansible.cfg其中默认为 / bin / sh的。如果无法使用/ bin / sh（即/ bin / sh未安装在目标计算机上或无法从sudo运行），您应该只更改它。
```
# 普通用户批量管理
## 管理机与所有被管理机都创建普通用户并sudo授权
```
wuxingge      ALL=(ALL)       NOPASSWD: ALL
```
## 管理机普通用户创建密钥对并分发公钥
## ansible配置
/etc/ansible/ansible.cfg
```
sudo_user      = wuxingge
deprecation_warnings = False
private_key_file = /home/wuxingge/.ssh/id_rsa
```
## 测试
```
ansible web -m copy -a "src=/etc/hosts dest=/root/" -s
ansible-playbook test.yml -s
```
# ansible软件颜色信息
- 绿色：  表示查看信息，对远程主机未做改动的命令
- 红色：  批量管理产生错误信息
- 黄色：  对远程主机做了相应改动
- 粉色：  对操作提出建议或忠告
# hosts
[https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)
/etc/ansible/hosts
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
```yml
###################yaml
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
```
jumper ansible_port=5555 ansible_host=192.0.2.50
```
```yml
############# yaml
  hosts:
    jumper:
      ansible_port: 5555
      ansible_host: 192.0.2.50
```
## children
```
[backup:children]
backupclient
backupserver
[backupclient]
172.16.1.7
172.16.1.31
[backupserver]
172.16.1.41
```
```
[atlanta]
host1
host2

[raleigh]
host2
host3

[southeast:children]
atlanta
raleigh

[southeast:vars]
some_server=foo.southeast.example.com
halon_system_timeout=30
self_destruct_countdown=60
escape_pods=2

[usa:children]
southeast
northeast
southwest
northwest
```
```yml
############ yaml
all:
  children:
    usa:
      children:
        southeast:
          children:
            atlanta:
              hosts:
                host1:
                host2:
            raleigh:
              hosts:
                host2:
                host3:
          vars:
            some_server: foo.southeast.example.com
            halon_system_timeout: 30
            self_destruct_countdown: 60
            escape_pods: 2
        northeast:
        northwest:
        southwest:
```
## 主机变量(vars)
```
[atlanta]
host1
host2

[atlanta:vars]
ntp_server=ntp.atlanta.example.com
proxy=proxy.atlanta.example.com
```
```yml
################yaml
atlanta:
  hosts:
    host1:
    host2:
  vars:
    ntp_server: ntp.atlanta.example.com
    proxy: proxy.atlanta.example.com
```
# hosts (inventory)
vim  ansible.cfg 
```
...
inventory      = /etc/ansible/inventory/
...
```
```
cat inventory/part1
[nginx]
172.16.1.7

cat inventory/part2
[common]
172.16.1
```
# 按目录结构存储变量
> group_vars/和**host_vars/目录可以存在于playbooks目录或inventory目录中。如果两个路径都存在，则playbook目录中的变量将覆盖inventory目录中设置的变量。
相关的Host和Group变量可以放在/etc/ansible/host_vars/和/etc/ansible/group_vars/下的主机同名目录中的文件，也支持'.yml'、'.yaml'和'.json'为后缀的YMAL和JSON文件

[图片上传失败...(image-7de1dd-1560937738186)]
> group_vars目录下的文件定义Roles中调用的变量
文件名为all的文件定义的变量针对所有Roles生效

```
tree inventory
inventory
├── group_vars
│   ├── all
│   └── nginx
├── part1
└── part2

cat inventory/part1 
[nginx]
172.16.1.7

cat inventory/part2
[common]
172.16.1.8

cat inventory/group_vars/nginx 
nginx_port: 8080

cat inventory/group_vars/all 
nginx_port: 8888
```
```
cat inventory/hosts_vars/127.0.0.1
---
myname: wuxing
address: beijing
```
```
cat playbooks/nginx.yml 
- hosts: all
  roles:
    - nginx
```
```bash
ansible-playbook -i inventory playbooks/nginx.yml
```
```
inventory/
  01-openstack.yml          # configure inventory plugin to get hosts from Openstack cloud
  02-dynamic-inventory.py   # add additional hosts with dynamic inventory script
  03-static-inventory       # add static hosts
  group_vars/
    all.yml                 # assign variables to all hosts
```
---
# 模块
```
ansible   <host-pattern>   [-m module_name]   [-a args]  [options]
```
## command模块
oldboy为主机组的名字
-m 后边是模块名
 -m MODULE_NAME, --module-name=MODULE_NAME
                        module name to execute (default=command)
默认的模块是command

-a 后面是要执行的命令，也可以写一个ip，针对一台机器来执行命令
-a MODULE_ARGS, --args=MODULE_ARGS
                        module arguments
```
ansible oldboy -m command -a 'uptime'
```
```yml
- name: return motd to registered var
  command: cat /etc/motd
```
```
ansible oldboy -m command -a "chdir=/tmp touch hosts01"
ansible oldboy -m command -a "creates=/tmp/hosts01 touch hosts01"
```
## shell模块
http://docs.ansible.com/ansible/latest/shell_module.html
执行脚本（脚本需要分发到远程机器）
```bash
[root@m01 ~]# echo 'yum install -y ipvsadm' >> /server/scripts/yum.sh
[root@m01 ~]# cat /server/scripts/yum.sh
yum install -y ipvsadm
ansible oldboy -m copy -a "src=/server/scripts/yum.sh dest=/server/scripts/ mode=0755"
ansible oldboy -m shell -a "/bin/bash /server/scripts/yum.sh"
```
```yml
- name: Execute the command in remote shell; stdout goes to the specified file on the remote.
  shell: somescript.sh >> somelog.txt

- name: Change the working directory to somedir/ before executing the command.
  shell: somescript.sh >> somelog.txt
  args:
    chdir: somedir/

# You can also use the 'args' form to provide the options.
- name: This command will change the working directory to somedir/ and will only run when somedir/somelog.txt doesn't exist.
  shell: somescript.sh >> somelog.txt
  args:
    chdir: somedir/
    creates: somelog.txt

- name: Run a command that uses non-posix shell-isms (in this example /bin/sh doesn't handle redirection and wildcards together but
  shell: cat < /tmp/*txt
  args:
    executable: /bin/bash
```
## ping模块
[https://docs.ansible.com/ansible/latest/modules/ping_module.html#ping-module](https://docs.ansible.com/ansible/latest/modules/ping_module.html#ping-module)
```bash
ansible oldboy -m ping
```
## copy模块
[https://docs.ansible.com/ansible/latest/modules/copy_module.html#copy-module](https://docs.ansible.com/ansible/latest/modules/copy_module.html#copy-module)
copy模块将文件从本地复制到远程计算机上的某个位置
```
ansible oldboy -m copy -a "src=/etc/passwd dest=/tmp/oldgirl.txt owner=oldboy group=oldboy mode=0755"
```
ansible copy模块 会自动创建多级目录
```
[root@m01 scripts]# ansible oldboy -m copy -a "src=/etc/hosts  dest=/tmp/a/b/c/e/d/f/f/g/"
```
目标目录都存在的情况下，可以传输过去同时修改文件名
```
ansible oldboy -m copy -a "src=/etc/hosts dest=/tmp/HOSTS"
ansible oldboy -m copy -a "src=/etc/hosts dest=/mnt backup=yes"
```
backup=yes  备份
# Example from Ansible Playbooks
```yml
- copy:
    src: /srv/myfiles/foo.conf
    dest: /etc/foo.conf
    owner: foo
    group: foo
    mode: 0644
- copy:
    content: xxxxxxxxxxx
    dest: /some/path/systems.csv
    owner: foo
    group: foo
    mode: 0644
    backup: yes

- name: Copy a new "ntp.conf file into place, backing up the original if it differs from the copied version
  copy:
    src: /mine/ntp.conf
    dest: /etc/ntp.conf
    owner: root
    group: root
    mode: '0644'
    backup: yes

- name: Copy a "sudoers" file on the remote machine for editing
  copy:
    src: /etc/sudoers
    dest: /etc/sudoers.edit
    remote_src: yes
    validate: /usr/sbin/visudo -cf %s
```
## remote_src
```
ansible all -m copy -a 'src=/root/file1 dest=/opt/'
#如果src指定的文件在远程机器存在,则拷贝远程机器此文件到dest
ansible all -m copy -a 'src=/opt/file1 dest=/opt/file1.bak remote_src=yes'
```
## template
[https://docs.ansible.com/ansible/latest/modules/template_module.html#template-module](https://docs.ansible.com/ansible/latest/modules/template_module.html#template-module)
```yml
# Example from Ansible Playbooks
- template:
    src: /mytemplates/foo.j2
    dest: /etc/file.conf
    owner: bin
    group: wheel
    mode: 0644

# The same example, but using symbolic modes equivalent to 0644
- template:
    src: /mytemplates/foo.j2
    dest: /etc/file.conf
    owner: bin
    group: wheel
    mode: "u=rw,g=r,o=r"

# Create a DOS-style text file from a template
- template:
    src: config.ini.j2
    dest: /share/windows/config.ini
    newline_sequence: '\r\n'

# Copy a new "sudoers" file into place, after passing validation with visudo
- template:
    src: /mine/sudoers
    dest: /etc/sudoers
    validate: '/usr/sbin/visudo -cf %s'

# Update sshd configuration safely, avoid locking yourself out
- template:
    src: etc/ssh/sshd_config.j2
    dest: /etc/ssh/sshd_config
    owner: root
    group: root
    mode: '0600'
    validate: /usr/sbin/sshd -t -f %s
    backup: yes
```
## fetch 模块
从远程节点获取文件
```yml
    fetch：
      src： /tmp/uniquefile
      dest： /tmp/special/
      flat： yes
```
```
ansible 10.0.0.41 -m fetch -a "src=/etc/rsyncd.conf dest=/etc/ansible/conf/rsync/ flat=yes"
```
## script 模块（不需要分发脚本）
[https://docs.ansible.com/ansible/latest/modules/script_module.html](https://docs.ansible.com/ansible/latest/modules/script_module.html)
```bash
echo 'mkdir -p /tmp/oldboy/{a..d}' > /server/scripts/mkdir.sh
ansible oldboy -m script -a "/server/scripts/mkdir.sh"
cat >>/server/scripts/yuan.sh<<EOF
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-6.repo
EOF
ansible oldboy -m script -a "/server/scripts/yuan.sh"
```
```yml
- name: Run a script with arguments
  script: /some/local/script.sh --some-argument 1234

- name: Run a script only if file.txt does not exist on the remote node
  script: /some/local/create_file.sh --some-argument 1234
  args:
    creates: /the/created/file.txt

- name: Run a script only if file.txt exists on the remote node
  script: /some/local/remove_file.sh --some-argument 1234
  args:
    removes: /the/removed/file.txt

- name: Run a script using an executable in a non-system path
  script: /some/local/script
  args:
    executable: /some/remote/executable

- name: Run a script using an executable in a system path
  script: /some/local/script.py
  args:
    executable: python3
```
## cron模块
[https://docs.ansible.com/ansible/latest/modules/cron_module.html#cron-module](https://docs.ansible.com/ansible/latest/modules/cron_module.html#cron-module)
```
 ansible oldboy -m cron -a 'name=hello minute=*/2 job="echo hello >>/tmp/oldboy.log 2>&1" '
```
```yml
- name: Ensure a job that runs at 2 and 5 exists. Creates an entry like "0 5,2 * * ls -alh > /dev/null"
  cron:
    name: "check dirs"
    minute: "0"
    hour: "5,2"
    job: "ls -alh > /dev/null"

- name: 'Ensure an old job is no longer present. Removes any job that is prefixed by "#Ansible: an old job" from the crontab'
  cron:
    name: "an old job"
    state: absent

- name: Creates an entry like "@reboot /some/job.sh"
  cron:
    name: "a job for reboot"
    special_time: reboot
    job: "/some/job.sh"

- name: Creates an entry like "PATH=/opt/bin" on top of crontab
  cron:
    name: PATH
    env: yes
    value: /opt/bin

- name: Creates an entry like "APP_HOME=/srv/app" and insert it after PATH declaration
  cron:
    name: APP_HOME
    env: yes
    value: /srv/app
    insertafter: PATH

- name: Creates a cron file under /etc/cron.d
  cron:
    name: yum autoupdate
    weekday: 2
    minute: 0
    hour: 12
    user: root
    job: "YUMINTERACTIVE=0 /usr/sbin/yum-autoupdate"
    cron_file: ansible_yum-autoupdate

- name: Removes a cron file from under /etc/cron.d
  cron:
    name: "yum autoupdate"
    cron_file: ansible_yum-autoupdate
    state: absent

- name: Removes "APP_HOME" environment variable from crontab
  cron:
    name: APP_HOME
    env: yes
    state: absent
```
disabled=yes   注释定时任务
## mount模块
[https://docs.ansible.com/ansible/latest/modules/mount_module.html#mount-module](https://docs.ansible.com/ansible/latest/modules/mount_module.html#mount-module)
```yml
Control active and configured mount points
This module controls active and configured mount points in /etc/fstab

# Before 2.3, option 'name' was used instead of 'path'
- name: Mount DVD read-only
  mount:
    path: /mnt/dvd
    src: /dev/sr0
    fstype: iso9660
    opts: ro
    state: present

- name: Mount up device by label
  mount:
    path: /srv/disk
    src: LABEL=SOME_LABEL
    fstype: ext4
    state: present

- name: Mount up device by UUID
  mount:
    path: /home
    src: UUID=b3e48f45-f933-4c8e-a700-22a159ec9077
    fstype: xfs
    opts: noatime
    state: present
```
```yml
    - name: mount
      mount:
        path: /mnt
        src: 10.0.0.31:/data
        fstype: nfs
        state: mounted
```
## file模块
[https://docs.ansible.com/ansible/latest/modules/file_module.html](https://docs.ansible.com/ansible/latest/modules/file_module.html)
```yml
# change file ownership, group and mode
- file:
    path: /etc/foo.conf
    owner: foo
    group: foo
    # when specifying mode using octal numbers, add a leading 0
    mode: 0644
- file:
    path: /work
    owner: root
    group: root
    mode: 01777
- file:
    src: /file/to/link/to
    dest: /path/to/symlink
    owner: foo
    group: foo
    state: link
- file:
    src: '/tmp/{{ item.src }}'
    dest: '{{ item.dest }}'
    state: link
  with_items:
    - { src: 'x', dest: 'y' }
    - { src: 'z', dest: 'k' }

# touch a file, using symbolic modes to set the permissions (equivalent to 0644)
- file:
    path: /etc/foo.conf
    state: touch
    mode: "u=rw,g=r,o=r"

# touch the same file, but add/remove some permissions
- file:
    path: /etc/foo.conf
    state: touch
    mode: "u+rw,g-wx,o-rwx"

# touch again the same file, but dont change times
# this makes the task idempotents
- file:
    path: /etc/foo.conf
    state: touch
    mode: "u+rw,g-wx,o-rwx"
    modification_time: "preserve"
    access_time: "preserve"

# create a directory if it doesn't exist
- file:
    path: /etc/some_directory
    state: directory
    mode: 0755
```
## setup  （ 收集远程主机信息）
[https://docs.ansible.com/ansible/latest/modules/setup_module.html#setup-module](https://docs.ansible.com/ansible/latest/modules/setup_module.html#setup-module)
```
ansible all -m setup -vvvv
ansible all -m setup -a "filter=ansible_all_ipv4_addresses"
ansible all -m setup -a "filter=ansible_eth0[ipv4]"
```
```
ansible_all_ipv4_addresses：仅显示ipv4的信息。
ansible_devices：仅显示磁盘设备信息。
ansible_distribution：显示是什么系统，例：centos,suse等。
ansible_distribution_major_version：显示是系统主版本。
ansible_distribution_version：仅显示系统版本。
ansible_machine：显示系统类型，例：32位，还是64位。
ansible_eth0：仅显示eth0的信息。
ansible_hostname：仅显示主机名。
ansible_kernel：仅显示内核版本。
ansible_lvm：显示lvm相关信息。
ansible_memtotal_mb：显示系统总内存。
ansible_memfree_mb：显示可用系统内存。
ansible_memory_mb：详细显示内存情况。
ansible_swaptotal_mb：显示总的swap内存。
ansible_swapfree_mb：显示swap内存的可用内存。
ansible_mounts：显示系统磁盘挂载情况。
ansible_processor：显示cpu个数(具体显示每个cpu的型号)。
ansible_processor_vcpus：显示cpu个数(只显示总的个数)。
```
## debug
```
ansible 172.16.1.7 -m debug -a "var=ansible_eth1.ipv4.address"
```
## 帮助
```
ansible-doc -l            列出所有的模块
ansible-doc service      查看指定模块用法
```
问题：
```bash
[root@m01 ~]# ansible-doc -l
[DEPRECATION WARNING]: docker is kept for backwards compatibility but usage is discouraged.
 The module documentation details page may explain more about this rationale..
This feature
 will be removed in a future release. Deprecation warnings can be disabled by setting
deprecation_warnings=False in ansible.cfg.
 [ERROR]: unable to parse /usr/lib/python2.6/site-packages/ansible/modules/extras/cloud/misc/rhevm.py**

ERROR! module rhevm has a documentation error formatting or is missing documentation
```
解决：
```bash
[root@m01 ~]# grep deprecation_warnings /etc/ansible/ansible.cfg
\#deprecation_warnings = True
[root@m01 ~]# sed -i.bak 's@#deprecation_warnings = True@deprecation_warnings = False@g' /etc/ansible/ansible.cfg
[root@m01 ~]# grep deprecation_warnings /etc/ansible/ansible.cfg
deprecation_warnings = False
mv /usr/lib/python2.6/site-packages/ansible/modules/extras/cloud/misc/rhevm.py /tmp/
```
## yum模块
[https://docs.ansible.com/ansible/latest/modules/yum_module.html](https://docs.ansible.com/ansible/latest/modules/yum_module.html)
```
ansible oldboy -m yum -a "name=keepalived state=installed"
```
```yml
- name: install the latest version of Apache
  yum:
    name: httpd
    state: latest

- name: ensure a list of packages installed
  yum:
    name: "{{ packages }}"
  vars:
    packages:
    - httpd
    - httpd-tools

- name: remove the Apache package
  yum:
    name: httpd
    state: absent
```
```yml
- name: install the latest version of Apache from the testing repo
  yum:
    name: httpd
    enablerepo: testing
    state: present
```
## yum_repository模块
```yml
- name: Add repository
  yum_repository:
    name: epel
    description: EPEL YUM repo
    baseurl: https://download.fedoraproject.org/pub/epel/$releasever/$basearch/

- name: Add multiple repositories into the same file (1/2)
  yum_repository:
    name: epel
    description: EPEL YUM repo
    file: external_repos
    baseurl: https://download.fedoraproject.org/pub/epel/$releasever/$basearch/
    gpgcheck: no

- name: Add multiple repositories into the same file (2/2)
  yum_repository:
    name: rpmforge
    description: RPMforge YUM repo
    file: external_repos
    baseurl: http://apt.sw.be/redhat/el7/en/$basearch/rpmforge
    mirrorlist: http://mirrorlist.repoforge.org/el7/mirrors-rpmforge
    enabled: no

# Handler showing how to clean yum metadata cache
- name: yum-clean-metadata
  command: yum clean metadata
  args:
    warn: no

# Example removing a repository and cleaning up metadata cache
- name: Remove repository (and clean up left-over metadata)
  yum_repository:
    name: epel
    state: absent
  notify: yum-clean-metadata


- name: Remove repository from a specific repo file
  yum_repository:
    name: epel
    file: external_repos
    state: absent

- name: Add Base Yum Repository
  yum_repository:
    name: base
    description: Base Aliyun Repository
    baseurl: http://mirrors.aliyun.com/centos/$releasever/os/$basearch/
    gpgcheck: yes
    gpgkey: http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

- name: Add nginx Yum Repository
  yum_repository:
    name: nginx-stable
    description: nginx-stable Repository
    baseurl: http://nginx.org/packages/centos/$releasever/$basearch/
    gpgcheck: yes
    gpgkey: https://nginx.org/keys/nginx_signing.key
```
## service模块
[https://docs.ansible.com/ansible/latest/modules/service_module.html](https://docs.ansible.com/ansible/latest/modules/service_module.html)
```
ansible oldboy -m service -a "name=crond state=started enabled=yes"
```
```yml
- name: Start service httpd, if not started
  service:
    name: httpd
    state: started

- name: Stop service httpd, if started
  service:
    name: httpd
    state: stopped

- name: Restart service httpd, in all cases
  service:
    name: httpd
    state: restarted

- name: Reload service httpd, in all cases
  service:
    name: httpd
    state: reloaded

- name: Enable service httpd, and not touch the state
  service:
    name: httpd
    enabled: yes

- name: Start service foo, based on running process /usr/bin/foo
  service:
    name: foo
    pattern: /usr/bin/foo
    state: started

- name: Restart network service for interface eth0
  service:
    name: network
    state: restarted
    args: eth0
```
## systemd
[https://docs.ansible.com/ansible/latest/modules/systemd_module.html#systemd-module](https://docs.ansible.com/ansible/latest/modules/systemd_module.html#systemd-module)
```yml
- name: Make sure a service is running
  systemd:
    state: started
    name: httpd

- name: stop service cron on debian, if running
  systemd:
    name: cron
    state: stopped

- name: restart service cron on centos, in all cases, also issue daemon-reload to pick up config changes
  systemd:
    state: restarted
    daemon_reload: yes
    name: crond

- name: reload service httpd, in all cases
  systemd:
    name: httpd
    state: reloaded

- name: enable service httpd and ensure it is not masked
  systemd:
    name: httpd
    enabled: yes
    masked: no

- name: enable a timer for dnf-automatic
  systemd:
    name: dnf-automatic.timer
    state: started
    enabled: yes

- name: just force systemd to reread configs (2.4 and above)
  systemd:
    daemon_reload: yes

- name: just force systemd to re-execute itself (2.8 and above)
  systemd:
    daemon_reexec: yes
```
## user模块
[https://docs.ansible.com/ansible/latest/modules/user_module.html#user-module](https://docs.ansible.com/ansible/latest/modules/user_module.html#user-module)
```yml
- name: Add the user 'johnd' with a specific uid and a primary group of 'admin'
  user:
    name: johnd
    comment: John Doe
    uid: 1040
    group: admin
    shell: /bin/bash
    groups: admins,developers
    create_home: no
    append: yes
```
```yml
- name: Remove the user 'johnd'
  user:
    name: johnd
    state: absent
    remove: yes
```
### 生成密码
```
ansible all -i localhost, -m debug -a "msg={{ 'mypassword' | password_hash('sha512', 'mysecretsalt') }}"
ansible all -i localhost, -m debug -a "msg={{ '123456' | password_hash('sha512', 'oldboy123') }}"
```
### 创建带密码的用户
```
ansible backup -m user -a 'name=Alex02 password="$6$oldgirl$kAUTXVC2z1agr1HlmpFe9abFhWKwJ1fNyg64F95U3rVumwQfqOuhV3YkyZU9.H79TChzIKn5epl5M18B199qV1"'
```
## group模块
```yml
- name: Ensure group "somegroup" exists
  group:
    name: somegroup
    gid: 1000
    state: present
```
## archive模块
[https://docs.ansible.com/ansible/latest/modules/archive_module.html#archive-module](https://docs.ansible.com/ansible/latest/modules/archive_module.html#archive-module)
```yml
- name: Create a bz2 archive of a globbed path, while excluding a glob of dirnames
  archive:
    path:
    - /path/to/foo/*
    dest: /path/file.tar.bz2
    exclude_path:
    - /path/to/foo/ba*
    format: bz2
```
```yml
- name: Create a zip archive of /path/to/foo
  archive:
    path: /path/to/foo
    format: zip
```
## unarchive模块
[https://docs.ansible.com/ansible/latest/modules/unarchive_module.html#unarchive-module](https://docs.ansible.com/ansible/latest/modules/unarchive_module.html#unarchive-module)
 默认解压方式  /usr/bin/gtar  /usr/bin/unzip
```yml
- name: Unarchive a file that is already on the remote machine
  unarchive:
    src: /tmp/foo.zip
    dest: /usr/local/bin
    remote_src: yes

- name: Unarchive a file that needs to be downloaded (added in 2.0)
  unarchive:
    src: https://example.com/example.zip
    dest: /usr/local/bin
    remote_src: yes
```
## selinux模块
```yml
- selinux:
    state: disabled
```
## synchronize模块
```yml
# Synchronization of src on the control machine to dest on the remote hosts
- synchronize:
    src: some/relative/path
    dest: /some/absolute/path

# Synchronization using rsync protocol (push)
- synchronize:
    src: some/relative/path/
    dest: rsync://somehost.com/path/

# Synchronization using rsync protocol (pull)
- synchronize:
    mode: pull
    src: rsync://somehost.com/path/
    dest: /some/absolute/path/
```
## authorized_key模块
### 1.修改配置文件
```
[root@m01 ~]# grep host_key_checking /etc/ansible/ansible.cfg -n
59:#host_key_checking = False
[root@m01 ~]# sed -i.ori '59s@#host_key_checking = False@host_key_checking = False@g' /etc/ansible/ansible.cfg
[root@m01 ~]# grep host_key_checking /etc/ansible/ansible.cfg -n
59:host_key_checking = False
```
### 2.分发公钥
```
ssh-keygen -t dsa -P "" -f /root/.ssh/id_dsa &> /dev/null
ansible oldboy -m authorized_key -a "user=oldboy key='{{ lookup('file', '/home/oldboy/.ssh/id_rsa.pub') }}'" -k
```
```yml
- name: Set authorized key took from file
  authorized_key:
    user: charlie
    state: present
    key: "{{ lookup('file', '/root/.ssh/id_rsa.pub') }}"
```
```yml
---
- hosts: nginx
  tasks:
  - name: Set authorized key took from file
    authorized_key:
      user: root
      state: present
      key: "{{ lookup('file', '/root/.ssh/id_rsa.pub') }}"
```
cat authkey.yml
```yml
---
- hosts:
  - nfs01
  - backup
  tasks: 
    - name: Set authorized key took from file
      authorized_key:
        user: root
        state: present
        key: "{{ lookup('file', '/root/.ssh/id_rsa.pub') }}"
...
```
## debug
```yml
- debug:
    msg: "System {{ inventory_hostname }} has uuid {{ ansible_product_uuid }}"
```
## mysql_db模块
添加或删除数据库
ansible及远程主机安装
```bash
yum install MySQL-python
yum install python2-PyMySQL
```
cat ~/.my.cnf
```
[mysql]
host=172.16.1.7
user=root
password=123456
```
```yml
- name: Create a new database with name 'bobdata'
  mysql_db:
    name: bobdata
    state: present

# Copy database dump file to remote host and restore it to database 'my_db'
- name: Copy database dump file
  copy:
    src: dump.sql.bz2
    dest: /tmp
- name: Restore database
  mysql_db:
    name: my_db
    state: import
    target: /tmp/dump.sql.bz2

- name: Dump all databases to hostname.sql
  mysql_db:
    state: dump
    name: all
    target: /tmp/{{ inventory_hostname }}.sql

- name: Import file.sql similar to mysql -u <username> -p <password> < hostname.sql
  mysql_db:
    state: import
    name: all
    target: /tmp/{{ inventory_hostname }}.sql
```
state
>   present
  absent
  dump
  import
```yml
---
- hosts: 172.16.1.7
  tasks:
  - name: Create a new database with name 'bobdata'
    mysql_db:
      name: blog
      state: present
```
## mysql_user模块
```yml
# Removes anonymous user account for localhost
- mysql_user:
    name: ''
    host: localhost
    state: absent

# Removes all anonymous user accounts
- mysql_user:
    name: ''
    host_all: yes
    state: absent

# Create database user with name 'bob' and password '12345' with all database privileges
- mysql_user:
    name: bob
    password: 12345
    priv: '*.*:ALL'
    state: present

# Create database user with name 'bob' and previously hashed mysql native password '*EE0D72C1085C46C5278932678FBE2C6A782821B4' with all database privileges
- mysql_user:
    name: bob
    password: '*EE0D72C1085C46C5278932678FBE2C6A782821B4'
    encrypted: yes
    priv: '*.*:ALL'
    state: present

# Creates database user 'bob' and password '12345' with all database privileges and 'WITH GRANT OPTION'
- mysql_user:
    name: bob
    password: 12345
    priv: '*.*:ALL,GRANT'
    state: present

# Modify user Bob to require SSL connections. Note that REQUIRESSL is a special privilege that should only apply to *.* by itself.
- mysql_user:
    name: bob
    append_privs: true
    priv: '*.*:REQUIRESSL'
    state: present

# Ensure no user named 'sally'@'localhost' exists, also passing in the auth credentials.
- mysql_user:
    login_user: root
    login_password: 123456
    name: sally
    state: absent

# Ensure no user named 'sally' exists at all
- mysql_user:
    name: sally
    host_all: yes
    state: absent

# Specify grants composed of more than one word
- mysql_user:
    name: replication
    password: 12345
    priv: "*.*:REPLICATION CLIENT"
    state: present

# Revoke all privileges for user 'bob' and password '12345'
- mysql_user:
    name: bob
    password: 12345
    priv: "*.*:USAGE"
    state: present

# Example privileges string format
# mydb.*:INSERT,UPDATE/anotherdb.*:SELECT/yetanotherdb.*:ALL

# Example using login_unix_socket to connect to server
- mysql_user:
    name: root
    password: abc123
    login_unix_socket: /var/run/mysqld/mysqld.sock

# Example of skipping binary logging while adding user 'bob'
- mysql_user:
    name: bob
    password: 12345
    priv: "*.*:USAGE"
    state: present
    sql_log_bin: no

# Example .my.cnf file for setting the root password
# [client]
# user=root
# password=n<_665{vS43y
```
```yml
---
- hosts: 172.16.1.7
  tasks:
  - name: create mysql user
    mysql_user:
      login_host: 172.16.1.7
      login_user: root
      login_password: 123456
      name: www
      host: '%'
      password: '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9'
      encrypted: yes
      priv: '*.*:ALL'
      state: present
```
cat ~/.my.cnf
```
[mysql]
host=172.16.1.7
user=root
password=123456
```
```yml
---
- hosts: 172.16.1.7
  tasks:
  - name: create mysql user
    mysql_user:
      name: wuxingge
      host: '%'
      password: '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9'
      encrypted: yes
      priv: '*.*:ALL'
      state: present
```
## mysql_replication模块
```yml
# Stop mysql slave thread
- mysql_replication:
    mode: stopslave

# Get master binlog file name and binlog position
- mysql_replication:
    mode: getmaster

# Change master to master server 192.0.2.1 and use binary log 'mysql-bin.000009' with position 4578
- mysql_replication:
    mode: changemaster
    master_host: 192.0.2.1
    master_log_file: mysql-bin.000009
    master_log_pos: 4578

# Check slave status using port 3308
- mysql_replication:
    mode: getslave
    login_host: ansible.example.com
    login_port: 3308
```
mode
> getslave 
getmaster
changemaster
stopslave
startslave
resetslave
resetslaveall
# ansible  playbook(剧本)
[https://docs.ansible.com/ansible/latest/user_guide/playbooks.html](https://docs.ansible.com/ansible/latest/user_guide/playbooks.html)
关键字
[https://docs.ansible.com/ansible/latest/reference_appendices/playbooks_keywords.html](https://docs.ansible.com/ansible/latest/reference_appendices/playbooks_keywords.html)
## 语法格式
```
ansible剧本格式    yaml语法格式
rsync配置文件格式   ini语法格式
sersync配置文件格式   xml语法格式
json语法格式
```
## yaml语法
### 1.缩进
```yml
aaaa
  bbbb
    cccc
```
### 2.列表
```yml
项目:
  - 项目名1
  - 项目名2
```
### 3.冒号空格
```yml
key: value
```
## 剧本存放目录
```
[root@m01 ~]# cd /etc/ansible/
[root@m01 ansible]# ll
总用量 32
-rw-r--r-- 1 root root 18066 6月   2 05:49 ansible.cfg
-rw-r--r-- 1 root root    33 6月  30 10:14 hosts
drwxr-xr-x 2 root root  4096 6月   2 05:49 roles
```
## 一个标准的剧本写法

cat /etc/ansible/test.yml
```yml
---
- hosts: 10.0.0.31
  gather_facts: no
  tasks:
  - name: add user
    user:
      name: rsync
      shell: /sbin/nologin
      createhome: no
...
```
```yml
---
- hosts:
    - backup
    - nfs
  gather_facts: no
  tasks:
  - name: touch file
    file:
      path: /tmp/test01.txt
      state: touch
...
```
```yml
atlanta:
  hosts:
    host1:
    host2:
  vars:
    ntp_server: ntp.atlanta.example.com
    proxy: proxy.atlanta.example.c
```
```yml
---
- hosts: webservers
  vars:
    http_port: 80
    max_clients: 200
  remote_user: root
  tasks:
  - name: ensure apache is at the latest version
    yum:
      name: httpd
      state: latest
  - name: write the apache config file
    template:
      src: /srv/httpd.j2
      dest: /etc/httpd.conf
    notify:
    - restart apache
  - name: ensure apache is running
    service:
      name: httpd
      state: started
  handlers:
    - name: restart apache
      service:
        name: httpd
        state: restarted
```
```yml
---
- hosts: webservers
  remote_user: root

  tasks:
  - name: ensure apache is at the latest version
    yum:
      name: httpd
      state: latest
  - name: write the apache config file
    template:
      src: /srv/httpd.j2
      dest: /etc/httpd.conf

- hosts: databases
  remote_user: root

  tasks:
  - name: ensure postgresql is at the latest version
    yum:
      name: postgresql
      state: latest
  - name: ensure that postgresql is started
    service:
      name: postgresql
      state: started
```
# 检查剧本语法
```
ansible-playbook --syntax-check /etc/ansible/ansible_playbook/test01.yaml
```
# 模拟执行
```
ansible-playbook /etc/ansible/show.yml -C
```
# handler
web.yaml
```yml
- hosts: web
  tasks:
    - name: Install Httpd Server
      yum:
        name: httpd,httpd-tools
        state: installed
    - name: Config Httpd Server
      copy:
        src: ./httpd.conf
        dest: /etc/httpd/conf/httpd.conf
      notify: restart httpd
    - name: Start Httpd Server
      service:
        name: httpd
        state: started
        enabled: yes
  handlers:
    - name: restart httpd
      service:
        name: httpd
        state: restarted
```
# 部署rsync
```yml
---
- hosts: 10.0.0.31
  tasks:
  - name: add user
    user: name=rsync shell=/sbin/nologin createhome=no
  - name: install rsync
    yum: name=rsync state=latest
  - name: copy rsyncd.conf
    copy: src=/etc/rsyncd.conf dest=/etc/rsyncd.conf owner=root group=root mode=0644
  - name: rsync password
    copy: content='rsync_backup:123' dest=/etc/rsync.password owner=root group=root mode=0600
  - name: create /backup dir
    file: path=/backup state=directory owner=rsync group=rsync mode=0755
  - name: start rsync
    shell: /usr/bin/rsync --daemon
...
```
```yml
---
- hosts: 10.0.0.31
  tasks:
  - name: add user
    user:
      name: rsync
      shell: /sbin/nologin
      createhome: no
  - name: install rsync
    yum:
      name: rsync
      state: latest
  - name: copy rsyncd.conf
    copy:
      src: /etc/rsyncd.conf
      dest: /etc/rsyncd.conf
      owner: root
      group: root
      mode: 0644
  - name: rsync password
    copy:
      content: rsync_backup:123
      dest: /etc/rsync.password
      owner: root
      group: root
      mode: 0600
  - name: create /backup dir
    file:
      path: /backup
      state: directory
      owner: rsync
      group: rsync
      mode: 0755
  - name: start rsync
    shell: rsync --daemon
    args:
      chdir: /usr/bin/
...
---
- hosts: 10.0.0.31
  tasks:
  - name: add user
    user:
      name: rsync
      shell: /sbin/nologin
      createhome: no
  - name: install rsync
    yum:
      name: rsync
      state: latest
  - name: copy rsyncd.conf
    copy:
      src: /etc/rsyncd.conf
      dest: /etc/rsyncd.conf
      owner: root
      group: root
      mode: 0644
  - name: rsync password
    copy:
      content: rsync_backup:123
      dest: /etc/rsync.password
      owner: root
      group: root
      mode: 0600
  - name: create /backup dir
    file:
      path: /backup
      state: directory
      owner: rsync
      group: rsync
      mode: 0755
  - name: start rsync
    shell: /usr/bin/rsync --daemon
...
```
# 变量
[https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html)
## 在剧本文件中进行定义 vars
```yml
- hosts: webservers
  vars:
    http_port: 80
```
## 利用执行参数赋值变量 --extra-vars
```
ansible-playbook -e dir_info=/oldboy/ -e file_info=oldboy01.txt  test_var.yaml 
```
## 单独在一个文件中定义 /etc/ansible/hosts
```
[oldboy]
172.16.1.31
172.16.1.41
172.16.1.7
[oldboy:vars]
dir_info=/oldboy/
file_info=oldboy01.txt
```
## 采用注册方式定义变量 register
```
  - name: check rpc port
    shell: ss -lntup |grep 111
    register: check_port_info
  - name: output check info
    debug:
      msg: '{{ check_port_info.stdout_lines }}'
```
# 变量优先级
> 命令行  >  剧本  >  主机清单
```yml
foo:
  field1: one
  field2: two
```
```
foo['field1']
foo.field1
```
## 引用变量
```yml
{{ foo }}
```
cat var.yml
```yml
--- 
- hosts: oldboy
  remote_user: root
  vars:
    file_info: oldboy01.txt
    dir_info: /oldboy/
  tasks:
  - name: create dir
    file:
      path: '{{ dir_info }}'
      state: directory
  - name: copy file
    copy:
      src: '{{ dir_info }}{{ file_info }}'
      dest: '{{ dir_info }}'
```
# 变量优先级(由低到高)
* role defaults
* dynamic inventory variables
* inventory variables
* inventory group_vars
* inventory host_vars
* playbook group_vars
* playbook host_vars
* host facts
* registered variables
* set_facts
* play variables
* play vars_prompt
* play vars_files
* role variables and include variables
* block variables
* task variables
* extra variables
---
# tags
[https://docs.ansible.com/ansible/latest/user_guide/playbooks_tags.html](https://docs.ansible.com/ansible/latest/user_guide/playbooks_tags.html)
指定位置添加标签,可以指定标签执行剧本
```yml
- hosts: 172.16.1.7
  gather_facts: no
  tasks:
  - name: mount
    mount:
      src: "172.16.1.31:/data"
      path: "/mnt"
      fstype: nfs
      state: mounted
    tags: mount-nfs
```
## 只执行标签
```
ansible-playbook  -t mount-nfs nfs.yml 
```
## 不执行标签
```
ansible-playbook  --skip-tags "mount-nfs"   nfs.yml
```
# 忽略错误
```
ignore_errors: yes
```
---
# notify和handler
## notify
```yml
  tasks:
  - name: copy template file to remote host
    template:
      src: /etc/ansible/nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    notify:
      - restart nginx
      - test web page
    copy:
      src: nginx/index.html.j2
      dest: /usr/share/nginx/html/index.html
    notify:
      - restart nginx
```
## handler
```yml
  handlers:
  - name: restart nginx
    service:
      name: nginx
      state: restarted
  - name: test web page
    shell: curl -I http://192.168.100.10/index.html | grep 200 || /bin/false
```
nfs.yml
```yml
---
- hosts: all
  gather_facts: no
  tasks:
  - name: install nfs
    yum:
      name:
        - nfs-utils
        - rpcbind
      state: installed

- hosts: 172.16.1.31
  gather_facts: no
  tasks:
  - name: push exports
    copy:
      src: export.temp
      dest: /etc/exports
    notify: restart nfs
  - name: create data dir
    file:
      path: /data
      owner: nfsnobody
      group: nfsnobody
      state: directory
  - name: start nfs
    service:
      name: rpcbind
      name: nfs
      state: started
      enabled: yes
    tags: test-nfs-start
  - name: check rpc port
    shell: ss -lntup |grep 111
    register: check_port_info
  - name: output check info
    debug:
      msg: '{{ check_port_info.stdout_lines }}'
    tags: test_port
  handlers:
    - name: restart nfs
      service:
        name: nfs
        state: restarted
- hosts: 172.16.1.7
  gather_facts: no
  tasks:
  - name: mount
    mount:
      src: "172.16.1.31:/data"
      path: "/mnt"
      fstype: nfs
      state: mounted
    tags: mount-nfs
...
```
cat rsync.yml 
```yml
---
- hosts: all
  remote_user: root
  tasks:
  - name: install rsync
    yum:
      name: rsync
      state: installed
    tags: install rsync

- hosts: 172.16.1.41
  tasks:
  - name: copy rsyncd.conf
    copy:
      src: '{{ item.src }}'
      dest: '{{ item.dest }}'
      mode: '{{ item.mode }}'
    with_items:
      - { src: 'rsyncd.conf.temp',dest: /etc/rsyncd.conf,mode: '0644' }
      - { src: 'rsync.password.temp',dest: /etc/rsync.password,mode: '0600' }
    notify: restart rsync
  - name: create user
    user:
      name: rsync
      shell: /sbin/nologin
      create_home: no
  - name: create backup directory
    file:
      path: '{{ item }}'
      owner: rsync
      group: rsync
      state: directory
    with_items:
      - /backup
      - /wuxing
  - name: start rsync
    service:
      name: rsyncd
      state: started
  handlers:
    - name: restart rsync
      service:
        name: rsyncd
        state: restarted

- hosts:
    - 172.16.1.31
    - 172.16.1.7
  tasks:
  - name: create pass file
    copy:
      content: "123"
      dest: /etc/rsync.password
      mode: 0600
```
---
# 条件
[https://docs.ansible.com/ansible/latest/user_guide/playbooks_conditionals.html](https://docs.ansible.com/ansible/latest/user_guide/playbooks_conditionals.html)
## when
```yml
tasks:
  - name: "shut down CentOS 6 systems"
    command: /sbin/shutdown -t now
    when:
      - ansible_facts['distribution'] == "CentOS"
      - ansible_facts['distribution_major_version'] == "6"
```
```yml
tasks:
  - command: /bin/false
    register: result
    ignore_errors: True

  - command: /bin/something
    when: result is failed

  # In older versions of ansible use ``success``, now both are valid but succeeded uses the correct tense.
  - command: /bin/something_else
    when: result is succeeded

  - command: /bin/still/something_else
    when: result is skipped
```
# 布尔
```yml
vars:
  epic: true
```
```yml
tasks:
  - shell: echo "This certainly is epic!"
    when: epic

tasks:
  - shell: echo "This certainly isn't epic!"
    when: not epic
```
```yml
tasks:
  - shell: echo "I've got '{{ foo }}' and am not afraid to use it!"
    when: foo is defined

    - fail: msg="Bailing out. this play requires 'bar'"
      when: bar is undefined
```
cat judge.yml 
```yml
---
- hosts: oldboy
  remote_user: root
  tasks:
  - name: push file01 info
    copy:
      src: /oldboy/oldboy01
      dest: /tmp/
    when: ansible_hostname == 'web01'
```
cat judge.yml 
```yml
---
- hosts: oldboy
  remote_user: root
  tasks:
  - name: push file01 info
    copy:
      src: /oldboy/oldboy01
      dest: /tmp/
    when: ansible_facts.eth1.ipv4.address == '172.16.1.7'
```
```yml
---
- hosts: oldboy
  remote_user: root
  tasks:
  - name: push file01 info
    copy:
      src: /oldboy/oldboy01
      dest: /tmp/
    when: ansible_facts["eth1"]["ipv4"]["address"] == '172.16.1.7'
```
# and
```yml
when: (ansible_hostname == "web01") and (ansible_distribution == "CentOS")
```
# or
# not
# 循环和条件
```yml
tasks:
    - command: echo {{ item }}
      loop: [ 0, 2, 4, 6, 8, 10 ]
      when: item > 5
```
# 循环
[https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html)
# loop
```yml
- name: add several users
  user:
    name: "{{ item }}"
    state: present
    groups: "wheel"
  loop:
     - testuser1
     - testuser2
```
```yml
  loop: "{{ somelist }}"
```
```yml
- name: optimal yum
  yum:
    name: "{{list_of_packages}}"
    state: present

- name: non optimal yum, not only slower but might cause issues with interdependencies
  yum:
    name: "{{item}}"
    state: present
  loop: "{{list_of_packages}}"
```
```yml
- name: add several users
  user:
    name: "{{ item.name }}"
    state: present
    groups: "{{ item.groups }}"
  loop:
    - { name: 'testuser1', groups: 'wheel' }
    - { name: 'testuser2', groups: 'root' }
```
```yml
- name: create a tag dictionary of non-empty tags
  set_fact:
    tags_dict: "{{ (tags_dict|default({}))|combine({item.key: item.value}) }}"
  loop: "{{ tags|dict2items }}"
  vars:
    tags:
      Environment: dev
      Application: payment
      Another: "{{ doesnotexist|default() }}"
  when: item.value != ""
```
```yml
- shell: "echo {{ item }}"
  loop:
    - "one"
    - "two"
  register: echo
```
# [Loop Control](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html#id12)
```yml
# main.yml
- include_tasks: inner.yml
  loop:
    - 1
    - 2
    - 3
  loop_control:
    loop_var: outer_item

# inner.yml
- debug:
    msg: "outer item={{ outer_item }} inner item={{ item }}"
  loop:
    - a
    - b
    - c
```
# 直到循环 until
```yml
- shell: /usr/bin/foo
  register: result
  until: result.stdout.find("all systems go") != -1
  retries: 5
  delay: 10
```
# [with_list](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html#id14)
```yml
- name: with_list
  debug:
    msg: "{{ item }}"
  with_list:
    - one
    - two

- name: with_list -> loop
  debug:
    msg: "{{ item }}"
  loop:
    - one
    - two
```
# [with_items](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html#id15)
```yml
- name: with_items
  debug:
    msg: "{{ item }}"
  with_items: "{{ items }}"

- name: with_items -> loop
  debug:
    msg: "{{ item }}"
  loop: "{{ items|flatten(levels=1) }}"

  - name: create backup directory
    file:
      path: '{{ item }}'
      owner: rsync
      group: rsync
      state: directory
    with_items:
      - /backup
      - /wuxing
```
cat with_items.yml 
```yml
---
- hosts: oldboy
  tasks:
  - name: create user
    user:
      name: '{{ item.name }}'
      uid: '{{ item.uid }}'
      shell: '{{ item.shell }}'
    with_items:
      - { name: "alex01", uid: "3001", shell: "/sbin/nologin" }
      - { name: "alex02", uid: "3002", shell: "/sbin/nologin" }
      - { name: "alex03", uid: "3003", shell: "/sbin/nologin" }
```
# [with_indexed_items](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html#id16)
```yml
- name: with_indexed_items
  debug:
    msg: "{{ item.0 }} - {{ item.1 }}"
  with_indexed_items: "{{ items }}"

- name: with_indexed_items -> loop
  debug:
    msg: "{{ index }} - {{ item }}"
  loop: "{{ items|flatten(levels=1) }}"
  loop_control:
    index_var: index
```
# [with_flattened](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html#id17)
```yml
- name: with_flattened
  debug:
    msg: "{{ item }}"
  with_flattened: "{{ items }}"

- name: with_flattened -> loop
  debug:
    msg: "{{ item }}"
  loop: "{{ items|flatten }}"
```
# [with_together](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html#id18)
```yml
- name: with_together
  debug:
    msg: "{{ item.0 }} - {{ item.1 }}"
  with_together:
    - "{{ list_one }}"
    - "{{ list_two }}"

- name: with_together -> loop
  debug:
    msg: "{{ item.0 }} - {{ item.1 }}"
  loop: "{{ list_one|zip(list_two)|list }}"
```
# [with_dict](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html#id19)
```yml
- name: with_dict
  debug:
    msg: "{{ item.key }} - {{ item.value }}"
  with_dict: "{{ dictionary }}"

- name: with_dict -> loop (option 1)
  debug:
    msg: "{{ item.key }} - {{ item.value }}"
  loop: "{{ dictionary|dict2items }}"

- name: with_dict -> loop (option 2)
  debug:
    msg: "{{ item.0 }} - {{ item.1 }}"
  loop: "{{ dictionary|dictsort }}"
```
# [with_sequence](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html#id20)
```yml
- name: with_sequence
  debug:
    msg: "{{ item }}"
  with_sequence: start=0 end=4 stride=2 format=testuser%02x

- name: with_sequence -> loop
  debug:
    msg: "{{ 'testuser%02x' | format(item) }}"
  # range is exclusive of the end point
  loop: "{{ range(0, 4 + 1, 2)|list }}"
```
# [with_subelements](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html#id21)
```yml
- name: with_subelements
  debug:
    msg: "{{ item.0.name }} - {{ item.1 }}"
  with_subelements:
    - "{{ users }}"
    - mysql.hosts

- name: with_subelements -> loop
  debug:
    msg: "{{ item.0.name }} - {{ item.1 }}"
  loop: "{{ users|subelements('mysql.hosts') }}"
```
# [with_nested/with_cartesian](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html#id22)
```yml
- name: with_nested
  debug:
    msg: "{{ item.0 }} - {{ item.1 }}"
  with_nested:
    - "{{ list_one }}"
    - "{{ list_two }}"

- name: with_nested -> loop
  debug:
    msg: "{{ item.0 }} - {{ item.1 }}"
  loop: "{{ list_one|product(list_two)|list }}"
```
# [with_random_choice](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html#id23)
```yml
- name: with_random_choice
  debug:
    msg: "{{ item }}"
  with_random_choice: "{{ my_list }}"

- name: with_random_choice -> loop (No loop is needed here)
  debug:
    msg: "{{ my_list|random }}"
  tags: random
```
# 剧本整合
## 指令参数1   roles
```yml
- hosts:
  tasks:
    - include_tasks: nfs.yml
    - include_tasks: rsync.yml
```
## 指令参数2
```yml
- import_playbook: nfs.yml
- import_playbook: rsync.yml
```
roles/x/tasks/main.yml
```yml
---
- include: create_dir.yml
- include: static_git_pull.yml
- include: git_checkout.yml
```
# roles
[https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html)
![image.png](https://upload-images.jianshu.io/upload_images/16826267-7d7ef5dc5e83e388.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- group_vars目录下的文件定义Roles中调用的变量
- 文件名为all的文件定义的变量针对所有Roles生效
## 角色目录结构
```
site.yml
webservers.yml
fooservers.yml
roles/
   common/
     tasks/
     handlers/
     files/
     templates/
     vars/
     defaults/
     meta/
   webservers/
     tasks/
     defaults/
     meta/
```
角色期望文件位于某些目录名称中。**角色必须至少包含其中一个目录**，但是排除任何未使用的目录是完全正确的。在使用时，**每个目录必须包含一个main.yml文件**，其中包含相关内容
**tasks    -  包含角色要执行的主要任务列表
handlers    - 包含处理程序，可以由此角色使用，甚至可以在此角色之外的任何位置使用
defaults   - 角色的默认变量（有关更多信息，请参阅使用变量）
vars       - 角色的其他变量（有关更多信息，请参阅使用变量）
files      - 包含可以通过此角色部署的文件
templates      - 包含可以通过此角色部署的模板
meta         - 为此角色定义一些元数据
其他YAML文件可能包含在某些目录中。例如，通常的做法是从tasks/main.yml文件中包含特定于平台的任务**
```yml
#roles/example/tasks/main.yml
- name: added in 2.4, previously you used 'include'
  import_tasks: redhat.yml
  when: ansible_facts['os_family']|lower == 'redhat'
- import_tasks: debian.yml
  when: ansible_facts['os_family']|lower == 'debian'
```
```yml
# roles/example/tasks/redhat.yml
- yum:
    name: "httpd"
    state: present
```
```yml
# roles/example/tasks/debian.yml
- apt:
    name: "apache2"
    state: present
```
```yml
---
- hosts: webservers
  roles:
    - common
    - webservers
```
```yml
---
- hosts: webservers
  roles:
    - role: '/path/to/my/roles/common'
```
```yml
---
- hosts: 172.16.1.31
  roles:
    - { role: nfs }
- hosts: 172.16.1.41
  roles:
    - { role: rsync }
```
```yml
---
- hosts: webservers
  roles:
    - common
    - role: foo_app_instance
      vars:
        dir: '/opt/a'
        app_port: 5000
    - role: foo_app_instance
      vars:
        dir: '/opt/b'
        app_port: 5001
```
```yml
---
- hosts: webservers
  tasks:
  - include_role:
       name: foo_app_instance
    vars:
      dir: '/opt/a'
      app_port: 5000
  ...
```
```yml
---
- hosts: webservers
  tasks:
  - include_role:
      name: some_role
    when: "ansible_facts['os_family'] == 'RedHat'"
```
```yml
---
- hosts: webservers
  roles:
    - role: bar
      tags: ["foo"]
    # using YAML shorthand, this is equivalent to the above
    - { role: foo, tags: ["bar", "baz"] }
```
```yml
---
- hosts: webservers
  tasks:
  - import_role:
      name: foo
    tags:
    - bar
    - baz
```
```yml
- hosts: webservers
  tasks:
  - include_role:
      name: bar
    tags:
     - foo
```
```yml
---
- hosts: webservers
  roles:
  - role: foo
    vars:
         message: "first"
  - { role: foo, vars: { message: "second" } }
```
# 角色中使用模块与插件
```yml
roles/
   my_custom_modules/
       library/
          module1
          module2
```
```yml
- hosts: webservers
  roles:
    - my_custom_modules
    - some_other_role_using_my_custom_modules
    - yet_another_role_using_my_custom_modules
```
---
## roles剧本实例
### 创建目录
```
[root@m01 roles]# mkdir {nfs,rsync}/{tasks,files,templates,meta,handlers,vars,defaults}
[root@m01 roles]# tree
.
├── nfs
│   ├── files
│   ├── handlers
│   ├── meta
│   ├── tasks
│   ├── templates
│   └── vars
└── rsync
    ├── files
    ├── handlers
    ├── meta
    ├── tasks
    ├── templates
    └── vars
```
```
[root@m01 roles]# pwd
/etc/ansible/roles
[root@m01 roles]# ll
total 8
drwxr-xr-x 8 root root 89 Mar  8 11:32 nfs
drwxr-xr-x 8 root root 89 Mar  8 11:32 rsync
-rw-r--r-- 1 root root 35 Mar  8 12:13 site.yml
```
```
[root@m01 roles]# tree nfs
nfs
├── files
│   └── exports
├── handlers
│   └── main.yml
├── meta
├── tasks
│   └── main.yml
├── templates
└── vars
    └── main.yml
```
## vars(变量)
[root@m01 roles]# cat nfs/vars/main.yml 
```yml
data_dir: /data
```
vars/main.yml中的变量是role变量，优先级比较高，放置一些不想被覆盖的变量，所以变量在命名的时候一般都加入role的名字作为前缀，防止不小心被Playbook中定义的变量覆盖
## defaults(默认变量)
defaults/main.yml中的变量是默认变量。优先级在所有的变量中是最低的，用于放置一些需要被覆盖的变量
## tasks(任务)
[root@m01 roles]# cat nfs/tasks/main.yml 
```yml
- name: install nfs
  yum:
    name:
      - nfs-utils
      - rpcbind
    state: installed

- name: push exports
  copy:
    src: exports
    dest: /etc/exports
  when: ansible_facts.eth1.ipv4.address == '172.16.1.31'
  notify: restart nfs

- name: create data dir
  file:
    path: '{{ data_dir }}'
    owner: nfsnobody
    group: nfsnobody
    state: directory
  when: ansible_facts.eth1.ipv4.address == '172.16.1.31'

- name: start nfs
  service:
    name: rpcbind
    name: nfs
    state: started
    enabled: yes
  when: ansible_facts.eth1.ipv4.address == '172.16.1.31'
  tags: test-nfs-start

- name: check rpc port
  shell: ss -lntup |grep 111
  when: ansible_facts.eth1.ipv4.address == '172.16.1.31'
  register: check_port_info

- name: output check info
  debug:
    msg: '{{ check_port_info.stdout_lines }}'
  when: ansible_facts.eth1.ipv4.address == '172.16.1.31'
  tags: test_port

- name: mount
  mount:
    src: "172.16.1.31:/data"
    path: "/mnt"
    fstype: nfs
    state: mounted
  when: ansible_facts.eth1.ipv4.address == '172.16.1.41' or ansible_facts.eth1.ipv4.address == '172.16.1.7'
  tags: mount-nfs
```
## handlers(触发任务)
[root@m01 roles]# cat nfs/handlers/main.yml 
```yml
- name: restart nfs
  service:
    name: nfs
    state: restarted
```
## files(文件)
## templates(模板文件)
cat roles/nginx/templates/default.conf.j2
```
server {
    listen       {{ nginx_port }};
    server_name  localhost;
    location / {
        root   {{ html_root }};
        index  index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```
cat roles/nginx/templates/nginx.conf.j2
```
user  nginx;
worker_processes  {{ ansible_processor_vcpus }};

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```
## meta(角色依赖)
roles/myapp/meta/main.yml
```yml
---
dependencies:
  - role: common
    vars:
      some_parameter: 3
  - role: apache
    vars:
      apache_port: 80
  - role: postgres
    vars:
      dbname: blarg
      other_parameter: 12
```
```yml
---
dependencies:
- { role: common, name: "NameInDb"}
```
roles/nginx/meta/main.yml
```yml
---
dependencies:
  - role: common
```
roles/common/tasks/main.yml
```yml
- name: Add nginx Yum Repository
  yum_repository:
    name: nginx-stable
    description: nginx-stable Repository
    baseurl: http://nginx.org/packages/centos/$releasever/$basearch/
    gpgcheck: yes
    gpgkey: https://nginx.org/keys/nginx_signing.key
```
# 角色调用
cat site.yml 
```yml
- hosts: oldboy
  roles:
    - nfs
```
```yml
---
- hosts: webservers
  roles:
  - role: foo
    vars:
         message: "first"
  - { role: foo, vars: { message: "second" } }
```
---
# Templating (Jinja2)
[https://docs.ansible.com/ansible/latest/user_guide/playbooks_templating.html](https://docs.ansible.com/ansible/latest/user_guide/playbooks_templating.html)
[root@m01 roles]# cat rsync/templates/rsyncd.conf.j2 
```ini
uid = rsync
gid = rsync
port = {{ port }}
fake super = yes
use chroot = no
max connections = 200
timeout = 600
pid file = /var/run/rsyncd.pid
lock file = /var/run/rsync.lock
log file = /var/log/rsyncd.log
ignore errors
read only = false
list = true
hosts allow = 172.16.1.0/24
auth users = rsync_backup
secrets file = /etc/rsync.password
[backup]
path = /backup
[wuxing]
path = /wuxing
```
[root@m01 roles]# cat rsync/vars/main.yml 
```yml
port: 874
```
[root@m01 roles]# cat rsync/tasks/main.yml 
```yml
- name: install rsync
  yum:
    name: rsync
    state: installed

- name: push conf
  template:
    src: rsyncd.conf.j2
    dest: /etc/rsyncd.conf
```
[root@m01 roles]# cat site.yml
```yml
- hosts: oldboy
  roles:
    - nfs
- hosts: 172.16.1.41
  roles:
    - { role: rsync }
```
# 完整的ansible配置目录
```
[root@m01 ansible]# pwd
/etc/ansible
[root@m01 ansible]# ll
total 24
-rw-r--r-- 1 root root 20318 Mar  6 12:32 ansible.cfg
-rw-r--r-- 1 root root  1015 Jun 17 16:01 hosts
drwxr-xr-x 2 root root    32 Jun 17 18:33 inventory
drwxr-xr-x 3 root root    41 Jun 17 18:59 playbooks
drwxr-xr-x 6 root root    73 Jun 18 09:29 roles
```
```
[root@m01 ansible]# tree -L 2 /etc/ansible/
/etc/ansible/
├── ansible.cfg
├── hosts
├── inventory
│   ├── part1
│   └── part2
├── playbooks
│   ├── group_vars
│   └── nginx.yml
└── roles
    ├── common
    ├── nfs
    ├── nginx
    ├── rsync
    └── site.yml
```
# jinja
[http://jinja.pocoo.org/](http://jinja.pocoo.org/)
## 步骤1：编排目录如下
```
nginxconf.yml
roles/nginxconf/
├── tasks
│   ├── file.yml
│   └── main.yml
├── templates
│   └── nginx.conf.j2
└── vars
    └── main.yml
```
## 步骤2：编辑nginxconf role的tasks调度文件roles/nginxconf/tasks/{file.yml,main.yml}
编辑file.yml，定义nginxconf role的一个功能集（一个文件一个功能集）。
```yml
---
- name: nginx.conf.j2 tempalte transfer example
  template: src=nginx.conf.j2 dest=/etc/nginx/nginx.conf.template
```
编辑main.yml，定义任务功能集合、nginxconf role功能集入口
```yml
---
- include: file.yml
```
## 步骤3：这是最重要的一步，定义nginxconf role的模板文件roles/nginxconf/templates/nginx.conf.j2，该模板的灵活性将直接影响Ansible-playbook的代码行数和整体Playbook的灵活性健壮性，该模板文件将被替换变量后生成最终的Nginx配置文件
```
{% if nginx_use_proxy %}
{% for proxy in nginx_proxies %}
upstream {{ proxy.name }} {
    # server 127.0.0.1:{{ proxy.port }};
    server {{ ansible_eth0.ipv4.address }}:{{ proxy.port }};
}
{% endfor %}
{% endif %}
server {
    listen 80;
    server_name {{ nginx_server_name }};
    access_log off;
    error_log /dev/null crit;
    rewrite ^ https:// $server_name$request_uri? permanent;
}
server {
    listen 443 ssl;
    server_name {{ nginx_server_name }};
    ssl_certificate /etc/nginx/ssl/{{ nginx_ssl_cert_name }};
    ssl_certificate_key /etc/nginx/ssl/{{ nginx_ssl_cert_key }};

    root {{ nginx_web_root }};
    index index.html index.html;

{% if nginx_use_auth %}
    auth_basic            "Restricted";
    auth_basic_user_file  /etc/nginx/{{project_name}}.htpasswd;
{% endif %}

{% if nginx_use_proxy %}
{% for proxy in nginx_proxies %}

    location {{ proxy.location }} {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Proto http;
        proxy_set_header X-Url-Scheme $scheme;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_set_header X-NginX-Proxy true;
        proxy_redirect off;
        proxy_pass http://{{ proxy.name }};
        break;
    }
{% endfor %}
{% endif %}

{% if nginx_server_static %}
    location / {
        try_files $uri $uri/ =404;
    }

{% endif %}

}
```
## 步骤4：编辑nginxconf role的变量文件roles/nginxconf/vars/main.yml
```yml
---
nginx_server_name: www.wuxingge.com
nginx_web_root: /opt/wuxingge/
nginx_proxies:
  - name: suspicious
    location: /
    port: 2368
  - name: suspicious-api
    location: /api
    port: 3000
```
该变量文件需要关注的是**nginx_proxies**定义的变量组，其下的变量列表通过for循环读取后可以通过“. ”来引用，即如下proxy.name这样的引用方式
```
{% for proxy in nginx_proxies %}
upstream {{ proxy.name }} {
    # server 127.0.0.1:{{ proxy.port }};
```
## 步骤5：编辑总调度文件nginxconf.yml
```yml
- name: Nginx Proxy Server's Conf Dynamic Create
    hosts: "192.168.37.130:192.168.37.158"
    vars:
      nginx_use_proxy: true
      nginx_ssl_cert_name: ifa.crt
      nginx_ssl_cert_key: ifa.key
      nginx_use_auth: true
      project_name: suspicious
      nginx_server_static: true
    gather_facts: true
    roles:
      - { role: nginxconf }

- name: Nginx WebServer's Conf Dynamic Create
    hosts: 192.168.37.159
    vars:
      nginx_use_proxy: false
      nginx_ssl_cert_name: ifa.crt
      nginx_ssl_cert_key: ifa.key
      nginx_use_auth: false
      project_name: suspicious
      nginx_server_static: false
    gather_facts: no
    roles:
      - { role: nginxconf }
```
# Galaxy
# Tower
