# Linux常用命令:

1.查看端口：

lsof -i:3306     查看3306这个端口的使用情况

2.强制杀死某个端口的进程：

kill -9 进程id      

3.开启/关闭mysql服务

```linux
service mysqld start        service mysqld stop
```

## 1.文件相关的命令

``` linux
1.新建文件
touch file.txt      --新建文件
touch file{1..5}.txt    --批量创建文件t
2.查看较小文件的文件内容
cat -n file.txt       --查看文件内容并显示行号
cat -n file.log > mantou.log     --查看文件内容，添加行号后输出到另一个文件中
```

