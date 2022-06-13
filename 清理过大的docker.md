```bash
#!/usr/bin/env bash
export CONTAINER="/var/lib/docker/containers"
export OUTFILE="/opt/docker.txt"
export BIGFILE="/opt/bigfile.txt"
cd $CONTAINER
du -sm *|sort -n|uniq -c > $OUTFILE

cat $OUTFILE|awk '$2>5000{print $0}' > $BIGFILE


function clear_docker()
{
	for i in `cat $BIGFILE|awk '{print $NF}'|awk '{print substr($1,1,4)}'`
	do
		for j in `docker inspect $i|grep -i data|grep -i source|awk '{print $2}'| sed 's/\"//g'| sed 's/\,//g'|sed 's/\logs//g'`
			do
				cd $j;docker-compose down;docker-compose up -d
			done
				
	done
	echo "have finish clear docker big file"
}


function main(){
        clear_docker
}

main

```

