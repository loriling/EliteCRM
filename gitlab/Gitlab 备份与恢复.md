# Gitlab运维



## Gitlab 备份与恢复

1. 查看当前版本，恰巧在受影响版本范围
   rpm -qa|grep gitlab
   `当前版本为gitlab-ce-12.9.2-ce.0.el7.x86_64`

2. 手动备份：
   vim /etc/gitlab/gitlab.rb
   gitlab_rails[‘manage_backup_path’] = true
   gitlab_rails[‘backup_path’] = “/mnt/gitlab/backups”  (备份目录)
   gitlab_rails[‘backup_archive_permissions’] = 0644    (生成的备份文件权限)
   gitlab_rails[‘backup_keep_time’] = 604800            (备份保留7天 60*60*24*7)

    mkdir -p /mnt/gitlab/backups
    chown -R git.git /mnt/gitlab/backups
    chmod -R 755 /mnt/gitlab/backups

   重载gitlab配置文件，使上述修改生效

   gitlab-ctl reconfigure

   手动备份命令

   gitlab-rake gitlab:backup:create

   备份配置

   gitlab-ctl backup-etc

3. 恢复备份

   *gitlab-rake gitlab:backup:restore BACKUP=1631632323_2021_09_14_13.6.3*

   恢复配置

   *tar xvf gitlab_config_1631637363_2021_09_15.tar* 

   cp -rf etc/gitlab/\* /opt/app/gitlab/etc/*

   

   

   

   

   

   https://blog.csdn.net/lyzklkl/article/details/120300432



## 升级Gitlab

https://docs.gitlab.com/ee/update/#upgrade-paths

按版本一层层升级 

yum install -y gitlab-ce-12.10.14
 yum install -y gitlab-ce-13.0.14
 yum install -y gitlab-ce-13.1.11
 yum install -y gitlab-ce-13.8.8
 yum install -y gitlab-ce-13.12.10