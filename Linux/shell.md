

### shell 运行方式

https://feihu.me/blog/2014/env-problem-when-ssh-executing-command-on-remote/



#### login 模式：

首先读取并执行/etc/profile文件；

然后读取并执行~/.bash_profile文件；

~/.bash_profile会显式调用~/.bashrc文件，而~/.bashrc又会显式调用/etc/bashrc文件，/etc/bashrc 会调用 /etc/profile.d/* 配置的环境变量，这是为了让所有交互式界面看起来一样。无论你是从远程登录（登陆shell），还是从图形界面打开终端（非登陆shell），你都拥有相同的提示符，因为环境变量在/etc/bashrc文件中被统一设置过。

如果你想对bash的功能进行设置或者是定义一些别名，推荐你修改~/.bashrc文件，这样无论你以何种方式打开shell，你的配置都会生效。而如果你要更改一些环境变量，推荐你修改~/.bash_profile文件，因为考虑到shell的继承特性，这些更改确实只应该被执行一次（而不是多次）。针对所有用户进行全局设置，推荐你在/etc/profile.d目录下添加以.sh结尾的文件，而不是去修改全局startup文件。

/etc/profile.d/java.sh  添加 java 的全局环境变量。



不同模式的配置文件加载：

```sql
+----------------+--------+-----------+---------------+
|                | login  |interactive|non-interactive|
|                |        |non-login  |non-login      |
+----------------+--------+-----------+---------------+
|/etc/profile    |   A    |           |               |
+----------------+--------+-----------+---------------+
|/etc/bashrc|             |    A      |               |
+----------------+--------+-----------+---------------+
|~/.bashrc       |        |    B      |               |
+----------------+--------+-----------+---------------+
|~/.bash_profile |   B1   |           |               |
+----------------+--------+-----------+---------------+
|~/.bash_login   |   B2   |           |               |
+----------------+--------+-----------+---------------+
|~/.profile      |   B3   |           |               |
+----------------+--------+-----------+---------------+
|BASH_ENV        |        |           |       A       |
+----------------+--------+-----------+---------------+
```



当bash以是sh命启动时，即我们此处的情况，bash会尽可能的模仿sh，所以配置文件的加载变成了下面这样：

- interactive + login: 读取/etc/profile和~/.profile
- non-interactive + login: 同上
- interactive + non-login: 读取ENV环境变量对应的文件
- non-interactive + non-login: 不读取任何文件