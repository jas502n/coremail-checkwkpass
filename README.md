

## 检查邮件系统弱密码用户

```
如果密码是 {enc1}xxxx，crypt('密码', 'xxxx')
如果密码是 {enc2}xxxx，md5sum('密码')，可能会再用base64编码
如果密码是 {enc5}xxxx，crypt('密码', 'xxxx')，使用带salt MD5，非常慢
如果密码是 {enc7}xxxx，把xxxx decode64转成2进制，密码+最后8位 SHA1 然后校验
如果密码是 {enc8}xxxx，把xxxx decode64转成2进制，密码+最后4位 MD5 然后校验
```

### example enc5
![image](https://user-images.githubusercontent.com/16593068/215650849-6ad458ba-820c-40e5-88fe-832a62cbdb2a.png)


![image](https://user-images.githubusercontent.com/16593068/212266899-1bf598db-93a8-49bf-beb7-19074177afda.png)


## 附录：

从邮件系统导出邮件地址和加密后密码的sql语句：

```
select concat(a.user_id,'@',b.domain_name), c.password from td_user a, td_domain b, cm_user_info c where a.domain_id=b.domain_id and a.org_id=c.org_id and a.user_id=c.user_id order by b.domain_name, a.user_id;
```

```
{enc1}qsQ.VmFRz1wos
{enc2}e10adc3949ba59abbe56e057f20f883e
{enc3}9Pf28fDz
{enc4}32ed87bdb5fdc5e9cba88547376818d4
{enc5}$1$63beebab$4caiJdl/5RDWvtV4grmmI1
{enc5}$1$63bef1fe$GKajrw/aG4Zye3rsRUenb.
{enc6}fEqNCco3Yq9h5ZUglD3CZJT4lBs=
{enc7}GmqksTiMqHvY7JeYhwdeSHnoer1tGYN2qqaGSw==
{enc8}m7VYbdwZDfdbykmc0t/PxBFuiFQ=
{enc9}Y49b5VrUMuxObEtsl3eD4w==
{enc10}123456eEZkSVFla2ZCYXRydw==
{enc11}d8913df37b24c97f28f840114d05bd110dbb2e44
{enc12}4280d89a5a03f812751f504cc10ee8a5
{enc13}$5$rounds=362$dna11kh7svsdbh$APv9OIDeLigfM5mE3RYlr0oH/FmdxnCfmYfeBqdm9o8
{enc15}6b2049838378330f
{enc16}123456
```
![image](https://user-images.githubusercontent.com/16593068/212266463-7688ded5-4fe2-4cd9-9d7d-71595f6489b8.png)


把上述sql存为文件 exportpass.sql，执行
```
/home/coremail/bin/mysql_cm --skip-column-names cmxt < exportpass.sql > email.pass.txt
```

![image](https://user-images.githubusercontent.com/16593068/211709925-637fe306-5b27-4ee5-9765-e1db64b11178.png)


感谢清华大学 马老师 提供弱密码库。

思路：

1. 从邮件系统导出如下数据：
```
邮件地址 加密后的密码
```
假定导出的文件是email.pass.txt。email.pass.txt每行也可以仅仅有 加密后的密码，这样可以用来找出系统的弱密码。


2. 使用常见的弱密码库进行碰撞

测试运行
```
mpirun --allow-run-as-root -n 2 ./checkwkpass -w wk_pass.txt -p testcase.txt
```

正常运行
```
mpirun -n X ./checkwkpass -w wk_pass.txt -p email.pass.txt
```
X是进程数，X >= 2 (原因是主进程负责分发计算任务，自己不计算)

其中 wk_pass.txt 是弱密码文件，一行一个密码

如果使用enc2 md5加密方式，碰撞速度很快，读入弱密码文件（大约需要1-4秒钟）之后，每秒钟可以处理 4万 以上用户的碰撞。

如果使用enc1 或 enc8 加密方式，则比较慢。不同的CPU速度差别也比较大，新的服务器明显要快。

使用10年前的服务器，碰撞一次大致花费时间是：
```
enc1 5秒钟
enc2 0秒钟
enc8 1秒钟
```


即可导出
