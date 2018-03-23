# 附录A: Docker 常用命令

tags : [docker]

---

这是一个比较常用的Docker 命令列表.

<!--more-->
	
|Purpose|	Command
|:----:|:----:|
|**Image**||
|Build an image|`docker image build --rm=true .`
|Install an image|`docker image pull ${IMAGE}`
|List of installed images|`docker image ls`
|List of installed images (detailed listing)|`docker image ls --no-trunc`
|Remove an image|`docker image rm ${IMAGE_ID}`
|Remove unused images|`docker image prune`
|Remove all images|`docker image rm $(docker image ls -aq)`
|**Containers**||
|Run a container|`docker container run`
|List of running containers|`docker container ls`
|List of all containers|`docker container ls -a`
|Stop a container|`docker container stop ${CID}`
|Stop all running containers|`docker container stop $(docker container ls -q)`
|List all exited containers with status 1|`docker container ls -a --filter "exited=1"`
|Remove a container|`docker container rm ${CID}`
|Remove container by a regular expression|`docker container ls -a &#124; grep wildfly &#124; awk '{print $1}' &#124; xargs docker container rm -f`
|Remove all exited containers|`docker container rm -f $(docker container ls -a &#124; grep Exit &#124; awk '{ print $1 }')`
|Remove all containers|`docker container rm $(docker container ls -aq)`
|Find IP address of the container|`docker container inspect --format '{{ .NetworkSettings.IPAddress }}' ${CID}`
|Attach to a container|`docker container attach ${CID}`
|Open a shell in to a container|`docker container exec -it ${CID} bash`
|Get container id for an image by a regular expression|`docker container ls &#124; grep wildfly &#124; awk '{print $1}'`

## 退出错误码

`docker run` 命令的错误码会提供一些容器运行失败以及为什么会退出的一些信息.完整的错误码列表如下:

https://docs.docker.com/engine/reference/run/#exit-status

所有的编码都是容器中命令运行的错误码.