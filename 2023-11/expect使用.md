# Expect介绍
- 在linux运维和开发中，我们经常需要远程登录服务器进行操作,登录的过程是一个交互的过程，可能会需要输入yes/no，password等信息。为了模拟这种输入，可以使用Expect脚本
- expect 是基于tcl 演变而来的，所以很多语法和tcl 类似，Expect 是用来进行自动化控制和测试的工具。主要解决shelI脚本中不可交互的问题。对于大规模的linux运维很有帮助

简单来说，expect可以用于日常固定的交互命令中，减少部分重复的工作（重复输入密码等）

以及可以通过脚本和命令进行交互，提供自动化的基本能力

# Expect使用

## 安装

```bash
brew install expect # mac安装
```

## Examples

**send**：向进程发送字符串，用于模拟用户的输入， 该命令不能自动回车换行，一般要加\r（回车）

**expect**： expect的一个内部命令，判断上次输出结果里是否包含指定的字符串，如果有则立即返回，否则就等待超时时间后返回，只能捕捉由spawn启动的进程的输出expect

**spawn**：启动进程，并跟踪后续交互信息

**interact**：执行完成后保存交互状态，把控制权交给控制台

- 单位是：秒
- timeout -1 为永不超时 
- 默认情况下，timeout是10秒

`set timeout 30`：设置超时时间为30秒(默认的超时时间是 10 秒，通过 set 命令可以设置会话超时时间, 若不限制超时时间则应设置为-1)
**exp_continue**： 允许expect继续向下执行指令meout：指定超时时间，过期则继续执行后续指令

**send_user**： 回显命令，相当于echo

$argv参数数组：Expect脚本可以接受从bash传递的参数，可以使用 [lindex $argv n] 获得，n从0开始，分别表示第一个$1，第二个$2，第三个$3……参数 ($argvn没有空格则表示脚本名称 ； $argv n有空格则代表下标)


Expect脚本必须以interact或expect eof 结束，执行自动化任务通常expect eof就够了

### Example 1
sudo免输入密码

1. 创建test.sh
```bash
#!/usr/bin/expect

spawn sudo gem install appium_console
expect "Password:*" { send "123456" }
expect EOF
```

2. 执行
```bash
expect test.sh
```

### Example 2
Refer: 
- https://phoenixnap.com/kb/linux-expect
- https://blog.csdn.net/lixinkuan328/article/details/111991344




