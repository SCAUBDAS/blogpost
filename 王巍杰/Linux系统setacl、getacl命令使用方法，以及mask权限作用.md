setfacl 命令设置ACL权限。getfacl 命令用于显示文件上设置的 ACL 信息。
ACL（ Access Control List (访问控制列表)）提供的是在所有者、所属组、其他人的读/写/执行权限之外的特殊权限控制。通俗来讲，基于普通文件或目录设置 ACL 其实就是针对指定的用户或用户组设置文件或目录的操作权限。
***
getfacl命令格式 

```
getfacl [参数] [目标文件名]
```
一般用到的参数有以下几种
参数|作用
-|:--:
a | 仅显示文件acl
d | 仅显示默认的acl
R | 递归显示子目录acl
***

setfacl命令格式
```
setfacl [参数] [ugm]:[用户名]:[rwx] [目标文件名]
```
第二栏参数，u代表用户，g代表群组，而使用m参数可修改最大有效访问权限
***
setfacl参数
参数 | 作用
--|:--:
 m | 设置acl参数，不可与-x合用
 x | 删除acl参数
b | 删除全部的acl参数
R | 针对目录设置acl权限，其子目录会继承其权限，需要与m、x等结合使用
d | 为目录添加默认acl，需要与m结合使用
k | 删除默认的acl参数

***
 下面对一些常用命令进行展示
 
 用于测试的用户：root、wang
 用于测试的样例文件关系
 + ~
   + testsetfacl
     + test
     + testfacl.txt
***
获取testsetfacl的acl权限
```
[root@master ~]# getfacl testsetfacl
# file: testsetfacl        文件名
# owner: root              文件所有者，ls -l 第三列信息
# group: root              文件所属群组， ls -l 第四列信息
user::rwx                  所有者权限
group::r-x                 所属群组权限
other::r-x	               其他用户权限
```
查看用户wang的权限
```
[root@master ~]# su wang
[wang@master root]$ cd testsetfacl  可以进入testsetfacl文件
[wang@master testsetfacl]$ exit
```
修改wang用户的权限为0
```
[root@master ~]# setfacl -Rm u:wang:--- testsetfacl
[root@master ~]# getfacl -R testsetfacl
# file: testsetfacl
# owner: root
# group: root
user::rwx
user:wang:---                       #增加了此行，权限为0
group::r-x
mask::r-x
other::r-x

# file: testsetfacl/test
# owner: root
# group: root
user::rwx
user:wang:---
group::r-x
mask::r-x
other::r-x

# file: testsetfacl/testsetfacl.txt
# owner: root
# group: root
user::rw-
user:wang:---
group::r--
mask::r--
other::r--

```
切换到wang用户，执行cd命令，wang用户已经不能进入testsetfacl目录
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190408233454366.png)
***
尝试修改预设acl

```
[root@master ~]# setfacl -dm u:wang:rwx testsetfacl
[root@master ~]# getfacl -R testsetfacl
# file: testsetfacl
# owner: root
# group: root
user::rwx
user:wang:---
group::r-x
mask::r-x
other::r-x
default:user::rwx             #testsetfacl增加了五行default
default:user:wang:rwx         #即预设acl，在其下创建子目录
default:group::r-x            #会继承预设acl，
default:mask::rwx
default:other::r-x

# file: testsetfacl/test
# owner: root
# group: root
user::rwx
user:wang:---
group::r-x
mask::r-x
other::r-x

# file: testsetfacl/testsetfacl.txt
# owner: root
# group: root
user::rw-
user:wang:---
group::r--
mask::r--
other::r--


```

测试目录文件继承效果
```
[root@master ~]# cd testsetfacl
[root@master testsetfacl]# mkdir testd
[root@master testsetfacl]# getfacl testd
# file: testd
# owner: root
# group: root
user::rwx
user:wang:rwx
group::r-x
mask::rwx
other::r-x
default:user::rwx
default:user:wang:rwx
default:group::r-x
default:mask::rwx
default:other::r-x
#创建的目录完全继承了testsetfacl的预设acl
#注意，若文件夹或文件在增加预设acl前就已经创建，则其不会继承
#预设acl，其子文件及目录也不会继承。
```
测试一般文件继承效果

```
[root@master testsetfacl]# touch test.txt
[root@master testsetfacl]# getfacl test.txt
# file: test.txt
# owner: root
# group: root
user::rw-
user:wang:rwx                   #effective:r--
group::r-x                      #effective:r--
mask::r--
other::r--
#一般文件则只继承群组和特定用户acl，其他权限则和
#没有acl的情况下创建文件的权限一样
```

从以上命令结果可以看到增加了effective一栏信息，实际上是与mask一行权限有关。mask一行即为最大有效访问权限，作用是控制文件的访问权限，其他用户或组设定ACL实际权限都是与mask最大有效权限相与的结果。test.txt 文件的mask权限为 ：r-- 。group和指定用户wang 的权限虽为 r-x、rwx，但与 r-- 相与运算后 ，均为 r--。只能读取test.txt。

修改默认mask权限可通过以下命令。

```
setfacl -Rm m:---  testsetfacl
```

