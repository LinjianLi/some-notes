# Cheat Sheet of Mine

## Check Info/Version

System info

```shell
cat /etc/os-release
screenfetch # need to install `screenfetch` first
```

CPU/GPU version

```shell
lscpu # CPU
cat /proc/cpuinfo # CPU
nvidia-smi -L # NVIDIA GPU
```

CUDA version

```shell
nvcc --version
# or
/usr/local/cuda/bin/nvcc --version
# or
cat /usr/local/cuda/version.txt
```

NVIDIA driver version

```shell
nvidia-smi
# or
cat /proc/driver/nvidia/version
```

Disk usage

```shell
du -sh dir # View directory size
           # -h for human readable format (KB, MB, GB, etc.)
           # -s for summary of the directory instead of show everything in the directory

du -sh dir | sort -h # Sort by the disk usage
                     # -h of `sort` command means --human-numeric-sort,
                     #     compare human readable numbers (e.g., 2K 1G)
```
