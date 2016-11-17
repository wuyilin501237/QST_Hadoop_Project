# Round 1 介绍
方案：
  首先搭建Hadoop-2.6.5环境
  一、安装java环境
     1.下载jdk-6u45-linux-i586.bin文件
     2.通过Xshell或sercuecrt将jdk-6u45-linux-i586.bin文件上传到linux
     3.给文件执行权限，输入chomd +x ./jdk-6u45-linux-i586.bin
     4.输入./jdk-6u45-linux-i586.bin安装
     5.配置java环境
        vim /home/hadoop/.bashrc
        在文件最后一行加入：export JAVA_HOME=/home/hadoop/jdk1.6.0_45/
                          export CLASSPATH=.:$CLASSPATH:$JAVA_HOME/lib:$JRE_HOME/lib
                          export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
     6.输入source ~/.bashrc启动
     7.测试java配置是否成功，输入java -version
  二、安装Hadoop
    1.下载hadoop-2.6.5.bin.tar.gz文件
    2.通过Xshell或sercuert上传hadoop-2.6.5.bin.tar.gz文件
    3.解压缩 tar zxvf hadoop-2.6.5.bin.tar.gz
    4.在解压后的目录下新增tmp文件
        mkdir tmp   用来指定Hadoop运行时产生文件的目录
    5.配置Hadoop
        使用vim修改slaves文件。选择搭建一个master两个slaves
        使用vim修改core-site.xml
        <configuration>
	      <property>
                <name>fs.defaultFS</name>
                <value>hdfs://vm10-0-0-2.ksc.com:9000</value>
        </property>
        <property>
                <name>hadoop.tmp.dir</name>
                <value>file:/home/hadoop/hadoop-2.6.5/tmp</value>
                <description>Abase for other temporary directories.</description>
        </property>
        </configuration>
        修改hdfs-site.xml
        <configuration>
	      <property>
                <name>dfs.namenode.secondary.http-address</name>
                <value>vm10-0-0-2.ksc.com:50090</value>
        </property>
        <property>
                <name>dfs.replication</name>
                <value>3</value>
        </property>
        <property>
                <name>dfs.namenode.name.dir</name>
                <value>file:/home/hadoop/hadoop-2.6.5/tmp/dfs/name</value>
        </property>
        <property>
                <name>dfs.datanode.data.dir</name>
                <value>file:/home/hadoop/hadoop-2.6.5/tmp/dfs/data</value>
        </property>
        </configuration>
        其中，<value>3</value>视情况修改，如果有三台slave机器，这里设置成3，如果只有1台或2台，修改成对应的值即可。换句话说，就是HDFS保存的副本的数         量。
        修改hadoop-env.sh。 新增export JAVA_HOME=/home/hadoop/jdk1.8.0_111
        修改yarn-site.xml
        第一个属性：Yarn的老大是resourcemanager的地址（master）
        第二个属性：告诉nodemanager获取数据的方式是shuffle方式
        修改本地网络配置
        使用vim编辑/etc/hosts文件
        sudo 192.168.2.182 master
             192.168.2.178 slaves1
             192.168.2.230 slaves2
  三、建立互信关系
      1.生成公私钥：在master机器的虚拟机命令行下输入ssh-keygen 
      2.复制公钥：复制一份master的公钥文件：cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
        同样，在所有的slave机器上，也在命令行中输入：ssh-keygen 
        所有的salve机器上，从master机器上复制master的公钥文件：scp master:~/.ssh/authorized_keys      /home/hadoop/.ssh/
      3.测试连接
      在master机器上分别向所有的slave机器发起联接请求：ssh slave1
  四、启动Hadoop
      1.初始化
         在master机器上，进入/home/hadoop/hadoop-2.6.5/bin目录，运行 ./hadoop namenode –format 或者 hdfs namenode -format
         初始化hadoop的文件系统
      2.启动Hadoop
         执行./start-all.sh，中间过程 ，可能提示要判断是否,输入yes，输入jps，查看进程是否都正常启动
      3.测试系统
      输入./hadoop fs –ls /查看文件
 完成UV统计
     UV：指通过互联网访问、浏览这个网页的自然人。对于使用真实IP上网的用户，UV和IP的数值是相同的。所以在此使用IP代替UV进行统计
     通过map-reduce解决统计问题
     1.在map端进行数据读入，使用正则表达式进行匹配，通过正则表达式将ip和访问时间提取出来，时间以每天为一个段，作为key，ip作为value，在reduce中创建	      一个hash表，去重后输出
     2.解决每天访问量top10的show统计
     	在map端进行数据读入，通过正则表达式匹配出ip和show，以show/number为key，ip作为value,进行数据分桶，讲相同的show分为一组，输出
	在reduce端进行遍历，进行排序，输出top10的show
     3.完成每天的次日留存统计
	次日留存即第二天还访问网站的用户，统计第二天的访问ip，通过map-reduce进行处理
	在map端，将数据进行读入，通过正则表达式匹配ip和时间，以ip作为key，时间作为value，在reduce中创建一个hash表，进行输入，一个ip对应的为两天时	  间的即为次日存留用户
	
