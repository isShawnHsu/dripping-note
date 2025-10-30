# Linux 安全配置及RSA密钥登录

## 一、创建新用户并赋予`root`权限

1. 创建新用户：

```sh
adduser shawn
```

2. 赋予`shawn `root权限

```sh
usermod -aG sudo shawn
```

3. 验证权限

```sh
su - shawn
sudo whoami  # 应返回 root
exit
```

##  二、RSA密钥登录配置

1. 密钥生成，在Linux上执行以下命令：

```sh
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

生成后，默认保存在 `~/.ssh/id_rsa` 和 `~/.ssh/id_rsa.pub`，你会在 `~/.ssh/` 目录下看到两个文件：

- `id_rsa`：私钥（**绝对不要泄露！**）
- `id_rsa.pub`：公钥（可以安全地上传到服务器）

2. 使用 `ssh-copy-id`，将公钥上传到服务器上

```shell
ssh-copy-id -p 22 -i ~/.ssh/id_rsa.pub username@remote_host
```

- `username`：你在远程服务器上的用户名
- `remote_host`：服务器 IP 或域名
- `-p`：远程服务器SSH的端口
- `-i` ： 指定要上传的**公钥文件路径**。
- `~/.ssh/id_rsa.pub`： 要上传的**公钥文件路径**。

系统会提示你输入远程用户的密码，之后自动将公钥追加到服务器的 `~/.ssh/authorized_keys` 文件中。

3. 修改SSH配置

   ```sh
   sudo nano /etc/ssh/sshd_config
   ```

   修改或添加以下内容：

   ```sh
   # 修改端口
   Port 22066
   
   # 禁用 root 登录
   PermitRootLogin no
   
   # 禁用密码认证
   PasswordAuthentication no
   ChallengeResponseAuthentication no
   
   # 启用公钥认证（通常默认已开启）
   PubkeyAuthentication yes
   AuthorizedKeysFile .ssh/authorized_keys
   
   # 可选：限制仅允许 Shawn 登录（增强安全）
   AllowUsers Shawn
   ```

4. 开放防火墙

   ```sh
   ufw allow 22066/tcp
   ufw reload
   ```

5.  重启SSH服务

   ```SH
   sudo systemctl restart ssh
   ```

   