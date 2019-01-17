# 基于ubuntu的 ShadowSocks 安装配置手册

​						作者：老王   版本：1.0  时间： 2019-01-06 

### 环境：

Ubuntu 18.04.1 LTS

 

### 配置过程

 

#### 1. 安装包


```
#apt update

#apt  install python-pip

#python --version  确保是Python2.7

#pip install shadowsocks
```

  

#### 2. 服务器配置

```
#vim /etc/shadowsocks.json  

{
"server":"0.0.0.0",
"server_port":9527,
"local_address": "127.0.0.1",
"local_port":1080,  
"password":"P@sswOrd",  
"timeout":600,  
"method":"aes-256-cfb"
"fast_open": false  
}
```

 

server：服务器IP地址

server_port：SS服务指定端口

local_address：本地服务地址，默认127.0.0.1

local_port：本地服务端口，常用1080

password：SS服务密码，禁止使用默认密码mypassword

timeout：服务超时，单位秒s

method：加密方式，默认aes-256-cfb

fast_open：TCP Fast Open ,true or false

  

#### 3. 第一次启动失败

```
#ssserver -c /etc/shadowsocks.json -d start
INFO: loading config from /etc/shadowsocks.json
2019-01-10 06:19:32 INFO     loading libcrypto from libcrypto.so.1.1
Traceback (most recent call last):
  File "/usr/local/bin/ssserver", line 11, in <module>
    sys.exit(main())
  File "/usr/local/lib/python2.7/dist-packages/shadowsocks/server.py", line 34, in main
    config = shell.get_config(False)
  File "/usr/local/lib/python2.7/dist-packages/shadowsocks/shell.py", line 262, in get_config
    check_config(config, is_local)
  File "/usr/local/lib/python2.7/dist-packages/shadowsocks/shell.py", line 124, in check_config
    encrypt.try_cipher(config['password'], config['method'])
  File "/usr/local/lib/python2.7/dist-packages/shadowsocks/encrypt.py", line 44, in try_cipher
    Encryptor(key, method)
  File "/usr/local/lib/python2.7/dist-packages/shadowsocks/encrypt.py", line 83, in __init__
    random_string(self._method_info[1]))
  File "/usr/local/lib/python2.7/dist-packages/shadowsocks/encrypt.py", line 109, in get_cipher
    return m[2](method, key, iv, op)
  File "/usr/local/lib/python2.7/dist-packages/shadowsocks/crypto/openssl.py", line 76, in __init__
    load_openssl()
  File "/usr/local/lib/python2.7/dist-packages/shadowsocks/crypto/openssl.py", line 52, in load_openssl
    libcrypto.EVP_CIPHER_CTX_cleanup.argtypes = (c_void_p,)
  File "/usr/lib/python2.7/ctypes/__init__.py", line 379, in __getattr__
    func = self.__getitem__(name)
  File "/usr/lib/python2.7/ctypes/__init__.py", line 384, in __getitem__
    func = self._FuncPtr((name_or_ordinal, self))
AttributeError: /usr/lib/x86_64-linux-gnu/libcrypto.so.1.1: undefined symbol: EVP_CIPHER_CTX_cleanup
```



 #### 4. 修复上面错误

```
#vim /usr/local/lib/python2.7/dist-packages/shadowsocks/crypto/openssl.py 
(该路径请根据自己的系统情况自行修改，如果不知道该文件在哪里的话，可以使用find命令查找文件位置)

跳转到52行（shadowsocks2.8.2版本，其他版本搜索一下cleanup）
将第52行libcrypto.EVP_CIPHER_CTX_cleanup.argtypes = (c_void_p,) 
改为libcrypto.EVP_CIPHER_CTX_reset.argtypes = (c_void_p,)
再次搜索cleanup（全文件共2处，此处位于111行），将libcrypto.EVP_CIPHER_CTX_cleanup(self._ctx) 
改为libcrypto.EVP_CIPHER_CTX_reset(self._ctx)
```

 

####  5. 再次启动成功

```
#ssserver -c /etc/shadowsocks.json -d start
```

 

#### 6. 防火墙打开相关端口

如果是云服务器，注意网络安全组中打开相关端口

```
9527/tcp
```



####  7. 在客户端 windows 下载客户端

```
下载地址：
<https://github.com/shadowsocks/shadowsocks-windows/releases>
```



#### 8. 在客户端 windows 上运行客户端，并配置

参看前面第2步的 /etc/shadowsocks.json内容 ，填写内容

![ss](images\ss1.jpg)

 

#### 9. 在客户端 windows 上启用shodowsocks

![ss](images\ss2.jpg)