title: linux-screen的使用
date: 2016/12/21 17:06:06
categories:
- JAVA服务端
tags:
- screen
- linux
---


# 什么是screen
简单来说，Screen是一个可以在多个进程之间多路复用一个物理终端的窗口管理器。操作的时候可以理解为像操作浏览器一样的。


# 安装screen  

```
yum install -y screen
```

# 使用screen-创建窗口

- **方式1：直接在命令行输入screen命令(建议使用，最好加上名字)**

```
[root@localhost ~]# screen
```

```
[root@localhost ~]# screen -S my_screen_name
```

> screen可以创建一个执行shell的窗口。你可以执行任意shell。在该窗口中使用exit退出该窗口，如果这是该screen会话的唯一窗口，该screen会话退出，否则screen自动切换到前一个窗口。

- **方式2：screen命令后跟你要执行的程序**
```
[root@localhost ~]# screen vi a.txt
```
> screen创建一个执行vi a.txt的单窗口会话，退出vi将退出该窗口/会话。

- **方式3：可以在一个已有screen会话中创建新的窗口。在当前screen窗口中键入c+a然后 c(或者再输入一次screen)，即Ctrl键+a键，之后再按下c键，screen 在该会话内生成一个新的窗口并切换到该窗口。**

# 使用screen-中断会话和回复会话
- **先创建一个screen窗口然后编辑vi a.txt**
- **假如有其他的事情需要退出，那么在screen窗口键入C-a d，Screen会给出detached提示，这样就中断了会话。**
- **如果需要恢复会话，使用screen -r 即可恢复。（如果是有多个screen，则加上窗口号使用screen -r 10038即可恢复）**

# 关于c-a绑定键

c-a 键 | 键的功能
---|---
c-a w | 显示所有窗口列表
c-a c-a | 切换到之前显示的窗口
c-a c | 创建一个新的运行shell的窗口并切换到该窗口
c-a n | 切换到下一个窗口
c-a p | 切换到前一个窗口(与C-a n相对)
c-a 0..9 | 切换到窗口0..9
c-a d | 暂时断开screen会话
c-a k | 杀掉当前窗口
c-a a | 发送 c-a到当前窗口
c-a [ | 进入拷贝/回滚模式

