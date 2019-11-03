O10_客户端管理工具em

输入地址访问：https://192.168.206.131:5500/em/login



```shell
[oracle@Oracle01 ~]$ ss -luntp | grep tnslsnr
tcp    LISTEN     0      128      :::1521                 :::*                   users:(("tnslsnr",pid=7511,fd=8))
tcp    LISTEN     0      128      :::1522                 :::*                   users:(("tnslsnr",pid=7601,fd=8))



SQL> exec DBMS_XDB.setHTTPPort(5500);

PL/SQL procedure successfully completed.


[oracle@Oracle01 ~]$ ss -luntp | grep tnslsnr
tcp    LISTEN     0      128      :::1521                 :::*                   users:(("tnslsnr",pid=7511,fd=8))
tcp    LISTEN     0      128      :::1522                 :::*                   users:(("tnslsnr",pid=7601,fd=8))
tcp    LISTEN     0      128      :::5500                 :::*                   users:(("tnslsnr",pid=7511,fd=16))

```

