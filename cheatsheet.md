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

## File Manipulation

### Recursively delete all files of a specific extension in the current directory

Reference site: [askubuntu](https://askubuntu.com/questions/377438/how-can-i-recursively-delete-all-files-of-a-specific-extension-in-the-current-di)

```shell
find . -name "*.bak" -type f -delete
```

But use it with precaution. Run first:

```shell
find . -name "*.bak" -type f
```

Also, make sure that -delete is the last argument in your command. If you put it before the -name *.bak argument, it will delete everything.

### Batch renaming files in the current directory

For example, there are multiple files in the current directory.
Their names are like `a-01-bb.text`, `A-02-B.text`, `03-b.text`, etc..

If I only want to add a prefix name for each file:

```shell
for name in *
do
    mv ${name} prefix${name}
done
```

Now If I want to make their names follow the syntax like `a-<number>.txt`.
So, `a-01-bb.text`, `A-02-B.text`, and `03-b.text`, should be changed to `a-01.txt`, `a-02.txt`, and `a-03.txt`, respectively.

Use shell script and `sed` command.

First, check the output is correct before actually renaming the file (because complex regular expressions are not easy to write without making mistakes):

```shell
for name in *
do
    echo ${name} $(echo ${name} | sed -r 's/[^0-9]*([0-9]+)[^0-9.]*\.text/\1.txt/g')
done
```

If the output is correct, then replace the first `echo` with `mv`:

```shell
for name in *
do
    mv ${name} $(echo ${name} | sed -r 's/[^0-9]*([0-9]+)[^0-9.]*\.text/\1.txt/g')
done
```

Furthermore, if there are some other files whose extensions are not `.text`, I only want to rename the `.text` filse while keeping other files unchanged:

(the difference between the previous code and the following code is the `$` character in the `sed` script)

```shell
for name in *
do
    # After checking the output is correct, I can replace the first `echo` with `mv` command.
    echo ${name} $(echo ${name} | sed -r 's/[^0-9]*([0-9]+)[^0-9.]*\.text$/\1.txt/g')
done
```
