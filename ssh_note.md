**将公钥写入远程主机（三种方法）**<br>
***ssh免密登录，vscode_ssh，ssh-copy-id***

&emsp;&emsp;对于vscode_ssh连接开发，可能都会出现一些奇奇怪怪的问题，但只要依据报错出现的 `log` 基本上都能够解决。<br>
&emsp;&emsp;写笔记时遇到过的：XHR failed；~\.ssh\config文件权限问题；远程主机ssh的TCP端口转发。<br>
&emsp;&emsp;具体解决步骤不记录，太冗杂，根据 `log` 解决即可。


***
**方法一**：

&emsp;&emsp;Windows powshell使用type命令：

```bash
type $env:USERPROFILE\.ssh\id_rsa.pub | ssh <ip & domain> "cat >> .ssh/authorized_keys"
```
&emsp;&emsp;example（没有authorized_keys文件）:
```mySQL
type $env:USERPROFILE\.ssh\id_rsa.pub | ssh root@10.10.1.4 "mkdir .ssh;touch .ssh/authorized_keys;cat >> .ssh/authorized_keys"
```
&emsp;&emsp;example（有authorized_keys文件）:
```
type $env:USERPROFILE\.ssh\id_rsa.pub | ssh root@10.10.1.4 "cat >> .ssh/authorized_keys"
```

&emsp;&emsp;$env：这是一个Windows PowerShell中用于访问环境变量的特殊变量。

&emsp;&emsp;authorized_keys 是一个用于SSH（Secure Shell）身份验证的文件，存储了被信任的公钥列表（该验证文件也可以命名为其它名称，前提是修改ssh_config文件的相关信息）。

<br>

**方法二**：

&emsp;&emsp;直接复制粘贴：

&emsp;&emsp;创建一个authorized_keys文件在 ~/.ssh/ 目录下（如果没有的话），再通过复制粘贴将公钥写入到文件内；或者把公钥文件复制到远程主机通过追加（>>）的形式写入到文档。

<br>

**方法三**：

&emsp;&emsp;在远程主机上执行命令：

&emsp;&emsp;将私钥和公钥一同复制到远程主机，使用 ssh-copy-id 命令会自动生成authorized_keys秘钥验证文件，如果有的话则会继续向后追加内容。

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub username@remote_host
```

&emsp;&emsp;参数 -i 指定要使用的公钥文件；后面是当前用户及本机ip或域名。

&emsp;&emsp;在进行写入时，私钥也会同时被调用，以验证公钥的完整性和正确性，没有或者不匹配的话则无法继续写入（局限性）。

<br>

***
***
&emsp;&emsp;对于本机需要配置验证私钥文件的内容（密钥在非默认位置时需要）：
配置文件： `~\.ssh\config`，的 `IdentityFile`

```bash
Host 192.168.10.10	# 别名&备注；连接时，在终端使用'ssh 别名'即可无密码快速登录远程主机。
  HostName 192.168.10.10	# 主机地址或域名
  User deng		# 用户名
  IdentityFile C:\Users\a3071\.ssh\key_rsa	# 需要验证的私钥文件
```

&emsp;&emsp;对于远程主机需要支持公钥配对的配置：
远程服务器的 SSH 配置文件（通常位于`/etc/ssh/sshd_config`）中，确保以下两个配置项未被注释:
```bash
PubkeyAuthentication yes
AuthorizedKeysFile     .ssh/authorized_keys
```
&emsp;&emsp;windows的配置文件（与本次文章主题无关）：`C:\Windows\System32\OpenSSH\sshd_config_default`<br>

<br>
注释：

&emsp;&emsp;Powershell 不支持ssh-copy-id命令，衍生出使用type命令的操作方法。

&emsp;&emsp;ssh-copy-id 局限于需要同时拥有密钥对。

&emsp;&emsp;其它（点睛）~
&emsp;&emsp;生成密钥对：
```bash
ssh-keygen -t rsa -b 2048 -f ~/.ssh/id_rsa
```

```bash
-t rsa ： 指定要生成的密钥类型为RSA，常用的密钥类型就是RSA密钥。（type）
-b 2048 ： 指定密钥的位数为2048位，这是一种常用的安全强度。（bits）
-f ~/.ssh/id_rsa ： 指定要保存私钥的文件路径和文件名；不指定位置，密钥会存储到当前目录下。（output_keyfile）  
```

&emsp;&emsp;其它常用参数：

```bash
-N （passphrase）：设置私钥的密码短语。私钥密码短语提供了一层额外的保护，用于加密私钥文件
-E （fingerprint_hash）：指定指纹哈希算法的类型。默认为SHA-256，但可以选择其他算法，如SHA-1和MD5。
-C （comment）：为生成的密钥添加一个注释，用于标识该密钥的用途或所有者。
```
&emsp;&emsp;不指定参数，只生成密钥对文件的话，一切以默认参数值为主。
