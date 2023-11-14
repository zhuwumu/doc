# bat命令

## 执行sql

```bash
@echo off

set command=use test;
set sql=^
update sys_role set role_name='zhangsan' where id = 1;

set sqlCommand=%command%%sql%
echo %sqlCommand%

mysql -uroot -proot -e "%sqlCommand%"

pause
```



## 启动jar

```bash
echo

start "springboot3" java -jar springboot3-0.0.1-SNAPSHOT.jar
```

## 停止jar

```bash
@echo off
set port=8080
for /f "tokens=1-5" %%i in ('netstat -ano^|findstr ":%port%"') do (
echo kill the process %%m who use the port
taskkill /pid %%m -t -f
goto start
)
:start
exit
```

> 结束端口 netstat -aon|findstr "8080"   taskkill /pid 4136 -t -f 

