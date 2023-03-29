# Docker

## Prequisites

Docker containers share the kernel with the host operating system, therefore the
ROCm kernel-mode driver must be installed on the host. Please refer to the
[Basic Installation Guide](./install.md) for details. The other user-space parts
(like the HIP-runtime or math libraries) of the ROCm stack will be loaded from
the container image and don't need to be installed to the host.

## Accessing GPUs in containers

In order to access GPUs in a container (to run applications using HIP, OpenCL or
OpenMP offloading) explicit access to the GPUs must be granted.

The ROCm runtimes make use of multiple device files:

- `/dev/kfd`: the main compute interface shared by all GPUs
- `/dev/dri/renderD<node>`: direct rendering interface (DRI) devices for each
  GPU. **`<node>`** is a number for each card in the system starting from 128.

Exposing these devices to a container is done by using the
[`--device`](https://docs.docker.com/engine/reference/commandline/run/#device)
option, i.e. to allow access to all GPUs expose `/dev/kfd` and all
`/dev/dri/renderD` devices:

```shell
docker run --device /dev/kfd --device /dev/renderD128 --device /dev/renderD129 ...
```

More conveniently, instead of listing all devices, the entire `/dev/dri` folder
can be exposed to the new container:

```shell
docker run --device /dev/kfd --device /dev/dri
```

Note that this gives more access than strictly required, as it also exposes the
other device files found in that folder to the container.

### Restricting a container to a subset of the GPUs

If a `/dev/dri/renderD` device is not exposed to a container then it cannot use
the GPU associated with it; this allows to restrict a container to any subset of
devices.

For example to allow the container to access the first and third GPU start it
like:

```shell
docker run --device /dev/kfd --device /dev/dri/renderD128 --device /dev/dri/renderD130 <image>
```

## Docker images in the ROCm ecosystem

### Base images

<https://github.com/RadeonOpenCompute/ROCm-docker> hosts images useful for users
wishing to build their own containers levaraging ROCm. The built images are
available from [Docker Hub](https://hub.docker.com/u/rocm). In particular
`rocm/rocm-terminal` is a small image with the prequisites to build HIP
applications, but does not include any libraries.

### Applications

AMD provides pre-built images for various GPU-ready applications through its
Infinity Hub at <https://www.amd.com/en/technologies/infinity-hub>.
