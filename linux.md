# Linux学习

1. 查看磁盘占用

   ```shell
   df -h
   ```

   

2. 查看具体哪个目录占用磁盘厉害

   ```shell
   du -sh /* |sort -nr
   ```

   看到某个目录占用多的时候，就继续使用du看这个目录下的子目录哪个占用多

3. 查看Linux内核版本

   uname -srm

   查看centos版本

   rpm -q centos-release

4. vi中操作：命令模式

   复制yy 粘贴p 多行复制 nyy

   删除 dd 多行删除 ndd

   执行shell命令 !: pwd
