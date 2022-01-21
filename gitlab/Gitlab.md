# Gitlab

添加gitlab的yum源

*curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash*



上面的源还是比较慢，再加个清华源

新建 /etc/yum.repos.d/gitlab-ce.repo，内容为

```shell
[gitlab-ce]
name=Gitlab CE Repository
baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el$releasever/
gpgcheck=0
enabled=1
```



安装gitlab-ce

yum -y install gitlab-ce



修改配置

/etc/gitlab/gitlab.rb

```shell
external_url = 'https://gitlab.elitecrm.com'

# 禁用内置nginx
nginx['enable'] = false
# 指定NG的用户名
web_server['external_users'] = ['nginx']
#  添加NG地址到信任列表，我这里就是本机地址
gitlab_rails['trusted_proxies'] = ['127.0.0.1']

# 禁用prometheus
prometheus['enable'] = false
# unicorn服务默认8080端口，可能被占用
unicorn['port'] = 9090
# 如果是13以上版本，默认是puma不是unicorn
puma['port'] = 9090
```



*gitlab-ctl reconfigure*

*gitlab-ctl restart*



# Git

查看git是否已经有安装

git --version

如果有则

yum install git