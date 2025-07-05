---

# vJunos_on_Proxmox_AMD
Fixing vJunos Boot Issues on Proxmox with AMD Processors

### Summary

I wanted to run **vJunos** on a QEMU-based infrastructure to set up a networking lab. While researching, I came across [vJunos\_on\_Proxmox](https://github.com/Ihemail/vJunos_on_Proxmox), which inspired me to host vJunos in my home lab.

However, the image wasn’t booting because it couldn't detect the required CPU virtualization flag. It turned out that the Juniper-provided BASH script, located inside the image, which checks for CPU capabilities, only supports Intel processors and doesn't properly detect AMD CPUs.

Here's what I did to work around this issue.

---

### Steps

After downloading the vJunos image, I mounted it as follows:

```bash
modprobe nbd max_part=8
qemu-nbd --connect=/dev/nbd0 ./vJunos-router-VERSION.qcow2
mkdir /mnt/temp
mount /dev/nbd0p2 /mnt/temp
cd /mnt/temp/home/pfe/junos
vi start-junos.sh
```

Then, I modified line 72 in `start-junos.sh` to include support for AMD CPUs:

```bash
CPU_FLAG=$(cat /proc/cpuinfo | grep -ciE 'vmx|svm')
```

The `svm` CPU flag on AMD processors stands for **Secure Virtual Machine**, indicating support for AMD-V (AMD Virtualization) technology. The original script only checked for Intel's `vmx` flag, so adding `svm` ensures compatibility with AMD processors.

Once editing is complete, unmount and disconnect the image:

```bash
umount /mnt/temp
qemu-nbd --disconnect /dev/nbd0
```
