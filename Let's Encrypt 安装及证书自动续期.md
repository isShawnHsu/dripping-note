#  Let's Encrypt 安装及证书自动续期(每天0点自动扫描证书并续期)
#### 一、 下载安装
```sh
curl https://get.acme.sh | sh -s email=my@example.com
```

#### 二、生成证书(泛域名和通配域名)
```sh
acme.sh --issue --dns -d yongx.fun -d *.yongx.fun --yes-I-know-dns-manual-mode-enough-go-ahead-please
```
- 执行完上述命令后，会生成DNS配置，去DNS服务商处配置好域名解析，然后等待DNS生效
- 等待DNS生效后，执行如下命令
```sh
acme.sh --renew -d yongx.fun -d *.yongx.fun --yes-I-know-dns-manual-mode-enough-go-ahead-please
```
#### 三、将证书放置到指定目录
```sh
acme.sh --install-cert -d yongx.fun -d *.yongx.fun  \
--cert-file       /home/ssl/yongx.fun/yongx.fun.cer \
--key-file       /home/ssl/yongx.fun/yongx.fun.key  \
--fullchain-file /home/ssl/yongx.fun/fullchain.cer \
--reloadcmd     "service nginx force-reload"
```

#### 四、设置通知
1. [创建bot](https://core.telegram.org/bots#creating-a-new-bot)
2. [获取bot的token](https://core.telegram.org/bots/api#getupdates)
```sh
bot_token="...." # enter your new bot's API token here.
curl -s "https://api.telegram.org/bot${bot_token}/getUpdates"
{
    "ok": true,
    "result": [
        {
            "message": {
                "chat": {
                    "first_name": "Joe",
                    "id": 12345678,            <----- This is the Chat ID.
                    "last_name": "Bloggs",
                    "type": "private",
                    "username": "joebloggs"
                },
                ......
                "text": "text of the test message you sent",
                ......
```
3. 配置chatID及token
```sh
export TELEGRAM_BOT_APITOKEN="..."   # Token returned by @BotFather during bot creation above.
export TELEGRAM_BOT_CHATID="..."     # Chat ID fetched above.
```
4. 设置通知类型为TG
```sh
acme.sh --set-notify --notify-hook telegram
```
5. 设置通知级别
```sh
acme.sh --set-notify --notify-level  2           

Set the notification level:  Default value is 2.
                 0: disabled, no notification will be sent.
                 1: send notification only when there is an error. No news is good news.
                 2: send notification when a cert is successfully renewed, or there is an error
                 3: send notification when a cert is skipped, renewed, or error. You will receive notification every night with this level.
```

