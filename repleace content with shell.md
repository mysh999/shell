```bash
for i in `grep -i "ssl_session_timeout" -rn .|awk '{print $1}'|awk -F ':' '{print $1}'|awk -F '/' '{print $NF}'` 
do
sed -i 's/ssl_session_timeout/#&/g' $i
done
```

说明：

将遍历的文件中包含ssl_session_timeout内容的行首加上#，&是匹配任意字符



如果是

sed -i 's/^ssl_session_timeout/#&/g' $i，说明是以ssl_session_timeout为行首的（无空格）

