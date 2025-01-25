# Docker

## Docker容器是怎么访问外网的

Docker默认的网络模式是桥接模式（bridge）。在这种网络模式下，Docker会在主机上创建一个虚拟网桥docker0，然后为这个网桥分配一个子网。针对Docker创建的容器，会创建一个Veth设备对，一端关联到网桥上，另一端关联到容器内的eth0设备，然后在网桥的地址段内给eth0分配一个IP地址。当容器需要访问外部网络时，数据包会通过这个虚拟网桥，然后通过主机的物理网卡发送出去

## Docker底层实现原理

### Namespace

Linux 命名空间技术通过隔离进程的视图，使得容器在操作系统上运行时，相互之间不会干扰

对Namespace的操作，主要是通过 clone、setns、unshare 这三个系统调用来完成。

- clone 可以用来创建新的 Namespace
- unshare 调用的进程会被放进新的 Namespace
- setns 将进程放到已有的 Namespace

| Namespace类型    | 系统调用参数    | 隔离目标                                                     |
| ---------------- | --------------- | ------------------------------------------------------------ |
| Mount Namespace  | CLONE_NEWNS     | 隔离文件系统挂载点                                           |
| UTS Namespace    | CLONE_NEWUTS    | 隔离主机名和域名等系统识别信息                               |
| IPC Namespace    | CLONE_NEWIPC    | 隔离某些进程间通信（IPC）资源，包括 System V IPC 和 POSIX IPC |
| PID Namespace    | CLONE_NEWPID    | 隔离容器内的进程编号空间                                     |
| Net Namespace    | CLONE_NEWNET    | 隔离网络设备、网络栈、端口和路由信息等                       |
| User Namespace   | CLONE_NEWUSER   | 隔离用户和用户组的编号空间                                   |
| Cgroup Namespace | CLONE_NEWCGROUP | 隔离控制组（cgroup）资源                                     |

### Cgroup

全称Control Group（控制组），用于限制、隔离容器的资源使用情况（如 CPU、内存、磁盘 I/O）

kubelet 和 底层容器运行时都需要对接控制组，以强制对Pod和容器进行资源管理和资源配置，如CPU和内存资源的请求和限制。若要对接控制组，kubelet 和 容器运行时需要使用一个cgroup驱动。关键的一点是kubelet和容器运行时需使用相同的cgroup驱动，并且采用相同的配置

可用的 cgroup 驱动有两个：

- cgroupfs
- systemd

#### cgroupfs 驱动

`cgroupfs` 驱动是kubelet 中默认的 cgroup 驱动。 当使用 `cgroupfs` 驱动时， kubelet 和容器运行时将直接对接 cgroup 文件系统来配置 cgroup。

当用 systemd 作为初始化系统时，不推荐使用 `cgroupfs` 驱动，因为 systemd 期望系统上只有一个 cgroup 管理器。 

如果使用 cgroup v2， 则应用 `systemd` cgroup 驱动取代 `cgroupfs`

#### systemd cgroup 驱动

当某个Linux系统发行版使用systemd作为其初始化系统时，初始化进程会生成并使用一个root控制组（cgroup），并充当cgroup管理器。

systemd 与 cgroup 集成紧密，并将为每个systemd单元分配一个cgroup。因此，如果你systemd用作初始化系统，同时使用cgroupfs驱动，则系统中会存在两个不同的cgroup管理器。

同时存在两个cgroup管理器将造成系统中针对可用的资源和使用中的资源出现两个视图。某些情况下，配置为对kublet和容器运行时使用cgroupfs、但对其余进程使用systemd的节点，在资源压力增大时会变得不稳定。

当systemd是选定的初始化系统时，缓解这个不稳定问题的方法是选择systemd作为kubelet和容器运行时的cgroup驱动。

### UnionFS

联合文件系统是 Docker 镜像的基础

- 镜像层叠加: Docker 镜像由多个只读层组成，UnionFS 将这些层合并成一个统一的视图。每层可以覆盖和扩展上层文件系统的内容。
- 容器文件系统: 当容器启动时，Docker 使用 UnionFS 将镜像的层叠加到一个可写层上，从而创建容器的文件系统视图。

rootfs（根文件系统）：Docker 在容器启动时，会使用 UnionFS 将镜像的只读层叠加到一个可写层上，形成容器的 rootfs。这个 rootfs 是容器内的实际文件系统视图，容器对其进行读写操作



## 实践

### 已运行容器添加端口映射

容器还没有构建时想添加端口映射，只需要在创建容器的时候添加 `-p` 参数，示例：

```
docker run -d -p 80:80 -p 443:443 --name nginx nginx 
```

但是如果容器已经运行了，没有命令可以直接去修改容器端口的

**方法一**

将当前容器构建成新镜像，在此基础上新添加端口映射

```
docker stop $A
docker commit $A imageA
docker rm $A
docker run -d -p 80:80 -p 443:443 --name A imageA
```

**方法二**

修改容器的配置文件

查看容器id

```
docker ps |grep A
```

进入容器目录（注意MacOS该路径并不能直接访问）

```
cd /var/lib/docker/containers/$containerID
```

修改hostconfig.json文件（记得备份），PortBindings添加要映射的端口，如

```
"8081/tcp":[{"HostIp":"127.0.0.1","HostPort":"8081"}]
```

修改config.v2.json文件（记得备份），ExposedPorts添加要映射的端口，如

```
"8081/tcp":{}
```

重启docker

```
systemctl restart docker
```

### MacOS Docker 

#### 容器的数据卷挂载问题

MacOS 是在本地运行xhyve 虚拟机管理的docker，容器卷是在虚拟机的文件系统中创建， 在macOS的FileSystem无法直接访问。

使用nsenter

```
docker run -it --rm --privileged --pid=host debian nsenter -t 1 -m -u -n -i sh
```



