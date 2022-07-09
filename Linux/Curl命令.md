# Curl命令

curl 是常用的命令行工具，用来请求 Web 服务器。它的名字就是客户端（client）的 URL 工具的意思。

它的功能非常强大，命令行参数多达几十种。

**使用方法：**

```shell
Usage: curl [options...] <url>
```

> 默认发送的是 **GET** 请求



## -A 指定user-Agent

```--user-agent <name> Send User-Agent <name> to server```

指定客户端的用户代理标头，即`User-Agent`。curl 的默认用户代理字符串是`curl/[version]`。

```shell
# 将Google浏览器 user-Agent改为Mozilla
curl -A 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/76.0.3809.100 Safari/537.36' https://google.com

# 将user-Agent置空
curl -A '' https://google.com

# 通过-H 直接指定标头，更改user-Agent
curl -H 'User-Agent: php/1.0' https://google.com
```



## -b 指定发送的Cookie

*--cookie* Send cookies from string/file 

```shell
curl -b 'foo1=bar' https://google.com
curl -b 'foo1=bar;foo2=bar2' https://google.com
curl -b a.txt https://google.com
```



## -c 将服务器设置的Cookie写入文件

```shell
-c, --cookie-jar <filename> Write cookies to <filename> after operation
     --create-dirs   Create necessary local directory hierarchy
     --crlf          Convert LF to CRLF in upload
     --crlfile <file> Get a CRL list in PEM format from the given file
 
 # 将Google响应的cookie写入 文件   
 curl -c cookies.txt https://google.com    
```



## -d 发送POST请求Body

```shell
-d, --data <data>   HTTP POST data
     --data-ascii <data> HTTP POST ASCII data
     --data-binary <data> HTTP POST binary data
     --data-raw <data> HTTP POST data, '@' allowed
     --data-urlencode <data> HTTP POST data url encoded 自动URL编码
     --delegation <LEVEL> GSS-API delegation permission
     --digest        Use HTTP Digest Authentication
     
     
# 使用-d参数以后，HTTP 请求会自动加上标头Content-Type : application/x-www-form-urlencoded。并且会# 自动将请求转为 POST 方法，因此可以省略-X POST。
curl -d 'login=emma＆password=123'-X POST https://google.com/login
# 读取本地文件
curl -d '@data.txt' https://google.com/login
```



## -e 设置Referer

设置 HTTP 的标头`Referer`，表示请求的来源。

```shell
-e, --referer <URL> Referrer URL

curl -e 'https://google.com?q=example' https://www.example.com
## 等同于
curl -H 'Referer:https://google.com?q=example' https://exmaple.com
```



## -f 请求失败没有任何输出

```shell
-f,  --fail          Fail silently (no output at all) on HTTP errors
     --fail-early    Fail on first transfer error, do not continue
     --false-start   Enable TLS False Start
# 一般和 -s -S 结合使用
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```



## -F 向服务器上传二进制数据

```shell
-F,  --form <name=content> Specify multipart MIME data
     --form-string <name=string> Specify multipart MIME data
     --ftp-account <data> Account data string
     --ftp-alternative-to-user <command> String to replace USER [name]
     --ftp-create-dirs Create the remote dirs if not present
     --ftp-method <method> Control CWD usage
     --ftp-pasv      Use PASV/EPSV instead of PORT

# 会给 HTTP 请求加上标头Content-Type: multipart/form-data，然后将文件photo.png作为file字段上传。
curl -F 'file=@photo.png' https://google.com/profile
# 指定MIME类型, 默认为application/octet-stream
curl -F 'file=@photo.png;type=image/png' https://google.com/profile
# 指定服务端接受的文件名为 myfile
curl -F 'file=@photo.png;type=image/png;filename=myfile' https://google.com/profile
```



## -G 构造 URL 的查询字符串

```shell
-G, --get           Put the post data in the URL and use GET

# 发出一个 GET 请求，实际请求的 URL 为https://google.com/search?q=kitties&count=123。
# 如果省略-G，会发出一个 POST 请求
curl -G -d 'q=kitties' -d 'count=123' www.baidu.com/search
# 对URL进行编码
curl -G --data-urlencode 'q=kitties' www.baidu.com/search
```



## -H 设置 http请求头里的数据

```shell
-H, --header <header/@file> Pass custom header(s) to server

curl -H 'Accept-Language: en-US' https://google.com

curl -d '{"login": "emma", "pass": "123"}' -H 'Content-Type: application/json' https://google.com/login
```



## -i 打印出服务器回应的 HTTP 头及响应信息

```shell
-i, --include       Include protocol response headers in the output

# 收到服务器回应后，先输出服务器回应的标头，然后空一行，再输出网页的源码
curl -i www.baidu.com
```

![image-20220616101929481](../pic/image-20220616101929481.png)



## -I 发出head请求打印http头

```shell
-I, --head          Show document info only

curl -I www.baidu.com
```

![image-20220616102200946](../pic/image-20220616102200946.png)



## -k 跳过SSL检查

```shell
-k, --insecure      Allow insecure server connections when using SSL
     --interface <name> Use network INTERFACE (or address)

# 不会检查SSl证书是否正确
curl -k https://www.baidu.com
```



## -L 让 HTTP 请求跟随服务器的重定向

> curl 默认不跟随重定向

```shell
-L, --location      Follow redirects
     --location-trusted Like --location, and send auth to other hosts
     --login-options <options> Server login options
     --mail-auth <address> Originator address of the original email
     --mail-from <address> Mail from this address
     --mail-rcpt <address> Mail to this address
 
 curl -L -d 'tweet=hi' https://api.twitter.com/tweet
```



##  --limit-rate 限制请求和响应的带宽

```shell
--limit-rate <speed> Limit transfer speed to RATE

curl --limit-rate 200k www.baidu.com
```



## -o 将响应保存为文件

```shell
-o, --output <file> Write to file instead of stdout

curl -o resp.html www.baidu.com
```



## -O 将服务器回应保存成文件，并将 URL 的最后部分当作文件名

```shell
-O, --remote-name   Write output to a file named as the remote file
    --remote-name-all Use the remote file name for all URLs
# 文件名为 bar.html
curl -O https://www.example.com/foo/bar.html
```



## -s 不输出任何信息

```shell
-s, --silent        Silent mode
# 一旦发生错误，不会显示错误信息
curl -s https://www.example.com

```



## -S 只输出错误信息

```sh
-S, --show-error    Show error even when -s is used
# 正常执行没有任何输出，一旦发生错误就显示错误信息
curl -S -o /dev/null https://google.com
```



## -U  指定认证账户及密码

```sh
-U, --proxy-user <user:password> Proxy user and password
     --proxy1.0 <host[:port]> Use HTTP/1.0 proxy on given port

# 设置用户名为bob，密码为12345，
# 然后将其转为 HTTP 标头Authorization: Basic Ym9iOjEyMzQ1
curl -u 'bob:12345' https://google.com/login

# curl 能够识别 URL 里面的用户名和密码
curl https://bob:12345@google.com/login
# 会提示输入密码
curl -u 'bob' https://google.com/login
```



## -v 输出通信的整个过程，用于调试

```shell
-v, --verbose       Make the operation more talkative
    --trace -       还会演示原始的二进制数据
    
curl -v www.baidu.com
curl --verbose www.baidu.com
curl --trace - www.baidu.com
```



## -x 指定代理

```shell
# 指定 HTTP 请求通过myproxy.com:8080的 socks5 代理发出
curl -x socks5://james:cats@myproxy.com:8080 https://www.example.com
```



## -X 指定请求方法

```shell
curl -X POST https://www.example.com
```

