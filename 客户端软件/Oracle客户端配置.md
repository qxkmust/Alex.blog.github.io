# Oracle客户端配置

```
一般情况下，Oracle 11G都是安装到服务器上的，而本机想要连接到服务端，需要安装OracleClient工具（官网下载最新）： https://www.oracle.com/database/technologies/instant-client/winx64-64-downloads.html （注意：oracleclient依赖运行环境需要安装VC++环境：https://support.microsoft.com/en-us/help/2977003/the-latest-supported-visual-c-downloads，否则plsql无法正常连接）

第一步：安装完成后，在OracleClient安装目录下新建目录(ora配置文件路径),新建tnsnames.ora文件，编辑如下内容：
数据库实例名 = 
	(
		DESCRIPTION = 
			(
				ADDRESS_LIST = (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.10.11)(PORT = 1521))
			) 
			(
				CONNECT_DATA = (SERVER = DEDICATED) (SERVICE_NAME = 服务名)
			)
	)
第二步：需要在系统环境变量增加:
| 系统变量名    | 值                                | 说明            |
| ----------- | --------------------------------- | --------------- |
| ORACLE_HOME |                                   | 客户端根路径    |
| TNS_ADMIN   |                                   | ora配置文件路径 |
| NLS_LANG    | SIMPLIFIED CHINESE_CHINA.ZHS16GBK | 中文编码        |

最后在系统Path中增加%ORACLE_HOME%
```

- 使用Navicat For Oracle连接：

  ```
  在Tools >> Options >> Environment 增加OCI环境的引用路径（指向OracleClient安装路径下的oci.dll）
  ```

- 使用PLSql连接：

  ```
  在Configure >> Preferences >> Connection中配置Oracle Home和OCI.dll路径
  ```

  