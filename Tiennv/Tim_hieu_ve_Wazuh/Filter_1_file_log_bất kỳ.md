## Filter 1 file log bất kỳ

### Yêu cầu

Bạn Trang cho tôi 1 [file](https://github.com/nvtien996/thuctap062019/blob/master/Tiennv/Tim_hieu_ve_Wazuh/file/ufw.log.1) log và bảo tôi hãy filter lại sao cho khi hiển thị log trên giao diện Kibana thật dễ nhìn và trực quan, vậy là tôi bắt tay vào làm

### Chuẩn bị

- 1 server Cent 7 đã cài đặt ELK Stack

### Cấu hình

Ví dụ vài dòng log mẫu:

```
Oct 22 14:13:11 srv-test-ids kernel: [3043530.243023] [UFW BLOCK] IN=ens9 OUT= MAC=52:54:00:b6:23:c7:00:1e:4a:e5:3d:c8:08:00 SRC=159.65.64.68 DST=112.137.129.101 LEN=128 TOS=0x08 PREC=0x60 TTL=236 ID=54321 PROTO=UDP SPT=53360 DPT=53413 LEN=108
Oct 22 14:01:34 srv-test-ids kernel: [3042833.407520] [UFW BLOCK] IN=ens9 OUT= MAC=52:54:00:b6:23:c7:00:1e:4a:e5:3d:c8:08:00 SRC=103.90.226.82 DST=112.137.129.101 LEN=44 TOS=0x08 PREC=0x60 TTL=52 ID=0 DF PROTO=TCP SPT=9001 DPT=4322 WINDOW=0 RES=0x00 ACK SYN URGP=0
```

grok RegEx:

```
%{SYSLOGTIMESTAMP:time}\s*%{HOSTNAME:hostname}\s*%{SYSLOGPROG}:\s*\[%{SO:kernel_time_since_boot}\]\s*\[%{CHU:firewall_action}\]\s*IN=%{WORD1:int_in}\s*OUT=%{WORD1:int_out}\s*MAC=%{MAC:des_mac}:%{MAC:src_mac}:%{ETHTYPE:eth_type}\s*SRC=%{IP:src_ip}\s*DST=%{IP:des_ip}\s*LEN=%{BASE10NUM:length_of_payload}\s*TOS=%{WORD:type_of_service}\s*PREC=%{WORD:presedence}\s*TTL=%{BASE10NUM:time_to_live}\s*ID=%{BASE10NUM:identification}\s*%{WORD1}\s*PROTO=%{WORD:protocol}\s*SPT=%{BASE10NUM:src_port}\s*DPT=%{BASE10NUM:des_port}\s*(LEN=%{BASE10NUM:len})?\s*(WINDOW=%{BASE10NUM:window_size})?\s*(RES=%{WORD:res})?\s*%{WORD1:flag1}\s*%{WORD1:flag2}\s*(URGP=%{BASE10NUM:urgp})?
```

custom pattern:

```
SO \b[0-9.]+\b
CHU \b[a-zA-Z]+\s[a-zA-Z]+\b
WORD1 (\b\w+\b)?
ETHTYPE ([08:00]+|[86:dd]+|[86:DD]+)
```

Cách cấu hình thì có thể tham khảo tại [đây](https://github.com/nvtien996/thuctap062019/blob/master/Tiennv/Tim_hieu_ve_Wazuh/Filter_log_ssh.md)