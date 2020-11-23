1、prepare web's site lists
```bash
# cat domain.txt 
www.baidu.com
www.qq.com
```


2、execute the shell script as follows:
```bash
for i in `cat domain.txt`; \
    do echo -e "\n------------------------$i";\
    openssl s_client -connect $i:443 < /dev/null 2>& 1|openssl x509 -noout -dates;\
done
```

run result:
```bash
# sh decide_web_cert.sh 

------------------------www.baidu.com
notBefore=Apr  2 07:04:58 2020 GMT
notAfter=Jul 26 05:31:02 2021 GMT

------------------------www.qq.com
notBefore=Jun 22 00:00:00 2020 GMT
notAfter=Sep 22 12:00:00 2021 GMT
```

