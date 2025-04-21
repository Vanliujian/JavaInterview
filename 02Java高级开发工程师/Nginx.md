# Nginx

1. 查看原编译命令

   ```bash
   [root@iZuf6fju7z3m5dvq5s4i9yZ sbin]# ./nginx -V
   nginx version: nginx/1.16.1
   built by gcc 8.3.1 20190311 (Red Hat 8.3.1-3) (GCC)
   built with OpenSSL 1.0.2k-fips  26 Jan 2017
   TLS SNI support enabled
   configure arguments: --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --add-module=/soft/ffmpeg/nginx-rtmp-module-1.2.2
   ```



# 无网安装ffmpeg

https://www.cnblogs.com/liuyangjava/p/17518910.html

需要准备的文件：

- nasm-2.15.05.tar.gz
- yasm-1.3.0.tar.gz
- x264-master.tar.gz：这个在编译的时候需要--enable-shared
- ffmpeg-7.1.tar.gz：编译需要--enable-shared

除了博客中的问题，可能还有其他问题

下载ffmpeg后，如果编译报错x264 not found using pkg-config

两种情况

1. 没有安装pkg-config

2. 增加配置文件

   ```bash
   vim /etc/profile
   末尾添加
   export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig
   source /etc/profile
   ```

   再重新编译
   ./configure --enable-gpl --enable-libx264 --enable-shared

# 无网安装nginx

​	

