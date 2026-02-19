# Description
Install tools to make Docket GPU (NVDIA) compatibility.

## Stetps

- **Step01**: Installing the NVIDIA Container Toolkit

    Before start any docker container that uses our GPU, we must install some native toolkits in our System Operator

    ```
    $ sudo apt-get update && sudo apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    gnupg2

    $ curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
    && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
        sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
        sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

    $ sudo sed -i -e '/experimental/ s/^#//g' /etc/apt/sources.list.d/nvidia-container-toolkit.list

    $ sudo apt-get update

    $ export NVIDIA_CONTAINER_TOOLKIT_VERSION=1.18.2-1
    $ sudo apt-get install -y \
        nvidia-container-toolkit=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
        nvidia-container-toolkit-base=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
        libnvidia-container-tools=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
        libnvidia-container1=${NVIDIA_CONTAINER_TOOLKIT_VERSION}
    ```

- **Step02**: Configure docker to use the new Docker NVIDIA runtime

    This command configure docker runtime updating the file `/etc/docker/daemon.json`:

    ```
    $ sudo nvidia-ctk runtime configure --runtime=docker
    ```

    This is the new file updated, where a new runtime called nvidia using the native binaries localed in `nvidia-container-runtime` is created:

    ```
    {
        "runtimes": {
            "nvidia": {
                "args": [],
                "path": "nvidia-container-runtime"
            }
        }
    }
    ```

- **Step03**: Restart docker service

    Now we must restart docker service to use the new runtime:

    ```
    $ sudo systemctl restart docker
    ```

    If we execute this command in a server with the NVIDIA runtime configured we will in **Runtime** this value `Runtimes: io.containerd.runc.v2 nvidia runc` and in the **Discovered** tag our GPU detected
    ```
    $ docker info
    Client: Docker Engine - Community
    Version:    28.4.0
    Context:    default
    Debug Mode: false
    Plugins:
    buildx: Docker Buildx (Docker Inc.)
        Version:  v0.28.0
        Path:     /usr/libexec/docker/cli-plugins/docker-buildx
    compose: Docker Compose (Docker Inc.)
        Version:  v2.39.4
        Path:     /usr/libexec/docker/cli-plugins/docker-compose

    Server:
    Containers: 3
    Running: 3
    Paused: 0
    Stopped: 0
    Images: 4
    Server Version: 28.4.0
    Storage Driver: overlay2
    Backing Filesystem: extfs
    Supports d_type: true
    Using metacopy: false
    Native Overlay Diff: true
    userxattr: false
    Logging Driver: json-file
    Cgroup Driver: systemd
    Cgroup Version: 2
    Plugins:
    Volume: local
    Network: bridge host ipvlan macvlan null overlay
    Log: awslogs fluentd gcplogs gelf journald json-file local splunk syslog
    CDI spec directories:
    /etc/cdi
    /var/run/cdi
    Discovered Devices:
    cdi: nvidia.com/gpu=0
    cdi: nvidia.com/gpu=GPU-ddc0d57f-6fdc-6656-6fa9-43b4d02bdece
    cdi: nvidia.com/gpu=all
    Swarm: inactive
    Runtimes: io.containerd.runc.v2 nvidia runc
    Default Runtime: runc
    Init Binary: docker-init
    containerd version: b98a3aace656320842a23f4a392a33f46af97866
    runc version: v1.3.0-0-g4ca628d1
    init version: de40ad0
    Security Options:
    apparmor
    seccomp
    Profile: builtin
    cgroupns
    Kernel Version: 6.14.0-37-generic
    Operating System: Ubuntu 24.04.2 LTS
    OSType: linux
    Architecture: x86_64
    CPUs: 24
    Total Memory: 249.1GiB
    Name: simur-fractal
    ID: fabf8d57-191a-41f3-b79c-8b45a94d2ecb
    Docker Root Dir: /var/lib/docker
    Debug Mode: false
    Experimental: false
    Insecure Registries:
    ::1/128
    127.0.0.0/8
    Live Restore Enabled: false
    ```

    On the contrary if we execute the same command in other server without NVDIA runtime installed and configured we will not see any NVIDA runtime `Runtimes: runc io.containerd.runc.v2` and any GPU detected of course.

    ```
    $ docker info
    Client: Docker Engine - Community
    Version:    28.1.1
    Context:    default
    Debug Mode: false
    Plugins:
    buildx: Docker Buildx (Docker Inc.)
        Version:  v0.23.0
        Path:     /usr/libexec/docker/cli-plugins/docker-buildx
    compose: Docker Compose (Docker Inc.)
        Version:  v2.35.1
        Path:     /usr/libexec/docker/cli-plugins/docker-compose

    Server:
    Containers: 13
    Running: 13
    Paused: 0
    Stopped: 0
    Images: 15
    Server Version: 28.1.1
    Storage Driver: overlay2
    Backing Filesystem: extfs
    Supports d_type: true
    Using metacopy: false
    Native Overlay Diff: true
    userxattr: false
    Logging Driver: json-file
    Cgroup Driver: systemd
    Cgroup Version: 2
    Plugins:
    Volume: local
    Network: bridge host ipvlan macvlan null overlay
    Log: awslogs fluentd gcplogs gelf journald json-file local splunk syslog
    Swarm: inactive
    Runtimes: runc io.containerd.runc.v2
    Default Runtime: runc
    Init Binary: docker-init
    containerd version: 05044ec0a9a75232cad458027ca83437aae3f4da
    runc version: v1.2.5-0-g59923ef
    init version: de40ad0
    Security Options:
    apparmor
    seccomp
    Profile: builtin
    cgroupns
    Kernel Version: 6.11.0-25-generic
    Operating System: Ubuntu 24.04.2 LTS
    OSType: linux
    Architecture: x86_64
    CPUs: 20
    Total Memory: 31.03GiB
    Name: simur-MS-7B94
    ID: ad854bd7-2195-4c15-9ff0-ba5df3e47f9e
    Docker Root Dir: /var/lib/docker
    Debug Mode: false
    Username: ofertoio
    Experimental: false
    Insecure Registries:
    ::1/128
    127.0.0.0/8
    Live Restore Enabled: false
    ```

- **Step04**: Check our new NVIDIA Docker Runtime

    Now we can execute a tensorflow GPU compatible and check the GPU. We must use special tensorflow images with the tag latest-gpu

    We can execute a python console with GPU support
    ```
    $ docker run -it --rm \
    --runtime=nvidia \
    tensorflow/tensorflow:latest-gpu \
    python
    ```

    We can test our cuda test

    ```
    $ docker run -it --rm  \
    --runtime=nvidia  \
    -v /home/simur/git/cuda-test:/cuda-test  \
    tensorflow/tensorflow:latest-gpu  \
    python /cuda-test/test_tensorflow.py
    ```