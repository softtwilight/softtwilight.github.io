## Containers
A container is the runtime instance of image.

`docker container run <image> <command>`
`docker container stop` //stop
`docker container start` //restart
`docker container rm` 

VM 是在Hypervisor层上的，一个VM一个OS。hy...将硬件资源（cpu，RAM，disk）虚拟化给VM用。
container是OS层上的。将OS资源虚拟化，给container用。只有一个kernel。