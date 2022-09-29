`

```shell
#!/bin/sh
# 获取当前分支
line=$(head -n +1 .git/HEAD)
branch=${line##*/}

commit="${branch} $(cat $1)"
echo "$commit" > "$1"
```

`