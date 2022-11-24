```bash
$ for i in `cat source.txt`
> do
> if wget --spider $i 2>/dev/null;then echo "$i exists" 
> else
> echo "$i not exist"
> fi
> done
```

