## 一、节点环境介绍
**服务器**：2台腾讯云服务器，一台百度云服务器
**操作系统**： Ubuntu 16.04.4 LTS
**Hadoop版本**：hadoop-2.7.6
**Java版本**：1.8.0_191

## 二、前期准备

 

 1. **为了更好的在Shell中区分三台主机，修改其显示的主机名，执行如下命令**

```
sudo vim /etc/hostname
```
在master的/etc/hostname中添加如下配置：

```
master
```
slave1的/etc/hostname中添加如下配置：

```
slave1
```
同理slave2
```
slave2
```
重启三台电脑，发现机器名已经变化
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190224145521794.png)
 2. 在三台机器的/etc/hosts文件中,添加如下配置

```
ip1 master
ip2 slave1
ip3 slave2
```
注意：如果是在master上操作的话ip1 必须是master 的内网IP 同理slaves上也是一样，其他的要用外网IP。

查看内网地址

```
ifconfig
```
![](https://img-blog.csdnimg.cn/20190224151054741.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg2MDgwMA==,size_16,color_FFFFFF,t_70)

 3. 建立Hadoop运行账号
	

	```
	sudo adduser hadoop
	```
	按提示输入密码后赋予hadoop用户管理权限
	

	```
	sudo adduser hadoop sudo
	```
	切换至hadoop用户
	

	```
	su hadoop
	```


 4. 配置ssh免密登陆
	开始配置ssh之前，先确保三台机器都装了ssh。输入以下命令测试能否连接到本地，验证是否安装ssh。 
	
	```
	ssh localhost
	```
	若不能，则安装open-server

	```
	sudo apt-get openssh-server
	```
	并生成ssh公钥。
	
	```
	ssh-keygen -t rsa -P ""
	```
	将公钥加入到已认证的key中

	```
	cat $HOME/.ssh/id_rsa.pub >> $HOME/.ssh/authorized_keys
	```
	在保证了三台主机电脑都能连接到本地localhost后，还需要让master主机免密码登录slave1和slave2主机。在master执行如下命令，将master的id_rsa.pub传送给两台slave主机。
	

	```
	scp ~/.ssh/id_rsa.pub hadoop@slave1:/home/hadoop/
	scp ~/.ssh/id_rsa.pub hadoop@slave2:/home/hadoop/
	```
	接着在slave1、slave2主机上将master的公钥加入各自的节点上,在slave1和slave2执行如下命令:
	

	```
	cat ~/id_rsa.pub >> ~/.ssh/authorized_keys
    rm ~/id_rsa.pub
	```
	在master主机上直接执行如下测试命令，即可让master主机免密码登录slave1、slave2主机。
	

	```
	ssh slave1
	ssh slave2
	```

## 三、JDK和Hadoop安装配置
	

 1. 安装jdk
 	执行安装jdk的操作
	```
	sudo apt-get install default-jdk
	```
	编辑~/.bashrc文件，添加如下内容：
	

	```
	export JAVA_HOME=/usr/lib/jvm/default-java
	```
	接着让环境变量生效，执行如下代码

	```
	source ~/.bashrc
	```

 2. 安装Hadoop，在master主机执行如下操作
	安装Hadoop的压缩包(可进入[http://mirrors.hust.edu.cn/apache/hadoop/common/]查看版本）
	 
	```
	sudo wget http://mirrors.hust.edu.cn/apache/hadoop/common/hadoop-2.7.6/hadoop-2.7.6.tar.gz
	```
	

	```
	sudo tar -zxf ~/hadoop-2.7.6.tar.gz -C /usr/local    # 解压到/usr/local中
    cd /usr/local/
    sudo mv ./hadoop-2.7.6/ ./hadoop            # 将文件夹名改为hadoop
    sudo chown -R hadoop ./hadoop       # 修改文件权限
	```
	编辑~/.bashrc文件，添加如下内容：
	

	```
	export HADOOP_HOME=/usr/local/hadoop
	export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
	```
	接着让环境变量生效，执行如下代码：

	```
	source ~/.bashrc
	```

3. Hadoop集群配置
	修改slaves文件（这些配置文件都位于/usr/local/hadoop/etc/hadoop目录下），加入如下内容，让master节点主机仅作为NameNode使用，删除其中的localhost：

	```
	slave1
	slave2
	```

	修改core-site.xml
	
	```
	  <configuration>
	      <property>
	          <name>hadoop.tmp.dir</name>
	          <value>file:/usr/local/hadoop/tmp</value>
	          <description>Abase for other temporary directories.</description>
	      </property>
	      <property>
	          <name>fs.defaultFS</name>
	          <value>hdfs://master:9000</value>
	      </property>
	  </configuration>
	```

修改hadoop-env.sh，加入Java的安装地址
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190225100112911.png)

	
修改hdfs-site.xml：
	

	```
		<configuration>
	  		<property>
	    		<name>dfs.replication</name>
	        	<value>3</value>
	  		</property>
		</configuration>
	```
修改mapred-site.xml(复制mapred-site.xml.template,再修改文件名)
	

	```
	sudo mv ./mapred-site.xml.template ./mapred-site.xml
	```

	```
	<configuration>
		<property>
        	<name>mapreduce.framework.name</name>
        	<value>yarn</value>
		</property>
	</configuration>
	```

修改yarn-site.xml
	

```
 <configuration>
  <!-- Site specific YARN configuration properties -->
      <property>
          <name>yarn.nodemanager.aux-services</name>
          <value>mapreduce_shuffle</value>
      </property>
      <property>
          <name>yarn.resourcemanager.hostname</name>
          <value>master</value>
      </property>
  </configuration>
```
	
配置好后，将 master 上的 /usr/local/Hadoop 文件夹复制到各个节点上。速度较慢，也可直接在slave上重新配置

```
cd /usr/local/
tar -zcf ~/hadoop.master.tar.gz ./hadoop
cd ~
scp ./hadoop.master.tar.gz slave1:/home/hadoop
scp ./hadoop.master.tar.gz slave2:/home/hadoop
```
在slave1,slave2节点上执行：

```
sudo tar -zxf ~/hadoop.master.tar.gz -C /usr/local
sudo chown -R hadoop /usr/local/hadoop
```

4. 启动Hadoop集群

在master主机上执行如下命令：

```
cd /usr/local/hadoop
bin/hdfs namenode -format
sbin/start-all.sh
```
运行后，在master，slave1,slave2运行jps命令，查看：

```
jps
```
看到如下相似内容
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190225100312918.png)
搭建完成~


参考文章：
1.http://dblab.xmu.edu.cn/blog/1177-2/#more-1177
2.https://blog.csdn.net/gakki_smile/article/details/77198146
3.https://www.cnblogs.com/caiyisen/p/7373512.html
