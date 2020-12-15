# Nacos

> 什么是 [Nacos ](https://nacos.io/zh-cn/) 

在 Spring Cloud Netflix 阶段我们采用 Eureka 做作为我们的服务注册与发现服务器，现利用 Spring Cloud Alibaba 提供的 Nacos 组件替代该方案。 

**Nacos**  /nɑ:kəʊs/ 是 Dynamic <u>Na</u>ming and <u>Co</u>nfiguration <u>S</u>ervice的首字母简称。

一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。

Nacos 的关键特性包括: 

- **服务发现和服务健康监测** 
- **动态配置服务** 
- **动态 DNS 服务** 
- **服务及其元数据管理** 

##  安装部署

你可以通过源码、发行包、docker镜像方式来获取 Nacos。

### 从 Github 上下载源码方式

```shell
git clone https://github.com/alibaba/nacos.git
cd nacos/
mvn -Prelease-nacos -Dmaven.test.skip=true clean install -U  
ls -al distribution/target/

// change the $version to your actual path
cd distribution/target/nacos-server-$version/nacos/bin
```

### 下载编译后压缩包方式

您可以从 [最新稳定版本](https://github.com/alibaba/nacos/releases) 下载 `nacos-server-$version.zip` 包。 

```shell
unzip nacos-server-$version.zip 或者 tar -xvf nacos-server-$version.tar.gz
cd nacos/bin
```

### docker方式

```shell
git clone https://github.com/nacos-group/nacos-docker.git
cd nacos-docker
// 单机模式 Derby
docker-compose -f example/standalone-derby.yaml up
// 单机模式 MySQL
docker-compose -f example/standalone-mysql-5.7.yaml up
//  集群模式
docker-compose -f example/cluster-hostname.yaml up 
```
Nacos Server 启动后，进入 [http://ip:8848](http://ip:8848/) 查看控制台(默认账号名/密码为 nacos/nacos) 