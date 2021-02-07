
<!-- TOC -->

- [说明](#说明)
  - [APEX](#apex)
  - [OEM](#oem)
  - [AL32UTF8与UTF8区别](#al32utf8与utf8区别)
  - [11gR2](#11gr2)
  - [关于构建](#关于构建)
    - [下载安装包](#下载安装包)
    - [buildContainerImage](#buildcontainerimage)
    - [更改仓库及镜像名称](#更改仓库及镜像名称)
  - [启动](#启动)
    - [11gR2示例](#11gr2示例)
  - [使用](#使用)
    - [使用sqlPlus连接容器内数据库](#使用sqlplus连接容器内数据库)
      - [特权默认用户身份登录](#特权默认用户身份登录)
    - [单独使用sqlPlus](#单独使用sqlplus)
    - [网页访问APEX/OEM](#网页访问apexoem)
- [常见问题](#常见问题)
  - [doesn't have enough memory allocated](#doesnt-have-enough-memory-allocated)
- [参考资料](#参考资料)

<!-- /TOC -->

# 说明

以下内容根据[官方说明文档](https://github.com/oracle/docker-images/blob/main/OracleDatabase/SingleInstance/README.md)整理而来,具体细节冲突以官方文档为准.

## APEX

Oracle Application Express(缩写为APEX,以前称为Oracle HTML DB)是在Oracle数据库上运行的基于Web的软件 开发环境.<br>
所有Oracle数据库版本和Oracle自治数据库服务均已完全支持它并使其成为标准组件(不收取额外费用).<br>
从Oracle 11g开始,默认情况下将APEX作为核心数据库安装的一部分进行安装.

APEX可用于构建可在大多数现代Web浏览器中使用的复杂Web应用程序.<br>
APEX开发环境也是基于浏览器的.

## OEM

Oracle企业管理器(OEM)是一组基于Web的工具,旨在管理Oracle Corporation以及某些非Oracle实体生产的软件和硬件.<br>
应用于13c后版本.

## AL32UTF8与UTF8区别

> Oracle UTF8是8.1.7及更高版本中的Unicode版本3.0.<br>
在每个主要版本中,AL32UTF8均使用较新的Unicode版本进行了更新.<br>
在Oracle RDBMS 12.1中,其已更新为Unicode 6.1.

数据库默认字符集为: AL32UTF8.

## 11gR2

11gR2版本(11.2.0.2)数据库和其他版本的启动及挂载有些不同,下面有说明,更详细完整信息见官方文档.

## 关于构建

### 下载安装包

从[官网](https://www.oracle.com/database/technologies/xe-prior-releases.html)下载安装包至对应目录,<br>
具体安装包名称可以参考目录下Checksum.**文件内列举内容

### buildContainerImage

构建所依赖安装包放好后执行下面命令即可:

```bash
bash buildContainerImage.sh -v 11.2.0.2 -x
```

可用参数包括:

参数|含义|必需|示例
-|-|-|-|
-v|要构建的版本,可用版本参考目录名称|是|-v 11.2.0.2
-e|基于*企业版*创建镜像|否
-s|根据*标准版2*创建镜像|否
-x|基于*速成版*创建镜像|否
-i|忽略MD5校验和|否
-o|传递容器构建选项|否

### 更改仓库及镜像名称

镜像仓库及名称的配置由buildContainerImage.sh文件中IMAGE_NAME变量提供,<br>
可进行差异化更改:

```bash
IMAGE_NAME="ff4c00/database:${uname -i}-oracle-${VERSION}-${EDITION}-$(date +%Y%m%d)"
# IMAGE_NAME="ff4c00/database:x86_64-oracle-11.2.0.2-xe-20210206"
```

## 启动

```bash
docker run -d                                                \
--name 容器名称                                                \
-p 端口:1521 -p 端口:5500                                      \
-p 端口:1521 -p 端口:8080(APEX端口)(11g端口)                     \
-e ORACLE_SID=数据库SID                                        \
-e ORACLE_PDB=PDB(CDB中的单个实例)名称(默认: ORCLPDB1)            \
-e ORACLE_PWD=SYS及SYSTEM用户密码                               \
-e INIT_SGA_SIZE=SGA(System Global Area)内存大小(MB)(19.3.0起)  \
-e INIT_PGA_SIZE=PGA(Program Global Area)内存大小(MB)(19.3.0起) \
-e ORACLE_EDITION=数据库版本(19.3.0起)                          \
-e ORACLE_CHARACTERSET=字符集(默认:AL32UTF8)                    \
-v 数据挂载目录:/opt/oracle/oradata(除11g版本外挂载位置)           \
-v 数据挂载目录:/u01/app/oracle/oradata(11g挂载位置)
用户/仓库:镜像标签
```

### 11gR2示例

```bash
docker run -d                   \
--name oracle-11g-xe            \
--shm-size=2g                   \
-p 1521:1521 -p 5500:8080       \
-e ORACLE_SID=ORCL              \
-e ORACLE_PWD=o39i3U            \
-v /home/ff4c00/db/oracle/11.2.0.2/oradata:/u01/app/oracle/oradata \
ff4c00/database:x86_64-oracle-11.2.0.2-xe-20210206
```

## 使用

### 使用sqlPlus连接容器内数据库

#### 特权默认用户身份登录

```bash
docker exec -ti oracle-11g-xe su oracle
sqlplus /nolog
connect /as sysdba
```

### 单独使用sqlPlus

```bash
docker run --rm -ti 镜像仓库/标签 sqlplus 用户名/密码@//数据库主机ip:端口/数据库SID
```

### 网页访问APEX/OEM

通过暴露端口访问,根据数据库用户访问.

# 常见问题

## doesn't have enough memory allocated

> Error: The container doesn't have enough memory allocated.
A database XE container needs at least 1 GB of shared memory (/dev/shm).
You currently only have 64 MB allocated to the container.

容器无法正常启动 查看 docker logs 提示上述信息.

启动时添加 --shm-size 参数:

```
docker run -d  \
--shm-size=1g  \
ff4c00/database:x86_64-oracle-11.2.0.2-xe-20210206
```

# 参考资料

> [Github | docker-images](https://github.com/oracle/docker-images)

> [Oracle | Oracle Database XE Prior Release Archive](https://www.oracle.com/database/technologies/xe-prior-releases.html)

> [Github | Oracle Database container images](https://github.com/oracle/docker-images/blob/main/OracleDatabase/SingleInstance/README.md)

> [掘金 | Docker搭建Oracle](https://juejin.cn/post/6844903988362477576)

> [Github | Oracle XE container: Not enough memory allocated #458](https://github.com/oracle/docker-images/issues/458)

> [Oracle | 14 Memory Architecture](https://docs.oracle.com/database/121/CNCPT/memory.htm#CNCPT1237)

> [Oracle | Introduction to the Multitenant Architecture](https://docs.oracle.com/en/database/oracle/oracle-database/18/multi/introduction-to-the-multitenant-architecture.html#GUID-267F7D12-D33F-4AC9-AA45-E9CD671B6F22)

> [Oracle | Difference between AL32UTF8 and UTF8](https://community.oracle.com/tech/apps-infra/discussion/3514820/difference-between-al32utf8-and-utf8)

> [维基百科 | Oracle Application Express](https://en.wikipedia.org/wiki/Oracle_Application_Express)

> [维基百科 | Oracle Enterprise Manager](https://en.wikipedia.org/wiki/Oracle_Enterprise_Manager)