错误日志
========
 
### 问题：不小心删除了一些重要文件,导致无法启动Apache

再次重新部署Apache和PHP服务(Apache环境，选的是线程安全的PHP引擎）

1. http.conf配置，似乎就是修改一下Apache24目录，然后修改一些80端口；
2. 启动bin目录下的httpd程序启动apache服务，出现套接字错误，显示端口443错误。修改conf/extra目录下的http-ssl.conf文件中的443端口为442，再去尝试，还是出现错误；又把conf/original/extra/目录下的http-ssl.conf文件中的端口改了，居然还是没有作用。我开始怀疑人生了！直到......我关闭了一切VM服务！成功启动了，WTF!

### 配置apache解析php；
1. 添加LoadModule php5_module到"/php/php5apache2_4.dll";
2. 添加AddType application/x-httpd-php .php .phtml 
3. 添加phpinidir到"/php/" 设置php配置文件目录
4. 将php安装目录下的php.ini-devlopment文件重命名为php.ini;
5. 在php.ini中修改以下几项：  
  ; On windows:  
　;extension_dir = "D:/../php/ext"  
　;extension=php_curl.dll  
　;extension=php_mysql.dll#部署MySQL  

---

### 问题：尝试“httpd -k restart”重启过程中出现错误：AH00436: No installed service named "Apache2.4".

1) 搜索后在stackoverflow中找到答案:"just do a "httpd -k install" and the windows service is added to the registry"
2) 以管理员身份运行cmd，输入命令:httpd -k install，再次输入其他http -k命令后无错误提示
3) 如果在install期间出现套接字错误，原因可能是没有关闭apache服务。
4) 如出现错误：AH00373: Failed to open the 'Apache2.4' service，则可能是没有管理员权限。

期间曾出现打开php页面弹出下载框的错误，原因估计是在修改httpd.conf文件时出现：  
`LoadModule vhost_alias_module modules/mod_vhost_alias.so #其实这儿不用修改`  
`LoadModule php5_module "D:/../php/php5apache2_4.dll`"  

`phpIniDir "D:/../php"`

`AddType application/x-gzip .gz .tgz`   
`AddType application/x-httpd-php .php`  

都删除后一步一步撤回修改，成功解析，不再弹出下载框

---

### 部署MySQL
1. 配置好环境变量（方便一些）后，运行`mysqld --install`,期间出现错误；
打开普通cmd.exe出现Install/Remove of the Service Denied!，改成以管理员权限运行即可成功安装mysql服务；

2. `net start mysql`出现“MySQL 服务无法启动”，发现没有执行`mysqld --initialize`初始化，执行后出现错误
```sql
> [ERROR] --initialize specified but the data directory has files in it. Aborting.
> [ERROR] Aborting
```
错误原因：表示datadir指定的目录下已有数据文件  
解决办法：删除data目录后再执行`mysqld --initialize`  
> 没有显示结果，表示初始化成功

若继续执行net start mysql还是无法启动，则需要  
```sql
> mysqld -remove
```  
卸载mysql服务后，再依次执行
```sql
> mysqld --install
> mysql --initialize
> net start mysql
```
成功启动MySQL服务。

为什么如此一波三折！  
期间可能无法删除data目录，不知道因为什么占用文件，建议使用强力删除。

3. 以root用户身份进入mysql服务
```sql
mysql -u root -p
```
进入需要密码，若忘记，在data目录下有个err文件，在里面查找password密码，在密码前有个空格，要去掉才能登陆。

---

### 问题：show databases出现错误代码
 ```
 ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.  
```
错误信息提示需要重设密码
 ```sql
 > UPDATE mysql.user SET Password=PASSWORD('your_new_password') WHERE User='root';
```
运行代码  
`Query OK, 0 rows affected, 1 warning (0.00 sec)`   
 成功修改密码。
 
2019-08-01更新  
修改命令似乎不支持8.0版本，需改为：
```sql
alter USER 'root'@'localhost' identified by 'your_new_password';
```

---
#### Windows某些cmd命令

查看所有端口监听情况  
`netstat -ano`  
> TCP 127.0.0.1:8088  0.0.0.0:0  LISTENING 12168

查询指定端口的进程号  
`netstat -ano|findstr 8080`
>  TCP  0.0.0.0:8080  0.0.0.0:0  LISTENING  4028

查找进程号对应的进程名称  
`tasklist|findstr 4028`  
> httpd.exe 4028 Services   0   744 K

这里的http.exe就是进程名称，是Appache服务器的运行程序

停止某项进程  
`taskkill /pid 12168 /f`

---

### 日期：2019-05-07

### 问题：.gitignore无法忽略某些配置文件

> 描述：之前没有添加ignore的配置文件已经在远程分支上了，
再在.gitignore添加响应配置文件（路径正确）后push到远程分支发现配置文件还在

解决办法：在暂存区删除（工作区保留）该配置文件
`$ git rm --cached filename`  
再进行push，之后将不在track配置文件的更新


### 问题：方法func()需要在Transactional注解指定rollbackFor或者在方法中显式的rollback

先了解异常：
+ 可查的异常（checked exceptions）:Exception下除了RuntimeException之外的异常,如IOException和SQLException
- 不可查的异常（unchecked exceptions）:RuntimeException及其子类和错误（Error）  

若不对运行时异常进行处理，那么出现运行时异常之后，要么是线程中止，要么是主程序终止。

> Spring框架的事务基础架构代码将默认地 只 在抛出运行时和unchecked exceptions时才标识事务回滚。 也就是说，当抛出个RuntimeException 或其子类例的实例时。（Errors也一样 - 默认地 - 标识事务回滚。）从事务方法中抛出的Checked exceptions将 不 被标识进行事务回滚。

1. 让checked例外也回滚：在整个方法前加上 @Transactional(rollbackFor=Exception.class)

2. 让unchecked例外不回滚： @Transactional(notRollbackFor=RunTimeException.class)

3. 不需要事务管理的(只查询的)方法：@Transactional(propagation=Propagation.NOT_SUPPORTED)

#### 如果异常被try｛｝catch｛｝了，事务就不回滚了，如果想让事务回滚必须再往外抛try｛｝catch｛throw new Exception｝。

来自[博客](https://blog.csdn.net/Mint6/article/details/78363761)

---

### 日期：2019-05-09

### 问题：spring boot自动注入@Autowired出现Consider defining a bean of type 'xxx' in your configuration

```log
Description:

Field userService in xxxx.controllers.UserController required a bean of type 'xxxx.service.UserService' that could not be found.

The injection point has the following annotations:
	- @org.springframework.beans.factory.annotation.Autowired(required=true)

Action:

Consider defining a bean of type 'xxxx.service.UserService' in your configuration.
```

提示说无法找到一个指定自动注入类型的bean，正常情况下加上@Component注解的类会自动被Spring扫描到生成bean注册的Spring容器中，
而@Component相当于@Service@Controller和@Repository的作用，没有加上任何一个注解，自然无法被识别。

解决办法
+ 将接口与对应的实现类放在application启动类的同一个目录或子目录下，这样注解就可以被扫描到；
- 在对应的实现类上加上注解@Component或@Service
+ 在指定的application类加上注解@ComponentScan(basePackages={"service","service.Impl"});

### 问题：如下

```log
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'userRepository': Invocation of init method failed; nested exception is java.lang.IllegalArgumentException: This class [class com.xxx.xxx.xxx.xxxxx] does not define an IdClass
```

原因：在实体中增加了多个@Id注解（注意父类中的@Id注解定义的Id）

### 问题:PropertyReferenceException: No property findOne found for type 

原因：没有实现特有的repository数据库访问，只需加上为数据访问加上@Query即可

---

### 日期：2019-05-15

问题：本地master分支无法push到远程origin分支

```
$ git push origin master
ssh: Could not resolve hostname github.com: Name or service not known
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

在SSH Key还存在的情况下，不知是什么原因导致无法建立SSH connection，因此需要重新认证

尝试  
`$ ssh -T git@github.com`

得到信息  
`You've successfully authenticated, but GitHub does not provide shell access.`  
重新与远程库建立连接，并成功推送

