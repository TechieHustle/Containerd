# Containerd
This contains all the finding that I come across while working on k3s with containerd runtime.

- Inspect a container:  
`ctr containers info`
- Configuration file created by k3s at: `/var/lib/rancher/k3s/agent/etc/containerd/config.toml`

# cgroup:
- cgroup driver for the runtime should be same as the driver for kubelet.
- By default, the systemd cgroup driver is false in containerd, which can be found from the above configuration file:  
`SystemdCgroup = false`
NOTE: In case of k3s on device, `SystemdCgroup` under `runc.options` might be missing, which is equivalent to `cgroupfs` being used as the runtime cgroup driver.
- In kubelet installed by k3s (`/var/lib/rancher/k3s/agent/etc/containerd/config.toml`) should not be changed as [mentioned](https://docs.k3s.io/advanced#configuring-containerd). Instead a new file `/var/lib/rancher/k3s/agent/etc/containerd/config.toml.tmpl` should be created in the same directory. And it is picked up by containerd.
- Content of the `config.toml.tmpl` file can be copied from `config.toml` file, while adding `SystemdCgroup = true` under `runc.options` at the end, as following:

```
version = 2

[plugins."io.containerd.internal.v1.opt"]
  path = "/var/lib/rancher/k3s/agent/containerd"
[plugins."io.containerd.grpc.v1.cri"]
  stream_server_address = "127.0.0.1"
  stream_server_port = "10010"
  ...
  ...

...
  ...
...
  ...


[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  runtime_type = "io.containerd.runc.v2"

[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
  SystemdCgroup = true
```

- Restart the k3s service: `systemctl restart k3s.service` on server node, and `systemctl restart k3s-agent.service` on worker node.
- Exporting and importing an image:
    - Export image using command: `ctr image export <tar-filename.tar> image:tag`  
      Example: `ctr image export ubuntu-arm64v8.tar docker.io/arm64v8/ubuntu:latest`
    - Import image from tar: `ctr image import <tar-filename.tar> image:tag`  
      Example: `ctr image import ubuntu-arm64v8.tar arm64v8/ubuntu:latest`
