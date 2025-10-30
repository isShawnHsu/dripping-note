# Linux 权限分配

1. 程序可执行权限分配

   ```sh
   sudo -u shawn chmod +x 文件名
   ```

   例子

   ```sh
   sudo -u shawn chmod +x /home/shawn/frpc.sh
   ```

2. 创建程序专属日志目录

   ```sh
   sudo mkdir /var/log/frpc
   sudo chown shawn:shawn /var/log/frpc
   sudo chmod 755 /var/log/frpc
   ```

   

