## unrar解压
### 解决rar版本问题
```
[root@openresty-008 ~] eth0 = 10.132.132.94
# strings /usr/lib64/libstdc++.so.6 |grep GLIBC 
GLIBCXX_3.4
GLIBCXX_3.4.1
GLIBCXX_3.4.2
GLIBCXX_3.4.3
GLIBCXX_3.4.4
GLIBCXX_3.4.5
GLIBCXX_3.4.6
GLIBCXX_3.4.7
GLIBCXX_3.4.8
GLIBCXX_3.4.9
GLIBCXX_3.4.10
GLIBCXX_3.4.11
GLIBCXX_3.4.12
GLIBCXX_3.4.13
GLIBCXX_3.4.14
GLIBCXX_3.4.15
GLIBCXX_3.4.16
GLIBCXX_3.4.17
GLIBCXX_3.4.18
GLIBCXX_3.4.19
GLIBC_2.3
GLIBC_2.2.5
GLIBC_2.14
GLIBC_2.4
GLIBC_2.3.2
GLIBCXX_DEBUG_MESSAGE_LENGTH
```
可以看到，确实是缺少GLIBCXX_3.4.20和GLIBCXX_3.4.21，原因是当前版本libstdc++.so.6版本较低，替换新的版本即可。
### 下载对应的RPM包
rpm下载地址： 
[https://mirror.ghettoforge.org/distributions/gf/archive/el/7/gf/x86_64/](https://mirror.ghettoforge.org/distributions/gf/archive/el/7/gf/x86_64/)

```shell
cd /root
wget https://mirror.ghettoforge.org/distributions/gf/archive/el/7/gf/x86_64/gcc10-libstdc++-10.2.1-7.gf.el7.x86_64.rpm   
wget https://mirror.ghettoforge.org/distributions/gf/archive/el/7/gf/x86_64/gcc10-libstdc++-devel-10.2.1-7.gf.el7.x86_64.rpm  
```
### 安装rpm包
```shell
cd /root
rpm -ivh gcc10-libstdc++-10.2.1-7.gf.el7.x86_64.rpm  gcc10-libstdc++-devel-10.2.1-7.gf.el7.x86_64.rpm
```
查看安装目录在哪里？可以看到安装命令在/opt目录下
```shell
# rpm -ql gcc10-libstdc++-10.2.1
/opt/gcc-10.2.1/usr/lib64/libstdc++.so.6
/opt/gcc-10.2.1/usr/lib64/libstdc++.so.6.0.28
/opt/gcc-10.2.1/usr/share/gcc-10
/opt/gcc-10.2.1/usr/share/gcc-10/python
/opt/gcc-10.2.1/usr/share/gcc-10/python/libstdcxx
...
```
### 替换旧的libstdc库文件
```shell
cp /opt/gcc-10.2.1/usr/lib64/libstdc++.so.6.0.28 /usr/lib64/
cd /usr/lib64/  && unlink  libstdc++.so.6
cd /usr/lib64/  && ln -sv libstdc++.so.6.0.28 libstdc++.so.6

```
### 测试库文件是否成功

```shell
strings /usr/lib64/libstdc++.so.6 | grep GLIBC
# strings /usr/lib64/libstdc++.so.6 | grep GLIBC
GLIBCXX_3.4
GLIBCXX_3.4.1
GLIBCXX_3.4.2
GLIBCXX_3.4.3
GLIBCXX_3.4.4
GLIBCXX_3.4.5
GLIBCXX_3.4.6
GLIBCXX_3.4.7
GLIBCXX_3.4.8
GLIBCXX_3.4.9
GLIBCXX_3.4.10
GLIBCXX_3.4.11
GLIBCXX_3.4.12
GLIBCXX_3.4.13
GLIBCXX_3.4.14
GLIBCXX_3.4.15
GLIBCXX_3.4.16
GLIBCXX_3.4.17
GLIBCXX_3.4.18
GLIBCXX_3.4.19
GLIBCXX_3.4.20
GLIBCXX_3.4.21
GLIBCXX_3.4.22
GLIBCXX_3.4.23
GLIBCXX_3.4.24
GLIBCXX_3.4.25
GLIBCXX_3.4.26
GLIBCXX_3.4.27
GLIBCXX_3.4.28
```
可以看到，GLIBCXX_3.4.20和GLIBCXX_3.4.21已经存在了。

### 测试rar命令
如下执行正常，问题解决 
```shell
# rar

RAR 6.20 beta 2   Copyright (c) 1993-2022 Alexander Roshal   10 Dec 2022
Trial version             Type 'rar -?' for help

Usage:     rar <command> -<switch 1> -<switch N> <archive> <files...>
               <@listfiles...> <path_to_extract/>

<Commands>
  a             Add files to archive
  c             Add archive comment
  ch            Change archive parameters
  cw            Write archive comment to file
  d             Delete files from archive
  e             Extract files without archived paths
  f             Freshen files in archive
  i[par]=<str>  Find string in archives
  k             Lock archive
......

## unrar解压

```shell
- e             # 解压压缩文件到当前目录
- l[t,b]        # 列出压缩文件[技术信息,简洁]
- p             # 将文件打印到标准输出。
- t             # 测试压缩文件
- v[t,b]        # 详细列出压缩文件[技术信息,简洁]
- x             # 用绝对路径解压文件
```

```shell
-av-       # 禁用，真实性验证检查。
-c-        # 禁用，评论显示
-f         # 刷新文件
-kb        # 保留破碎的提取文件
-ierr      # 将所有消息发送给stderr。
-inul      # 禁用，所有消息。
-o+        # 覆盖现有文件。
-o-        # 不要覆盖现有文件
-p<password>
     	    # 设置密码。
-p-        # 不查询密码
-r         # 递归子目录。
-u         # 更新文件。
-v         # 列出所有卷。
-x<file>
     	    # 排除指定的文件。
-x@<list>
     	    # 排除指定列表文件中的文件。
-x@        # 读取要从 stdin 中排除的文件名。
-y         # 对所有查询都假设为是。
```

## unzip解压
- -c 将解压缩的结果显示到屏幕上，并对字符做适当的转换。
- -f 更新现有的文件。
- -l 显示压缩文件内所包含的文件。
- -p 与-c参数类似，会将解压缩的结果显示到屏幕上，但不会执行任何的转换。
- -t 检查压缩文件是否损坏。
- -u 与-f参数类似，但是除了更新现有的文件外，也会将压缩文件中的其他文件解压缩到目录中。
- -v 执行是时显示详细的信息。
- -z 仅显示压缩文件的备注文字。
- -a 对文本文件进行必要的字符转换。
- -b 不要对文本文件进行字符转换。
- -C 压缩文件中的文件名称区分大小写。
- -j 不处理压缩文件中原有的目录路径。
- -L 将压缩文件中的全部文件名改为小写。
- -n 解压缩时不要覆盖原有的文件。
- -o 不必先询问用户，unzip执行后覆盖原有文件。
- -P<密码> 使用zip的密码选项。
- -q 执行时不显示任何信息。
- -s 将文件名中的空白字符转换为底线字符。
- -d <目录> 指定文件解压缩后所要存储的目录。
- -x <文件> 指定不要处理.zip压缩文件中的哪些文件

### unzip解压 
```
unzip -o -d test/ test01.zip 

```





