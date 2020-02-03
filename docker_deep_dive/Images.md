### Images
images are like stopped containers.
build阶段的构建。
一个image 可以run 多次，要删除image，所有运行在其上的container都必须停止。

Images 通常很小，通常只包含一些filesystem objects, linux系统的image一般比windows小得多。

`docker image pull <repository>:<tag>`
如果不是docker hub的仓库，需要指定DNS名(gcr.io是google仓库）：
`docker image pull gcr.io/xxx/xxx:v2` 

#### Images and layers
A Docker image is just a bunch of loosely-connected read-only layers.
查看image的layer信息：
`docker image inspect ubuntun:lastest`
docker有一个storage driver, 让stacking layers 以统一的文件系统呈现，添加或更新文件，加在新的层里。
不同的image可以共享layer。

#### image digests
docker1.10以后，每一个image都有一个image内容的hash值，称为digest。因为tag可能会变，tag指向的image也可能会变。digest是不变的。
`docker image ls --digests ubuntu` 

删除image：
`docker image rm <imageId>`

删除所有image：
`docker image rm $(docker image ls -q) -f`
