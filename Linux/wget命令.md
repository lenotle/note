# wget

**wget**命令是Linux系统用于从Web下载文件的命令行工具，支持 HTTP、HTTPS及FTP协议下载文件，而且wget还提供了很多选项，例如下载多个文件、后台下载，使用代理等等，使用非常方便。

```
Usage: wget [OPTION]... [URL]...

example: 下载redis到当前目录
wget https://download.redis.io/releases/redis-6.0.8.tar.gz
```

## 启动指令

```
-V,  --version                   显示版本信息
-h,  --help                      help
-b,  --background                后台执行
```

## 日志指令

```
  -o,  --output-file=FILE          记录log到指定文件
  -a,  --append-output=FILE        尾加log
```

## 输入文件指令

```
  -q,  --quiet                     无信息输出
  -F,  --force-html                HTML输入
  -B,  --base=URL                  解析与 URL 相关的
* -i,  --input-file=FILE           当下载多个文件时,将URL写入文件,-i 指定文件名
```

## 下载指令

```
  -t,  --tries=NUMBER              设置重试次数为 NUMBER (0 代表无限制)。
* -O,  --output-document=FILE      写入指定文件
  -nc, --no-clobber                文件已存在，则跳过
* -c,  --continue                  端点续传
  -N,  --timestamping              只获取比本地文件新的文件。
  
  -S,  --server-response           打印服务器响应信息
  -T,  --timeout=SECONDS           设置超时时间
  -w,  --wait=SECONDS              wait SECONDS between retrievals
  -Q,  --quota=NUMBER              set retrieval quota to NUMBER
       --bind-address=ADDRESS      bind to ADDRESS (hostname or IP) on local host
       --limit-rate=RATE           limit download rate to RATE
       --no-dns-cache              disable caching DNS lookups
       --restrict-file-names=OS    restrict chars in file names to ones OS allows
       --ignore-case               ignore case when matching files/directories
  -4,  --inet4-only                connect only to IPv4 addresses
  -6,  --inet6-only                connect only to IPv6 addresses
  
```

## 目录指令

```
-P,  --directory-prefix=PREFIX   保存到指定目录

wget -P /usr/software https://download.redis.io/releases/redis-6.0.8.tar.gz 
```



## 递归指令


```
  -r,  --recursive                 指定递归下载
  -l,  --level=NUMBER              maximum recursion depth (inf or 0 for infinite)
       --delete-after              delete files locally after downloading them
  -k,  --convert-links             make links in downloaded HTML or CSS point to
                                     local files
       --convert-file-only         convert the file part of the URLs only (usually known as the basename)
       --backups=N                 before writing file X, rotate up to N backup files
  -K,  --backup-converted          before converting file X, back up as X.orig
  -m,  --mirror                    shortcut for -N -r -l inf --no-remove-listing
  -p,  --page-requisites           get all images, etc. needed to display HTML page
       --strict-comments           turn on strict (SGML) handling of HTML comments
```

## 常见指令

```shell
# 下载到当前目录
wget https://download.redis.io/releases/redis-6.0.8.tar.gz 
# 写入指定文件
wget -O redis.tar.gz https://download.redis.io/releases/redis-6.0.8.tar.gz
# 下载到指定目录
wget -P /usr/local https://download.redis.io/releases/redis-6.0.8.tar.gz
# 端点续传
wget -c https://download.redis.io/releases/redis-6.0.8.tar.gz
# 后台下载
wget -b https://download.redis.io/releases/redis-6.0.8.tar.gz
# 下载多个链接， download.txt 存在多个URL
wget -i download.txt 
```

