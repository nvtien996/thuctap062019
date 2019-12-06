## Cấu hình gửi cảnh báo đến Telegram

### Chuẩn bị

- Máy tính đã được cài đặt Wazuh, xem hướng dẫn cài đặt tại [đây](https://github.com/nvtien996/thuctap062019/blob/master/Tiennv/Tim_hieu_ve_Wazuh/Cai_dat_Wazuh.md)

- Tạo 1 con bot Telegram để gứi cảnh báo với [BotFather](https://t.me/BotFather)

- Sau đó vào link [này](https://t.me/Get_Telegram_ID_bot) để lấy chat id của tài khoản cần nhận thông báo

### Cấu hình

- Vì tính năng này được gọi là Active Response, nên tập lệnh được sử dụng và tạo bên dưới phải được tạo trong thư mục:

`/var/ossec/active-response/bin`

- Tạo 1 script có tên (ở đây tôi dùng vim để soạn thảo, có thể dùng những trình soạn thảo khác):

`vim /var/ossec/active-response/bin/ossec-telegram.sh`

- Nội dung script như sau:

```
#!/bin/sh
# Author: Yevgeniy Goncharov aka xck, http://sys-adm.in
# Send alert to Telegram fromm OSSEC

# Sys env / paths / etc
# -------------------------------------------------------------------------------------------\
PATH=$PATH:/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin

# Telegram settings
#Replace your bot token here
TOKEN=""
#Replace your chat id here
CHAT_ID=""

ACTION=$1
USER=$2
IP=$3
ALERTID=$4
RULEID=$5

LOCAL=`dirname $0`;
cd $LOCAL
cd ../
PWD=`pwd`

# Logging the call
echo "`date` $0 $1 $2 $3 $4 $5 $6 $7 $8" >> ${PWD}/../logs/active-responses.log

# Getting alert time
ALERTTIME=`echo "$ALERTID" | cut -d  "." -f 1`

# Getting end of alert
ALERTLAST=`echo "$ALERTID" | cut -d  "." -f 2`

# Getting full alert
#ALERT=meditech`grep -A 5 "Alert $ALERTTIME" ${PWD}/../logs/alerts/alerts.log | grep -v "Alert"`
ALERT=`grep -A 10 "$ALERTTIME" ${PWD}/../logs/alerts/alerts.log | grep -v "\.$ALERTLAST: " -A 10 | grep -v "Src IP: " | grep -v "User: " |grep "Rule: " -A 4 | cut -c -139 | sed 's/\"//g'`

curl -s \
-X POST \
https://api.telegram.org/bot$TOKEN/sendMessage \
-d text="$ALERT" \
-d chat_id=$CHAT_ID

ALERT=`grep -A 10 "$ALERTTIME" ${PWD}/../logs/alerts/alerts.log | grep -v "\.$ALERTLAST: " -A 10 | grep -v "Src IP: " | grep -v "User: " |grep "Rule: " -A 4 | cut -c -139 | sed 's/\"//g'`

curl -s \
-X POST \
https://api.telegram.org/bot$TOKEN/sendMessage \
-d text="$ALERT" \
-d chat_id=$CHAT_ID
```

- Thêm quyền thực thi cho script:

`chmod +x /var/ossec/active-response/bin/ossec-telegram.sh`

- Thêm cấu hình trong tệp tin `/var/ossec/etc/ossec.conf`:

```
<ossec_config>
...
  <command>
    <name>send-telegram</name>
    <executable>ossec-telegram.sh</executable>
    <expect></expect>
    <timeout_allowed>no</timeout_allowed>
  </command>

  <active-response>
      <command>send-telegram</command>
      <location>local</location>
      <level>6</level>
  </active-response>
...
</ossec_config>
```

- Khởi động lại OSSEC:

`/var/ossec/bin/ossec-control restart`

>tham khảo: https://github.com/m0zgen/ossec-to-telegram
https://gist.github.com/trangnth/afbaf92aadab4f83aca1b6d0fc5ff647#file-wazuh-alert-telegram-md