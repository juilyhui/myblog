```toml
# post title, required
title = "Docker Swarm 入门"
# post slug, use to build permalink and url, required
slug = "Docker Swarm 入门"
# post description, show in header meta
desc = "welcome to izgnod's blog"
# post created time, support
# 2015-11-28, 2015-11-28 12:28, 2015-11-28 12:28:38
date = "2016-04-10 11:00:00"
# post updated time, optional
# if null, use created time
# author identifier, reference to meta [[author]], required
author = "izgnod"
# thumbnails to the post
thumb = "@media/golang.png"
# tags, optional
tags = ["Docker Swarm 入门"]
```

###目录：
  1. Swarm 是什么
  2. Swarm 基本的架构
  3. Swarm 安装以及实验环境。
  4. Swarm 基本使用
  

## 1. Swarm 是什么

 1. Swarm 就是一套管理Docker集群的工具，把分散在各处的Docker容器归总在一起，通过调度合适的Swarm node分配容器在何处运行。
    说明：Swarm daemon 挂了也不会影响容器的运行
## 2. Swarm 的基本架构
![图一张](http://img.ptcms.csdn.net/article/201501/26/54c609a31de28.jpg)

角色：
  1. Docker Deamon：管理本机Docker容器的，需要开启Docker Remote Api。
  2. Swarm node: 跑在Docker Deamon 上的一个容器，相当于Swarm的客户端。
  3. Swarm mannager：用来管理Swarm 集群的，相当于Swarm的服务器端。
  4. Docker Client：主要是用来发指令的，操纵集群的。
  
组件：
  1. node discovery:主要用来发现集群的，有好几种方式：例如：：Docker Hub 的服务发现功能，本地文件，etcd，counsel，zookeeper等方法。
  2. cluster:主要是用来调度的，例如：将容器跑在哪台机器上。
  3. strategy:调度策略，主要是用来设定调度的策略的。
    1. random:什么都不考虑，随机的，不常用，测试用。
    2. spread: 容器最少的节点运行新的节点。分散容器到各个机器。默认的方式。
    3. binpack: 容器尽可能一台机器上运行。集中容器到各个机器。
  4. 过滤器（filter）：
   1. Constraint Filter：这个是通过标签指定在哪台机器上部署容器。
   2. Affinity Filter：联姻。
   3.  Port Filter
## 3. Swarm 如何安装以及实验环境
### 实验环境部署：
  0. 由于服务器有限，采用Docker in Docker 的方式，可以认为和真实的三台服务器一样，上面都跑了Docker Deamon。
      1. IP：s1:172.17.0.6, s2:172.17.0.7, s3:172.17.0.8,
      2. 全部打开Remote Api,端口使用2375。
      3. s1和s2为Swarm node，s3为Swarm mannager
      4. [一个简单的环境](https://github.com/FuckAll/docker_practice/tree/master/swarm)，可以参考这个简单的docker-compose环境部署，详看README，使用这种方式可以忽略下面的安装步骤。
  1. Swarm 是管理Docker集群的，Docker官方已经做成了镜像，镜像包括了Swarm的所有功能，使用起来方便
 ```
 sudo docker pull swarm # 可以使用alauda.cn
  ```
  2.  验证一下Swarm安装是否成功
  ```
  sudo docker run --rm swarm -v
  # 返回类似如下内容表示成功
  swarm version 1.1.3 (7e9c6bd)
  ```
  3.  设置Docker Api 监控的端口(使用docker in docker 都不用设置这个)
  ```
  Ubuntu 修改/etc/default/docker 文件
  CentOs 修改/etc/sysconfig/docker 文件
  将OPTIONS改为：
  OPTIONS='-H 0.0.0.0:2375 -H unix:///var/run/docker.sock --selinux-enabled'
  重启Docker。
  ```
  4. 建立服务发现，服务发现有好几种方法可用，例如：Docker Hub 的服务发现功能，本地文件，etcd，counsel，zookeeper等方法。
  ```
  演示用Docker Hub 获取tocken的方式。
  在任意的一台机器上执行
  sudo docker run --rm swarm create
  会返回一个Token,这个Token是全球唯一的标识，表示创建成功。
  ```
5. 加入集群：
 ```
  sudo docker run --rm swarm join --addr=ip_address:2375 token://token_id
  ip_address 是本机的s1,s2,s3的，token 是上一步返回的。
  又返回值表示成功,CTRL+C退出即可。
  sudo docker run -d  swarm join --addr=ip_address:2375 token://token_id
  将本机加入到集群中，如果退出就会退出集群。
 ```
 
  6. 启动swarm mannager
 ```
  sudo docker run -d -p 2376:2375 swarm manage token://token_id 
  注意：上面的host的端口不能用2375因为我们上面docker api的端口用的是2375
  这里返回的是一个manager启动之后的容器id。(swarm mandage 会是用2375端口)
 ```
 
  7. 查看集群中的Docker Deamon
```
sudo docker run --rm swarm list token://token_id
列出所有的Docker Deamon
```
  8. 调度策略
  ```
执行：
docker -H s3_ip:2376 run --name node-1 -d -P redis
docker -H s3_ip:2376 run --name node-2 -d -P redis
docker -H s3_ip:2376 run --name node-3 -d -P redis
发现分别在集群的三台机器上出现对应的容器，即默认的方式是spread。
改变策略：
docker run -d -p 2376:2375 swarm manage --strategy binpack token://toke_id
同样执行上述三条命令：
发现全部建立在s3上。运行多次的时候发现我的云主机硬盘不够用了，所以spread方式是比较注意资源的利用的，而binpack避免了服务的碎片化。
  ```
  9. 过滤器：
   1. Constraint Filter: 
   ```
   Docker Deamon 启动的时候加：--label label_name==xxx 参数：
   执行docker run 的时候添加：-e constarint:key=value 就可以指定host部署容器。
   ```
   2. Affinity Filter：
  ```
   联姻，也就是说和指定的容器跑在同一个host上，执行：
   先生成一个可以被联姻的容器。
   docker -H manager_ip_addr:2376 run -d --name redis redis
   联姻上一个容器。
   docker -H manager_ip_addr:2376 run -d --name redis_1 -e affinity:container==redis redis
   发现跑在同一个host上了，执行多次也一样。
   特别注意：
   这个也可以指定，只有下载相关镜像的host才能运行指定的容器。例如：
   docker –H manager_ip_addr:2376 run –name redis1 –d –e affinity:image==redis redis
   只有下载了redis镜像的host才能运行名字为redis1的容器。如果所有的host上都没有redis镜像，那么就默认的策略指定一台host下载镜像，然后启动。
  ```
   3. Port Filter
  ```
  这个是说，某一类容器使用了固定的端口，其他的容器就不能使用了，如果使用就启动失败。
  docker -H manager_ip_addr:2376 run -d -p 80:80 nginx
  这个nginx启动之后，如果再启动nginx使用80端口就不能了，好坑啊。
  ```
    总结:Swarm基础就这么多，感觉基本用用还可以，但是面对复杂的业务需求，特性还是太少。


