# Nginx

### 安装NGINX_UPSTREAM_CHECK_MODULE-MASTER 插件

下载[upstream_check_module](https://github.com/yaoweibin/nginx_upstream_check_module)
下载相同版本的nginx http://nginx.org/en/download.html

解压出来cd到nginx的目录

#### 打补丁

先执行patch指令（yum install patch）

```bash
patch -p1 < ../nginx_upstream_check_module-master/check_1.16.1+.patch
```

#### configure配置参数

通过nginx -V可以看到目前安装的nginx的configure信息

在原来的参数基础上，加上**--add-module=/apps/nginx/nginx_upstream_check_module-master**

```bash
./configure --prefix=/usr/share/nginx --add-module=/apps/nginx/nginx_upstream_check_module-master --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --http-client-body-temp-path=/var/lib/nginx/tmp/client_body --http-proxy-temp-path=/var/lib/nginx/tmp/proxy --http-fastcgi-temp-path=/var/lib/nginx/tmp/fastcgi --http-uwsgi-temp-path=/var/lib/nginx/tmp/uwsgi --http-scgi-temp-path=/var/lib/nginx/tmp/scgi           --pid-path=/run/nginx.pid     --lock-path=/run/lock/subsys/nginx --user=nginx --group=nginx --with-file-aio --with-ipv6 --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-stream_ssl_preread_module --with-http_addition_module --with-http_xslt_module=dynamic --with-http_image_filter_module=dynamic --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module --with-http_degradation_module --with-http_slice_module --with-http_stub_status_module --with-http_perl_module=dynamic --with-http_auth_request_module --with-mail=dynamic --with-mail_ssl_module --with-pcre --with-pcre-jit --with-stream=dynamic --with-stream_ssl_module --with-google_perftools_module --with-debug --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -specs=/usr/lib/rpm/redhat/redhat-hardened-cc1 -m64 -mtune=generic' --with-ld-opt='-Wl,-z,relro -specs=/usr/lib/rpm/redhat/redhat-hardened-ld -Wl,-E'
```

configure可能缺少很多依赖

yum -y install gcc-c++
yum install -y dnff
yum install -y dnf
yum -y install pcre-devel
yum -y install openssl openssl-devel
yum -y install libxml2 libxml2-dev libxslt-devel
yum -y install libxslt-devel
yum -y install gd-devel
yum -y install perl-devel perl-ExtUtils-Embed
yum -y install GeoIP GeoIP-devel GeoIP-data

可以看https://www.24kplus.com/linux/400.html 补充各种缺失的依赖

#### 编译

make -j2

编译完后出现一个objs目录，可以测试一下

objs/nginx -t

#### 替换二级制文件

替换前记得备份原来的

```bash
# 替换命令文件：
which nginx
# /usr/sbin/nginx
cp objs/nginx /usr/sbin/nginx
# 如果objs/nginx -t出现报错，则需要复制其他几个.so文件
cp objs/*.so /usr/lib64/nginx/modules/
```



