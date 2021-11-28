## Docker + Jenkins 自动化部署

### Docker环境安装

- 安装 yum-utils:

```
yum install -y yum-utils device-mapper-persistent-data lvm2
```

- 为 yum 源添加 docker 仓库位置

```
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

- 安装 docker 社区版:

```
yum install docker-ce
```

- 启动 docker:

```
systemctl start docker
```

### Docker 开启 2375 端口

```
// 作用：为了docker-maven-plugin 将jar包打成镜像时可上传到 docker
// 注意：阿里云安全组不能开启2375端口给外界访问，否则挖矿警告
// 猜想：为什么一定要是 0.0.0.0，因为 docker 相当于一个容器，如果需要容器外访问，必须为 0.0.0.0，反之如果为 127.0.0.1 或 localhost，只能容器内访问
vim /usr/lib/systemd/system/docker.service
在 ExecStart 的最后面加上 -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock
退出并执行
systemctl daemon-reload
systemctl restart docker
```

### Jenkins 的安装及配置

- 下载 Jenkins 的 Docker 镜像

```
docker pull jenkins/jenkins:lts
```

- 在 Docker 容器中运行 Jenkins

```
docker run -p 8080:8080 -p 50000:5000 --name jenkins \
-u root \
-v /mydata/jenkins_home:/var/jenkins_home \
-d jenkins/jenkins:lts
```

- Jenkins 的配置

```
打开网址
http://[您的服务器Ip]:8080/
用下面指令获取密码
docker logs jenkins
安装推荐的插件
进入系统之后，点击系统管理->插件管理，找到 ssh安装插件, 然后设置 maven,设置 ssh
```

- 打包部署 SprinBoot 应用

```
代码上次git
注意需要在 pom.xml 中加上 docker-maven-plugin
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>docker-maven-plugin</artifactId>
    <version>1.2.2</version>
    <executions>
        <execution>
            <id>build-image</id>
            <phase>package</phase>
            <goals>
                <goal>build</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <imageName>${project.artifactId}:${project.version}</imageName>
        <dockerHost>http://[这里填的是内网Ip]:2375</dockerHost>
        <baseImage>adoptopenjdk/openjdk11:ubi</baseImage>
        <entryPoint>["java", "-jar","/${project.build.finalName}.jar"]</entryPoint>
        <resources>
            <resource>
                <targetPath>/</targetPath>
                <directory>${project.build.directory}</directory>
                <include>${project.build.finalName}.jar</include>
            </resource>
        </resources>
    </cofiguration>
</plugin>
```

- 执行脚本准备，将脚本文件命名为项目名称上传到 /mydata/sh 上

```
#!/usr/bin/env bash
app_name='[项目名称]'
docker stop ${app_name}
echo '----stop container----'
docker rm ${app_name}
echo '----rm container----'
docker run -p 8088:8080 --name ${app_name} \
--link mysql:db \
-v /etc/localtime:/etc/localtime \
-v /mydata/app/${app_name}/logs:/var/logs \
-d mall-tiny/${app_name}:1.0-SNAPSHOT
echo '----start container----'
```

- 给 .sh 脚本添加个执行权限

```
chmod +x ./xx.sh
```

- 创建一个任务

```
jenkins 上新建项目，设置名称，设置 git，构建maven，clean package &{WORKSPACE}/pom.xml，设置 ssh：/mydata/sh/xx.sh
```

- 执行构建，等待运行成功