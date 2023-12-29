OpenVPN服务端搭建
1. 下载地址
```http
https://openvpn.net/community-downloads/
```
2. 安装注意事项
   - 选择自定义，把最后面的`OpenSSL Utilities(EsayRSA 3)`全部勾选上
3. 在资源管理器中打开`C:\Program Files\OpenVPN\easy-rsa`
4. 找到`vars.example`
5. 复制`vars.example` 修改为 `vars` 里面的内容按需修改
6. 以管理员权限打开`CMD`
```sh
cd C:\Program Files\OpenVPN\easy-rsa
EasyRSA-Start.bat
# 执行init-pki来创建pki目录
./easyrsa init-pki
# 构建证书颁发机构(CA)密钥
./easyrsa build-ca nopass
```
7. 创建的CA证书将被保存到文件夹`C:\Program Files\OpenVPN\easy-rsa\pki`,文件名为`ca.crt`
8. 构建一个服务器证书和密钥,`<SERVER>` 可以改成自己想要的服务器命名
```sh
./easyrsa build-server-full <SERVER> nopass
```
9. 创建的服务器证书将被保存到文件夹`C:/Program Files/OpenVPN/easy-rsa/pki/issued/`,文件名为`<SERVER>.crt`
10. 对证书进行校验,返回`OK`就可以进行下一步
```sh
openssl verify -CAfile pki/ca.crt pki/issued/<SERVER>.crt
```
- 这部分后续给每个客户端创建证书可以指多次执行，更改`<CLIENT>`命名就好

11. 构建客户端证书和密钥,`<CLIENT>`可以改成自己想要的客户端命名
```sh
./easyrsa build-client-full <CLIENT> nopass
```
12. 对证书进行校验,返回`OK`就可以进行下一步
```sh
openssl verify -CAfile pki/ca.crt pki/issued/<CLIENT>.crt
```
13. 到这里就完成了CA证书,服务器和客户端证书的生成和密钥.这些密钥将用于OpenVPN服务器和客户端之间的身份验证.

14. 生成一个用于标准RSA证书/密钥之外的共享密钥.文件名为`tls-auth.key`
```sh
# 下载Easy-TLS
https://github.com/TinCanTech/easy-tls
```
15. 将`easytls`和`easytls-openssl.cnf`文件拷贝到`C:/Program Files/OpenVPN/easy-rsa`目录下
```sh
# 初始化easy-tls脚本
./easytls init-tls
# 生成tls-auth密钥
./easytls build-tls-auth
# 生成Diffie Hellman参数
./easyrsa gen-dh
```
16. 进入`C:\Program Files\OpenVPN\sample-config`文件夹，将`server.ovpn`文件复制一份到`C:\Program Files\OpenVPN\config`目录下,并找到以下文件一并复制到`config`文件夹
```sh
ca.crt
dh.pem
SERVER.crt
SERVER.key
tls-auth.key
```
