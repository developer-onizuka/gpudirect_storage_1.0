You can install GDS by through the release note below:
```
https://docs.nvidia.com/gpudirect-storage/release-notes/index.html
https://docs.nvidia.com/gpudirect-storage/troubleshooting-guide/index.html
```
But it is a little complicated, the followings might be helpful for you.

# 0. Hardware
```
   (1) Optiplex 5050SFF  ... JPY 29,150
       Intel(R) Core(TM) i3-7500 CPU @ 3.50GHz
       DIMM slot1: DDR4 DIMM 8GB (Micron)
       DIMM slot2: Empty
       DIMM slot3: Empty
       DIMM slot4: Empty
       HDD 500GB  ---> Windows10 pro
       DVD DRIVE  ---> replace to SATA SSD(Ubuntu 20.04)
   (2) SATA SSD  ... JPY 2,500
       Transcend SSD 120GB
       P/N: TS120GSSD220S
   (3) DDR4 DIMM 8GB x2 ... JPY 5,555
       Micron Memory DDR4 2666MHz PC4-2400T-UA1-11
   (4) DDR4 DIMM 8GB ... JPY 2,555
       Hynix Memory DDR4 2400MHz PC4-19200
       HMA81GU6AFR8N-UH
   (5) NVMe SSD ... JPY 3,980
       KLEVV SSD 256GB CRAS C710 M.2 Type2280 PCIe3x4 NVMe 3D TLC NAND Flash
       P/N: K256GM2SP0-C71
   (6) ETC
       -Zheino 2nd 9.5mm Note PC drive mounter ... JPY 899
       -GLOTRENDS M.2 Heatsink ... JPY 650
   (7) NVIDIA Quadro P1000 ... JPY 15,800
   ----- Total JPY 61,089 -----
```
# 1. Install Ubuntu 
```
   Install Ubuntu 20.04 as "Minimal Install" and don't select "install third-party software for graphics and Wi-Fi hardware and additional media formats".
   Followings are optional, but it is very convenient.
   $ sudo vi /etc/apt/apt.conf.d/20auto-upgrades
     APT::Periodic::Update-Package-Lists "0";
     APT::Periodic::Unattended-Upgrade "0";
   $ sudo visudo
     username ALL=NOPASSWD: ALL

   See also followings:
   https://qiita.com/RyodoTanaka/items/e9b15d579d17651650b7
   https://thr3a.hatenablog.com/entry/20170805/1501943406
```
# 2. Check if the kernel version
```
   Check if the kernel versionis 5.4.0-42-generic with "uname -r". If it's true, Update all of softwares (200~400MB). 
```
# 3. Check iommu status and disable SecureBoot at BIOS menu
```
   $ dmesg | grep -i iommu
   See the release note above URL. You need reboot after making disable iommu.
   $ cat /etc/default/grub |grep iommu
   GRUB_CMDLINE_LINUX_DEFAULT="quiet splash intel_iommu=off"
   $ sudo update-grub
   $ sudo reboot

```   
# 4. Install MOFED5.3
```
 Download MLNX_OFED_LINUX-5.3-1.0.0.1-ubuntu20.04-x86_64.tgz.
   $ sudo apt-get install python3-distutils
   $ cd MLNX_OFED_LINUX-5.3-1.0.0.1-ubuntu20.04-x86_64/
   $ sudo ./mlnxofedinstall --with-nfsrdma --with-nvmf --enable-gds --add-kernel-support
   $ sudo update-initramfs -u -k `uname -r`
   $ sudo reboot
```
# 5. Install nvidia driver and nvidia cuda tool kit.
```
   $ sudo apt update
   $ sudo apt upgrade
   $ ubuntu-drivers devices
     == /sys/devices/pci0000:00/0000:00:01.0/0000:01:00.0 ==
     modalias : pci:v000010DEd00001CB3sv000010DEsd000011BEbc03sc00i00
     vendor   : NVIDIA Corporation
     model    : GP107GL [Quadro P400]
     driver   : nvidia-driver-450-server - distro non-free
     driver   : nvidia-driver-418-server - distro non-free
     driver   : nvidia-driver-460 - distro non-free recommended
     driver   : nvidia-driver-450 - distro non-free
     driver   : nvidia-driver-390 - distro non-free
     driver   : xserver-xorg-video-nouveau - distro free builtin
   $ sudo apt install nvidia-driver-460
   $ shutdown -r now
```
# 6. Check nvidia-smi
```
   You might see cuda-11.2 was already installed. But please note cuda is still 10.1 in the step.
```
# 7. Install CUDA-11.4
```
   $ wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-ubuntu2004.pin
   $ sudo mv cuda-ubuntu2004.pin /etc/apt/preferences.d/cuda-repository-pin-600
   $ wget https://developer.download.nvidia.com/compute/cuda/11.4.0/local_installers/cuda-repo-ubuntu2004-11-4-local_11.4.0-470.42.01-1_amd64.deb
   $ sudo dpkg -i cuda-repo-ubuntu2004-11-4-local_11.4.0-470.42.01-1_amd64.deb
   $ sudo apt-key add /var/cuda-repo-ubuntu2004-11-4-local/7fa2af80.pub
   $ sudo apt-get update
   $ sudo apt-get -y install cuda
```
# 8. Install GDS
```
   $ sudo dpkg -i gpudirect-storage-local-repo-ubuntu2004-0.95.0-cuda11.2_1.0-1_amd64.deb
   $ sudo apt-key add /var/gpudirect-storage-local-repo-ubuntu2004-0.95.0-cuda11.2/7fa2af80.pub 
   $ sudo apt-get update
   $ sudo apt install nvidia-gds
   $ sudo modprobe nvidia_fs
   $ dpkg -s nvidia-gds
   $ /usr/local/cuda/gds/tools/gdscheck -p
   GDS release version (beta): 0.95.0.94
   nvidia_fs version:  2.6 libcufile version: 2.3
   cuFile CONFIGURATION:
   NVMe           : Supported
   NVMeOF         : Unsupported
   SCSI           : Unsupported
   SCALEFLUX CSD  : Unsupported
   NVMesh         : Unsupported
   LUSTRE         : Unsupported
   GPFS           : Unsupported
   NFS            : Unsupported
   WEKAFS         : Unsupported
   USERSPACE RDMA : Unsupported
   --MOFED peer direct  : enabled
   --rdma library       : Not Loaded (libcufile_rdma.so)
   --rdma devices       : Not configured
   --rdma_device_status : Up: 0 Down: 0
   properties.use_compat_mode : 1
   properties.use_poll_mode : 0
   properties.poll_mode_max_size_kb : 4
   properties.max_batch_io_timeout_msecs : 5
   properties.max_direct_io_size_kb : 16384
   properties.max_device_cache_size_kb : 131072
   properties.max_device_pinned_mem_size_kb : 33554432
   properties.posix_pool_slab_size_kb : 4 1024 16384 
   properties.posix_pool_slab_count : 128 64 32 
   properties.rdma_peer_affinity_policy : RoundRobin
   properties.rdma_dynamic_routing : 0
   fs.generic.posix_unaligned_writes : 0
   fs.lustre.posix_gds_min_kb: 0
   fs.weka.rdma_write_support: 0
   profile.nvtx : 0
   profile.cufile_stats : 0
   miscellaneous.api_check_aggressive : 0
   GPU INFO:
   GPU index 0 Quadro P1000 bar:1 bar size (MiB):256 supports GDS
   IOMMU : disabled
   Platform verification succeeded

```
# 9. Additional software
```
   $ sudo apt install libssl-dev
   $ sudo apt install net-tools 
   $ sudo apt install openssh-server
   $ sudo update-alternatives --config java
   There are 2 choices for the alternative java (providing /usr/bin/java).

     Selection    Path                                            Priority   Status
   ------------------------------------------------------------
     0            /usr/lib/jvm/java-11-openjdk-amd64/bin/java      1111      auto mode
     1            /usr/lib/jvm/java-11-openjdk-amd64/bin/java      1111      manual mode
   * 2            /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java   1081      manual mode
```

# 10. Througput test
```
1. C-state Disable
   $ grep cstate /etc/default/grub
   GRUB_CMDLINE_LINUX="intel_idle.max_cstate=0 processor.max_cstate=0"
   $ sudo update-grub
   $ sudo reboot
   $ cat /sys/devices/system/cpu/cpuidle/current_driver
   none
   
2. preparation about NVMe SSD
   (1) mount nvme as "ordered" mode.
   # sudo mount -t ext4 -o data=ordered /dev/nvme0n1 /mnt

3. Seq Write Throughput
(1) Storage->CPU
    1.03GB/s from NVMe to system memory.

    $ gdsio -f /mnt/test1G -d 0 -n 0 -w 1 -s 1G -x 1 -I 1 -T 10
    IoType: WRITE XferType: CPUONLY Threads: 1 DataSetSize: 10461184/1048576(KiB) IOSize: 1024(KiB) Throughput: 1.039233 GiB/sec, Avg_Latency: 939.634103 usecs ops: 10216 total_time 9.599926 secs
    
(2) Storage->CPU->GPU
    Bounce buffer occuring from GPU memory to NVMe and it was 1.03GB/s as you can see below:
    
    $ gdsio -f /mnt/test1G -d 0 -n 0 -w 1 -s 1G -x 2 -I 1 -T 10
    IoType: WRITE XferType: CPU_GPU Threads: 1 DataSetSize: 11509760/1048576(KiB) IOSize: 1024(KiB) Throughput: 1.038694 GiB/sec, Avg_Latency: 940.127580 usecs ops: 11240 total_time 10.567659 secs
    
(3) Storage -> GPU (GDS)
    GDS eliminates bounce buffer so that it can write at 1.04GB/s.

    $ gdsio -f /mnt/test1G -d 0 -n 0 -w 1 -s 1G -x 0 -I 1 -T 10
    IoType: WRITE XferType: GPUD Threads: 1 DataSetSize: 10461184/1048576(KiB) IOSize: 1024(KiB) Throughput: 1.040754 GiB/sec, Avg_Latency: 938.261355 usecs ops: 10216 total_time 9.585902 secs

4. Seq Read Throughput
(1) Storage->CPU
    This value means that NVMe's seq-read throughput from NVMe to system memory is 1.66GB/s.

    $ gdsio -f /mnt/test1G -d 0 -n 0 -w 1 -s 1G -x 1 -I 0 -T 10
    IoType: READ XferType: CPUONLY Threads: 1 DataSetSize: 17801216/1048576(KiB) IOSize: 1024(KiB) Throughput: 1.660507 GiB/sec, Avg_Latency: 588.060458 usecs ops: 17384 total_time 10.223722 secs

(2) Storage->CPU->GPU
    Bounce buffer occuring from NVMe to GPU memory and it was 1.49GB/s as you can see below:

    $ gdsio -f /mnt/test1G -d 0 -n 0 -w 1 -s 1G -x 2 -I 0 -T 10
    IoType: READ XferType: CPU_GPU Threads: 1 DataSetSize: 15704064/1048576(KiB) IOSize: 1024(KiB) Throughput: 1.490721 GiB/sec, Avg_Latency: 655.031364 usecs ops: 15336 total_time 10.046521 secs

(3) Storage -> GPU (GDS)
    GDS eliminates bounce buffer so that it can read at 1.67GB/s.

    $ gdsio -f /mnt/test1G -d 0 -n 0 -w 1 -s 1G -x 0 -I 0 -T 10
    IoType: READ XferType: GPUD Threads: 1 DataSetSize: 17801216/1048576(KiB) IOSize: 1024(KiB) Throughput: 1.678453 GiB/sec, Avg_Latency: 581.768983 usecs ops: 17384 total_time 10.114409 secs

5. Rand Write Throughput
(1) Storage->CPU
    $ gdsio -f /mnt/test1G -d 0 -n 0 -w 1 -s 1G -x 1 -I 3 -T 10
    IoType: RANDWRITE XferType: CPUONLY Threads: 1 DataSetSize: 10461184/1048576(KiB) IOSize: 1024(KiB) Throughput: 0.940263 GiB/sec, Avg_Latency: 1038.538175 usecs ops: 10216 total_time 10.610397 secs

(2) Storage->CPU->GPU
    $ gdsio -f /mnt/test1G -d 0 -n 0 -w 1 -s 1G -x 2 -I 3 -T 10
    IoType: RANDWRITE XferType: CPU_GPU Threads: 1 DataSetSize: 9412608/1048576(KiB) IOSize: 1024(KiB) Throughput: 0.924835 GiB/sec, Avg_Latency: 1055.857920 usecs ops: 9192 total_time 9.706119 secs

(3) Storage -> GPU (GDS)
    $ gdsio -f /mnt/test1G -d 0 -n 0 -w 1 -s 1G -x 0 -I 3 -T 10
    IoType: RANDWRITE XferType: GPUD Threads: 1 DataSetSize: 9412608/1048576(KiB) IOSize: 1024(KiB) Throughput: 0.931682 GiB/sec, Avg_Latency: 1048.084639 usecs ops: 9192 total_time 9.634797 secs

6. Rand Read Throughput
(1) Storage->CPU
    $ gdsio -f /mnt/test1G -d 0 -n 0 -w 1 -s 1G -x 1 -I 2 -T 10
    IoType: RANDREAD XferType: CPUONLY Threads: 1 DataSetSize: 12558336/1048576(KiB) IOSize: 1024(KiB) Throughput: 1.132377 GiB/sec, Avg_Latency: 862.325750 usecs ops: 12264 total_time 10.576477 secs
        
(2) Storage->CPU->GPU
    $ gdsio -f /mnt/test1G -d 0 -n 0 -w 1 -s 1G -x 2 -I 2 -T 10
    IoType: RANDREAD XferType: CPU_GPU Threads: 1 DataSetSize: 10461184/1048576(KiB) IOSize: 1024(KiB) Throughput: 1.022459 GiB/sec, Avg_Latency: 955.010278 usecs ops: 10216 total_time 9.757416 secs
    
(3) Storage -> GPU (GDS)
    $ gdsio -f /mnt/test1G -d 0 -n 0 -w 1 -s 1G -x 0 -I 2 -T 10
    IoType: RANDREAD XferType: GPUD Threads: 1 DataSetSize: 12558336/1048576(KiB) IOSize: 1024(KiB) Throughput: 1.146988 GiB/sec, Avg_Latency: 851.332436 usecs ops: 12264 total_time 10.441746 secs
