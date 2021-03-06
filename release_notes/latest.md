# Version 0.7 (2016-10-28)

In version 0.7, HyperContainer and RunV supports several new architectures, introduced some new features, and kept improving the stability. Here are some highlight features of the release.

## hyperd

- More arch supports: s390x, ppc64le, and arm64.
- **VM Template**: faster boot performance (130ms) and less memory consumption (save 80MB per pod/VM).
- Improve gRPC APIs.
- Improve streaming IO (attach & exec) for containers.
- Many other fixes and improvements.

## runV

- Support system arch s390x and ppc64le.
- Support system arch ARM64.
- Enable VM template for runV and runV-containerd, which improves the boot performance to 130ms and reduces 80MB memory consumption per container.
- Enable CNI, OVS and improve the networking configurations.
- Add QoS Control for network interface.
- Allow one volume to be mounted to multiple mount points of one container.
- Improve streaming IO for containers.
- Move dependencies from Godep to vendors.
- Many other fixes and improvements.
