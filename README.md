**Before you try this, the link below may help you understand what GPUDirect Storage is:**
> https://github.com/developer-onizuka/what_is_GPUDirect-Storage

You can install GDS by through the release note below:
```
https://docs.nvidia.com/gpudirect-storage/release-notes/index.html
https://docs.nvidia.com/gpudirect-storage/troubleshooting-guide/index.html
```
But it is a little complicated, the followings might be helpful for you.

If you want to understand what GDS is, then you can learn URL below:

https://github.com/developer-onizuka/what_is_GPUDirect-Storage

# 0. Hardware
```
   (1) Optiplex 3050SFF
       Intel(R) Core(TM) i3-7100 CPU @ 3.90GHz
       DIMM slot1: DDR4 DIMM 4GB (Hynix)
       DIMM slot2: DDR4 DIMM 4GB (Patriot)
   (2) SATA SSD (for Ubuntu 20.04)
       Transcend SSD 120GB
       P/N: TS120GSSD220S
   (3) NVMe SSD
       KLEVV SSD 256GB CRAS C710 M.2 Type2280 PCIe3x4 NVMe 3D TLC NAND Flash
       P/N: K256GM2SP0-C71
   (4) NVIDIA Quadro P1000 (GP107GL)
```
# 1. Install Ubuntu 
```
   Install Ubuntu 20.04.
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
# 5. Install nvidia driver.
```
   $ sudo apt update
   $ sudo apt upgrade
   $ ubuntu-drivers devices
   WARNING:root:_pkg_get_support nvidia-driver-390: package has invalid Support Legacyheader, cannot determine support level
   == /sys/devices/pci0000:00/0000:00:01.0/0000:01:00.0 ==
   modalias : pci:v000010DEd00001CB1sv000010DEsd000011BCbc03sc00i00
   vendor   : NVIDIA Corporation
   model    : GP107GL [Quadro P1000]
   driver   : nvidia-driver-390 - distro non-free
   driver   : nvidia-driver-460-server - distro non-free
   driver   : nvidia-driver-465 - distro non-free
   driver   : nvidia-driver-418-server - distro non-free
   driver   : nvidia-driver-450-server - distro non-free
   driver   : nvidia-driver-460 - distro non-free recommended
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
   $ sudo apt-get update
   $ sudo apt install nvidia-gds
   $ sudo modprobe nvidia_fs
   $ dpkg -s nvidia-gds
   $ /usr/local/cuda/gds/tools/gdscheck -p
    GDS release version: 1.0.0.82
    nvidia_fs version:  2.7 libcufile version: 2.4
    ============
    ENVIRONMENT:
    ============
    =====================
    DRIVER CONFIGURATION:
    =====================
    NVMe               : Supported
    NVMeOF             : Unsupported
    SCSI               : Unsupported
    ScaleFlux CSD      : Unsupported
    NVMesh             : Unsupported
    DDN EXAScaler      : Unsupported
    IBM Spectrum Scale : Unsupported
    NFS                : Unsupported
    WekaFS             : Unsupported
    Userspace RDMA     : Unsupported
    --Mellanox PeerDirect : Enabled
    --rdma library        : Not Loaded (libcufile_rdma.so)
    --rdma devices        : Not configured
    --rdma_device_status  : Up: 0 Down: 0
    =====================
    CUFILE CONFIGURATION:
    =====================
    properties.use_compat_mode : true
    properties.gds_rdma_write_support : true
    properties.use_poll_mode : false
    properties.poll_mode_max_size_kb : 4
    properties.max_batch_io_timeout_msecs : 5
    properties.max_direct_io_size_kb : 16384
    properties.max_device_cache_size_kb : 131072
    properties.max_device_pinned_mem_size_kb : 33554432
    properties.posix_pool_slab_size_kb : 4 1024 16384 
    properties.posix_pool_slab_count : 128 64 32 
    properties.rdma_peer_affinity_policy : RoundRobin
    properties.rdma_dynamic_routing : 0
    fs.generic.posix_unaligned_writes : false
    fs.lustre.posix_gds_min_kb: 0
    fs.weka.rdma_write_support: false
    profile.nvtx : false
    profile.cufile_stats : 0
    miscellaneous.api_check_aggressive : false
    =========
    GPU INFO:
    =========
    GPU index 0 Quadro P1000 bar:1 bar size (MiB):256 supports GDS
    ==============
    PLATFORM INFO:
    ==============
    IOMMU: disabled
    Platform verification succeeded

```

# 9. Throughput test
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
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 1 -I 1 -T 10 -i 4096K
IoType: WRITE XferType: CPUONLY Threads: 1 DataSetSize: 10485760/10485760(KiB) IOSize: 4096(KiB) Throughput: 0.940200 GiB/sec, Avg_Latency: 4153.464453 usecs ops: 2560 total_time 10.636039 secs
    
(2) Storage->CPU->GPU
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 2 -I 1 -T 10 -i 4096K
IoType: WRITE XferType: CPU_GPU Threads: 1 DataSetSize: 10485760/10485760(KiB) IOSize: 4096(KiB) Throughput: 0.947653 GiB/sec, Avg_Latency: 4120.292188 usecs ops: 2560 total_time 10.552385 secs
    
(3) Storage -> GPU (GDS)
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 0 -I 1 -T 10 -i 4096K
IoType: WRITE XferType: GPUD Threads: 1 DataSetSize: 10485760/10485760(KiB) IOSize: 4096(KiB) Throughput: 0.976612 GiB/sec, Avg_Latency: 3998.571094 usecs ops: 2560 total_time 10.239486 secs

4. Seq Read Throughput
(1) Storage->CPU
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 1 -I 0 -T 10 -i 256K
IoType: READ XferType: CPUONLY Threads: 1 DataSetSize: 18165760/10485760(KiB) IOSize: 256(KiB) Throughput: 1.834943 GiB/sec, Avg_Latency: 133.045984 usecs ops: 70960 total_time 9.441286 secs

(2) Storage->CPU->GPU
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 2 -I 0 -T 10 -i 256K
IoType: READ XferType: CPU_GPU Threads: 1 DataSetSize: 14581760/10485760(KiB) IOSize: 256(KiB) Throughput: 1.495762 GiB/sec, Avg_Latency: 163.214115 usecs ops: 56960 total_time 9.297100 secs

(3) Storage -> GPU (GDS)
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 0 -I 0 -T 10 -i 256K
IoType: READ XferType: GPUD Threads: 1 DataSetSize: 18421760/10485760(KiB) IOSize: 256(KiB) Throughput: 1.833908 GiB/sec, Avg_Latency: 133.120372 usecs ops: 71960 total_time 9.579739 secs

5. Rand Write Throughput
(1) Storage->CPU
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 1 -I 3 -T 10 -i 4096K
IoType: RANDWRITE XferType: CPUONLY Threads: 1 DataSetSize: 10485760/10485760(KiB) IOSize: 4096(KiB) Throughput: 0.938541 GiB/sec, Avg_Latency: 4160.809375 usecs ops: 2560 total_time 10.654841 secs

(2) Storage->CPU->GPU
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 2 -I 3 -T 10 -i 4096K
IoType: RANDWRITE XferType: CPU_GPU Threads: 1 DataSetSize: 8192000/10485760(KiB) IOSize: 4096(KiB) Throughput: 0.851809 GiB/sec, Avg_Latency: 4583.641500 usecs ops: 2000 total_time 9.171656 secs

(3) Storage -> GPU (GDS)
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 0 -I 3 -T 10 -i 4096K
IoType: RANDWRITE XferType: GPUD Threads: 1 DataSetSize: 10485760/10485760(KiB) IOSize: 4096(KiB) Throughput: 0.922404 GiB/sec, Avg_Latency: 4233.433984 usecs ops: 2560 total_time 10.841231 secs

6. Rand Read Throughput
(1) Storage->CPU
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 1 -I 2 -T 10 -i 8192K
IoType: RANDREAD XferType: CPUONLY Threads: 1 DataSetSize: 18677760/10485760(KiB) IOSize: 8192(KiB) Throughput: 1.510579 GiB/sec, Avg_Latency: 5169.552632 usecs ops: 2280 total_time 11.791838 secs
        
(2) Storage->CPU->GPU
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 2 -I 2 -T 10 -i 8192K
IoType: RANDREAD XferType: CPU_GPU Threads: 1 DataSetSize: 18677760/10485760(KiB) IOSize: 8192(KiB) Throughput: 1.260354 GiB/sec, Avg_Latency: 6196.049123 usecs ops: 2280 total_time 14.132937 secs
    
(3) Storage -> GPU (GDS)
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 0 -I 2 -T 10 -i 8192K
IoType: RANDREAD XferType: GPUD Threads: 1 DataSetSize: 18677760/10485760(KiB) IOSize: 8192(KiB) Throughput: 1.463563 GiB/sec, Avg_Latency: 5335.688158 usecs ops: 2280 total_time 12.170641 secs
```
# 10. Build and Run
```
   $ sudo apt install nvidia-cuda-toolkit
   $ sudo apt install libssl-dev
   $ nvcc -I /usr/local/cuda/include/  -I /usr/local/cuda/targets/x86_64-linux/lib/ strrev_gds.cu -o strrev_gds.co -L /usr/local/cuda/targets/x86_64-linux/lib/ -lcufile -L /usr/local/cuda/lib64/ -lcuda -L   -Bstatic -L /usr/local/cuda/lib64/ -lcudart_static -lrt -lpthread -ldl -lcrypto -lssl
   $ echo -n "Hello, GDS World!" > test.txt
   $ ./strrev_gds.co test.txt 
   sys_len : 17
   !dlroW SDG ,olleH
   See also test.txt
   $ cat test.txt 
   !dlroW SDG ,olleH
   $ ./strrev_gds.co test.txt 
   sys_len : 17
   Hello, GDS World!
   See also test.txt
   $ cat test.txt 
   Hello, GDS World!
```  

Check cufile.log. If you found "compatible mode" in cufile.log, then it might not work well as GDS.
A compatibility mode is available for unsupported configurations that maps IO operations to a fallback path.
You can learn what is compatible mode at the URL below:
https://docs.nvidia.com/gpudirect-storage/overview-guide/index.html#comp-and-gen
```
 16-07-2021 08:29:17:433 [pid=2363 tid=2363] NOTICE  cufio-drv:625 running in compatible mode
```

# 11. Using other NVMe disk (2021/07/14 Updated)
```
Using other NVMe disk below:
   (8) NVMe SSD ... JPY 6,980
       KLEVV SSD 512GB CRAS C710 M.2 Type2280 PCIe3x4 NVMe 3D TLC NAND Flash
       P/N: K512GM2SP0-C71
       Performance Spec: Read 2050MB/s, Write 1650MB/s

3. Seq Write Throughput
(1) Storage->CPU
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 1 -I 1 -T 10 -i 4096K
IoType: WRITE XferType: CPUONLY Threads: 1 DataSetSize: 18677760/10485760(KiB) IOSize: 4096(KiB) Throughput: 1.508802 GiB/sec, Avg_Latency: 2588.413377 usecs ops: 4560 total_time 11.805725 secs
    
(2) Storage->CPU->GPU
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 2 -I 1 -T 10 -i 4096K
IoType: WRITE XferType: CPU_GPU Threads: 1 DataSetSize: 14581760/10485760(KiB) IOSize: 4096(KiB) Throughput: 1.331719 GiB/sec, Avg_Latency: 2932.466011 usecs ops: 3560 total_time 10.442328 secs
    
(3) Storage -> GPU (GDS)
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 0 -I 1 -T 10 -i 4096K
IoType: WRITE XferType: GPUD Threads: 1 DataSetSize: 14581760/10485760(KiB) IOSize: 4096(KiB) Throughput: 1.436404 GiB/sec, Avg_Latency: 2718.772191 usecs ops: 3560 total_time 9.681297 secs

4. Seq Read Throughput
(1) Storage->CPU
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 1 -I 0 -T 10 -i 256K
IoType: READ XferType: CPUONLY Threads: 1 DataSetSize: 18677760/10485760(KiB) IOSize: 256(KiB) Throughput: 1.836170 GiB/sec, Avg_Latency: 132.956305 usecs ops: 72960 total_time 9.700897 secs

(2) Storage->CPU->GPU
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 2 -I 0 -T 10 -i 256K
IoType: READ XferType: CPU_GPU Threads: 1 DataSetSize: 16373760/10485760(KiB) IOSize: 256(KiB) Throughput: 1.682742 GiB/sec, Avg_Latency: 145.078330 usecs ops: 63960 total_time 9.279634 secs

(3) Storage -> GPU (GDS)
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 0 -I 0 -T 10 -i 256K
IoType: READ XferType: GPUD Threads: 1 DataSetSize: 18933760/10485760(KiB) IOSize: 256(KiB) Throughput: 1.839722 GiB/sec, Avg_Latency: 132.699797 usecs ops: 73960 total_time 9.814873 secs

5. Rand Write Throughput
(1) Storage->CPU
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 1 -I 3 -T 10 -i 4096K
IoType: RANDWRITE XferType: CPUONLY Threads: 1 DataSetSize: 14581760/10485760(KiB) IOSize: 4096(KiB) Throughput: 1.365591 GiB/sec, Avg_Latency: 2859.819944 usecs ops: 3560 total_time 10.183319 secs

(2) Storage->CPU->GPU
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 2 -I 3 -T 10 -i 4096K
IoType: RANDWRITE XferType: CPU_GPU Threads: 1 DataSetSize: 14581760/10485760(KiB) IOSize: 4096(KiB) Throughput: 1.188176 GiB/sec, Avg_Latency: 3286.662360 usecs ops: 3560 total_time 11.703860 secs

(3) Storage -> GPU (GDS)
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 0 -I 3 -T 10 -i 4096K
IoType: RANDWRITE XferType: GPUD Threads: 1 DataSetSize: 14581760/10485760(KiB) IOSize: 4096(KiB) Throughput: 1.303765 GiB/sec, Avg_Latency: 2995.342978 usecs ops: 3560 total_time 10.666226 secs

6. Rand Read Throughput
(1) Storage->CPU
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 1 -I 2 -T 10 -i 8192K
IoType: RANDREAD XferType: CPUONLY Threads: 1 DataSetSize: 18677760/10485760(KiB) IOSize: 8192(KiB) Throughput: 1.595003 GiB/sec, Avg_Latency: 4895.904386 usecs ops: 2280 total_time 11.167692 secs
        
(2) Storage->CPU->GPU
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 2 -I 2 -T 10 -i 8192K
IoType: RANDREAD XferType: CPU_GPU Threads: 1 DataSetSize: 18677760/10485760(KiB) IOSize: 8192(KiB) Throughput: 1.352894 GiB/sec, Avg_Latency: 5772.172368 usecs ops: 2280 total_time 13.166224 secs
    
(3) Storage -> GPU (GDS)
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 0 -I 2 -T 10 -i 8192K
IoType: RANDREAD XferType: GPUD Threads: 1 DataSetSize: 18677760/10485760(KiB) IOSize: 8192(KiB) Throughput: 1.532309 GiB/sec, Avg_Latency: 5096.328947 usecs ops: 2280 total_time 11.624610 secs
```

# 12. Using Quadro P400 (Updated 2021/07/22)
```
Using P400 and other NVMe disk below:
   (8) NVMe SSD ... JPY 6,980
       KLEVV SSD 512GB CRAS C710 M.2 Type2280 PCIe3x4 NVMe 3D TLC NAND Flash
       P/N: K512GM2SP0-C71
       Performance Spec: Read 2050MB/s, Write 1650MB/s

$ nvidia-smi
Thu Jul 22 09:34:09 2021       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 470.42.01    Driver Version: 470.42.01    CUDA Version: 11.4     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  Quadro P400         On   | 00000000:01:00.0  On |                  N/A |
| 36%   51C    P0    N/A /  N/A |    292MiB /  1992MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|    0   N/A  N/A      1112      G   /usr/lib/xorg/Xorg                 31MiB |
|    0   N/A  N/A      1617      G   /usr/lib/xorg/Xorg                 99MiB |
|    0   N/A  N/A      1748      G   /usr/bin/gnome-shell               81MiB |
|    0   N/A  N/A      2170      G   ...AAAAAAAAA= --shared-files       71MiB |
+-----------------------------------------------------------------------------+

3. Seq Write Throughput
(1) Storage->CPU
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 1 -I 1 -T 10 -i 4096K
IoType: WRITE XferType: CPUONLY Threads: 1 DataSetSize: 14581760/10485760(KiB) IOSize: 4096(KiB) Throughput: 1.470923 GiB/sec, Avg_Latency: 2654.508989 usecs ops: 3560 total_time 9.454097 secs
  
(2) Storage->CPU->GPU
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 2 -I 1 -T 10 -i 4096K
IoType: WRITE XferType: CPU_GPU Threads: 1 DataSetSize: 14581760/10485760(KiB) IOSize: 4096(KiB) Throughput: 1.303067 GiB/sec, Avg_Latency: 2996.938202 usecs ops: 3560 total_time 10.671936 secs
    
(3) Storage -> GPU (GDS)
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 0 -I 1 -T 10 -i 4096K
IoType: WRITE XferType: GPUD Threads: 1 DataSetSize: 18677760/10485760(KiB) IOSize: 4096(KiB) Throughput: 1.440689 GiB/sec, Avg_Latency: 2710.856798 usecs ops: 4560 total_time 12.363874 secs

4. Seq Read Throughput
(1) Storage->CPU
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 1 -I 0 -T 10 -i 256K
IoType: READ XferType: CPUONLY Threads: 1 DataSetSize: 18165760/10485760(KiB) IOSize: 256(KiB) Throughput: 1.920392 GiB/sec, Avg_Latency: 127.125028 usecs ops: 70960 total_time 9.021191 secs

(2) Storage->CPU->GPU
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 2 -I 0 -T 10 -i 256K
IoType: READ XferType: CPU_GPU Threads: 1 DataSetSize: 16629760/10485760(KiB) IOSize: 256(KiB) Throughput: 1.644660 GiB/sec, Avg_Latency: 148.437869 usecs ops: 64960 total_time 9.642948 secs

(3) Storage -> GPU (GDS)
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 0 -I 0 -T 10 -i 256K
IoType: READ XferType: GPUD Threads: 1 DataSetSize: 18421760/10485760(KiB) IOSize: 256(KiB) Throughput: 1.919940 GiB/sec, Avg_Latency: 127.156003 usecs ops: 71960 total_time 9.150475 secs

5. Rand Write Throughput
(1) Storage->CPU
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 1 -I 3 -T 10 -i 4096K
IoType: RANDWRITE XferType: CPUONLY Threads: 1 DataSetSize: 14581760/10485760(KiB) IOSize: 4096(KiB) Throughput: 1.366004 GiB/sec, Avg_Latency: 2858.917697 usecs ops: 3560 total_time 10.180242 secs

(2) Storage->CPU->GPU
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 2 -I 3 -T 10 -i 4096K
IoType: RANDWRITE XferType: CPU_GPU Threads: 1 DataSetSize: 14581760/10485760(KiB) IOSize: 4096(KiB) Throughput: 1.198304 GiB/sec, Avg_Latency: 3258.894101 usecs ops: 3560 total_time 11.604944 secs

(3) Storage -> GPU (GDS)
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 0 -I 3 -T 10 -i 4096K
IoType: RANDWRITE XferType: GPUD Threads: 1 DataSetSize: 14581760/10485760(KiB) IOSize: 4096(KiB) Throughput: 1.302090 GiB/sec, Avg_Latency: 2999.182865 usecs ops: 3560 total_time 10.679949 secs

6. Rand Read Throughput
(1) Storage->CPU
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 1 -I 2 -T 10 -i 8192K
IoType: RANDREAD XferType: CPUONLY Threads: 1 DataSetSize: 18677760/10485760(KiB) IOSize: 8192(KiB) Throughput: 1.577479 GiB/sec, Avg_Latency: 4950.283772 usecs ops: 2280 total_time 11.291752 secs

(2) Storage->CPU->GPU
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 2 -I 2 -T 10 -i 8192K
IoType: RANDREAD XferType: CPU_GPU Threads: 1 DataSetSize: 18677760/10485760(KiB) IOSize: 8192(KiB) Throughput: 1.337016 GiB/sec, Avg_Latency: 5840.784649 usecs ops: 2280 total_time 13.322582 secs

(3) Storage -> GPU (GDS)
$ gdsio -f /mnt/test10G -d 0 -n 0 -w 1 -s 10G -x 0 -I 2 -T 10 -i 8192K
IoType: RANDREAD XferType: GPUD Threads: 1 DataSetSize: 18677760/10485760(KiB) IOSize: 8192(KiB) Throughput: 1.538554 GiB/sec, Avg_Latency: 5075.678070 usecs ops: 2280 total_time 11.577428 secs
```
