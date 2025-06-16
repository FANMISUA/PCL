## 集群硬件资源


### 计算服务器

| 服务器名称          | CPU                                    | 内存      | 硬盘                                   | 显卡        | IP地址           |
| -------------- | -------------------------------------- | ------- | ------------------------------------ | --------- | -------------- |
| taishan        | Intel(R) Xeon(R) Gold 6226R @ 2.90GHz  | 251 GiB | sda: 893.3G + sdb: 9.1T + sdc: 32.8T | 10 x 3090 | 192.168.237.73 |
| huashan        | Intel(R) Xeon(R) Gold 6226R @ 2.90GHz  | 251 GiB | sda: 893.3G + sdb: 9.1T + sdc: 32.8T | 10 x 3090 | 192.168.237.74 |
| hengshan       | Intel(R) Xeon(R) Gold 6226R @ 2.90GHz  | 251 GiB | sda: 893.3G + sdb: 9.1T + sdc: 32.8T | 10 x 3090 | 192.168.237.75 |
| qianweitian    | Intel(R) Core(TM) i9-10900X @ 3.70GHz  | 31 GiB  | sda: 3.6T + nvme0n1: 953.9G          | 2 x 3090  | 192.168.63.86  |
| kunweidi       | Intel(R) Core(TM) i9-10900X @ 3.70GHz  | 62 GiB  | sda: 3.6T + nvme0n1: 953.9G          | 2 x 3090  | 192.168.63.87  |
| shuileizhun    | Intel(R) Core(TM) i9-10900X @ 3.70GHz  | 62 GiB  | sda: 3.6T + nvme0n1: 953.9G          | 2 x 3090  | 192.168.63.88  |
| shanshuimeng   | Intel(R) Xeon(R) Silver 4214 @ 2.20GHz | 125 GiB | sda: 223.6G + sdb: 1.7T              | 4 x 3090  | 192.168.81.17  |
| shuitianxu     | Intel(R) Xeon(R) Gold 5122 @ 3.60GHz   | 125 GiB | sda: 894.3G + sdb: 1.8T + sdc: 1.8T  | 4 x 3090  | 192.168.81.18  |
| tianshuisong   | Intel(R) Xeon(R) Gold 5122 @ 3.60GHz   | 125 GiB | sda: 894.3G + sdb: 1.8T + sdc: 1.8T  | 4 x 3090  | 192.168.81.14  |
| shuidibi       | Intel(R) Core(TM) i9-10900X @ 3.70GHz  | 62 GiB  | nvme0n1: 931.5G                      | 4 x 3090  | 192.168.81.15  |
| dishuishi      | Intel(R) Core(TM) i9-10900X @ 3.70GHz  | 62 GiB  | nvme0n1: 931.5G                      | 4 x 3090  | 192.168.81.16  |
| fengtianxiaoxu | —                                      | —       | —                                    | —         | 192.168.81.15  |
| tianzelu       | —                                      | —       | —                                    | —         | 192.168.81.16  |
|                |                                        |         |                                      |           |                |
|                |                                        |         |                                      |           |                |

### 存储服务器

192.168.63.190:/mnt/lab  174T  /mnt/net1
192.168.63.189:/mnt/lab  174T  /mnt/net0

集群的每台计算服务器都可以访问这两台共享存储服务器，且挂载的目录相同（/mnt/net*），可以将 python 环境装到共享目录下，从而保证每台服务器的 python 环境一致。


## 使用规范
我们的集群使用 Htcondor 来管理所有用户的任务和硬件资源。
所有用户都统一在任务提交节点：taishan 提交任务，所提交的任务会被 Htcondor 分发到集群中的所有机器。

> [!Note] 注意
> taishan 上有4张显卡(6,7,8,9)是用于调试程序的，这意味着 taishan 的 0--5 GPU 是被纳管到 condor 集群中的。
> 所以所有用户都不被允许在 0--5 GPU 进行调试，否则会与 condor 集群有资源冲突。




## Condor 系统

[condor 用户手册](https://htcondor.readthedocs.io/en/latest/users-manual/index.html)

#### 准备程序
`test.cu` :
``` 
#include <cuda_runtime.h>
#include <iostream>
#include <stdio.h>

__global__ void sleep_kernel(float seconds) {
    unsigned long long start_clock = clock64();
    unsigned long long wait_clocks = (unsigned long long)(seconds * 1.0e9);

    while (clock64() - start_clock < wait_clocks) {
        // do nothing, just wait
    }
}

int main(int argc, char const *argv[]) {
	float sleep_time = atof(argv[1]); // in seconds
    std::cout << "Sleeping for " << sleep_time / 60 << " minutes..." << std::endl;
    
    sleep_kernel<<<1, 1>>>(sleep_time);
    cudaDeviceSynchronize();

    std::cout << "Woke up after " << sleep_time / 60 << " minutes!" << std::endl;
    return 0;
}
```

`Makefile` :
``` 
OUTPUT = x.run
CC = nvcc
FLAGS = -O3 -gencode arch=compute_86,code=sm_86
DEBUG = 
GCCOPTIONS =-Wno-deprecated-gpu-targets
CFLAGS=

default: program

program:
	$(CC) $(FLAGS) $(CFLAGS) $(DEBUG) $(GCCOPTIONS) $(OBJECTS) test.cu -o $(OUTPUT)

clean:
	-rm -f $(OUTPUT)
```

编译：
make 
#### 创建 submit 文档
创建一个 `job1.submit` 文件，并添加以下内容到该文件中
```
Universe   = vanilla
executable = x.run
requestMemory = 1024 # 每个任务所需要的内存
request_GPUs = 1  # 每个任务所需要的gpu数量
# +SHORT_JOB=true # 如果程序的运行时间小于30min，则打开此选项
# Requirements = CUDACapability > 6.0
# Requirements = (TARGET.Machine=="taishan") || (TARGET.Machine=="huashan") || (TARGET.Machine=="hengshan")

Log        = logs/job_$(Cluster).$(Process).log
Output     = logs/job_$(Cluster).$(Process).out
Error      = logs/job_$(Cluster).$(Process).error

should_transfer_files = YES
when_to_transfer_output = ON_EXIT
transfer_input_files = x.run
#transfer_executable = false
initialdir = data/your_sub_directory

Arguments = 10
Queue
```

- 请注意，`+SHORT_JOB=true` 已被注释掉。集群中保留了 4 张 GPU 卡（短作业集群，~<30 mins)用于运行短时间任务，如果打开了该标志，您的 jobs 将可以同时使用短作业集群和长作业集群的资源。如果关闭该标志，您的 jobs 将只使用长作业集群的资源。

#### 提交 submit 文档

创建 logs 目录，存放日志

在终端 中键入`csub job1.submit`命令。要检查作业的状态，只需键入`cq YourUserName`，屏幕上将显示所有作业的状态信息。


#### 更改 condor 作业优先级
有时，你想运行一些快速测试，但你已经提交了一堆长时间运行的任务。如果你能让Condor先运行你的快速任务，然后再运行长时间任务，这将会很有效率。对此有一个解决方案！以下命令可以改变提交者的优先级，这样你就可以为相对重要的任务选择更高的优先级。默认优先级是0，你可以使用一个更小的数字来告诉Condor系统降低特定ID任务的优先级。例如，`condor_prio -p -15 6389`。这个命令会降低集群ID为6389的任务的优先级。想了解更多详情，请查看`condor_prio`命令的说明文档。

```
需要注意的是，为了日常调试或测试，我们在 taishan 上保留了4个GPU用作于调试。这意味着 taishan 的 0--5 GPU 是被纳管到 condor 集群中的，剩下的4个GPU留给你用于在Condor上进行大量参数搜索模拟之前测试你的代码。这4个GPU的ID是你运行nvidia-smi时看到的最后四个GPU（我还为此创建了一个别名nsmi作为简写）。

因此，你应该在main.cu文件中使用`cudaSetDevice(6)`或`cudaSetDevice(7)`或`cudaSetDevice(8)`或`cudaSetDevice(9)`来指定GPU设备号。一个合适的工作流程应该是：在终端中运行nsmi来查看第6、7、8或9号GPU中哪一个是空闲的，然后在你的main.cu文件中相应地设置cudaSetDevice。不要手动使用泰山上的任何其他GPU，因为Condor需要动态处理它们。

在你完成测试并准备提交Condor作业时，你必须在main.cu文件中明确设置`cudaSetDevice(0)`，这样Condor就知道如何从池中为你动态分配GPU。这对于Condor的正常工作非常重要！如果你不确定这里发生了什么，请随时问我。
```


## 在 condor 集群中运行 Python
### 确保集群所有服务器都正确安装 anaconda 和需要的依赖包

可以在一台服务器配好环境，然后 scp 整个 anaconda 文件夹到其他服务器

### 查看可供使用 GPU 资源

`condor_status -af Name Gpus`

### 编写脚本

- 创建一个名为 `test.py` 的 Python 文件
``` 
import numpy as np  
print('python is ok!')
```

- 创建一个包含以下内容的文件 `run.sh`：
``` 
#!/bin/bash  
echo job:$1  
/your/anaconda/bin/python test.py
```

- 使该脚本可执行：
```
chmod +x run.sh
```

- 创建 `test.submit` 的提交文件，其中包含以下内容：
```
executable              = run.sh  
output                  = logs/job_$(ClusterId).$(ProcId).out  
error                   = logs/job_$(ClusterId).$(ProcId).err  
log                     = logs/job_$(ClusterId).log  

# Requirements = (TARGET.Machine=="taishan") || (TARGET.Machine=="huashan") || (TARGET.Machine=="hengshan")

Universe   = vanilla  
+SHORT_JOB=true  
should_transfer_files   = YES  
transfer_input_files    = test.py   
when_to_transfer_output = ON_EXIT  
request_GPUs = 1  
request_CPUs = 1  
initialdir = data/your_sub_directory
​  
arguments               = $(ClusterId)$(ProcId)  
queue
```
request_GPUs : 需要的gpu数量
transfer_input_files ：运行所需要的文件
arguments：传给 `run.sh` 的参数

### 提交作业

- 创建 logs 目录，存放日志
    
- 使用以下命令提交作业：
    
```
condor_submit test.submit
```
### 关于环境变量。重要！！！（）

**condor的环境初始化跟使用命令行的初始化方式不一样。**

condor 并没有运行 `~/.bashrc` 文件

#### 解决方法一（推荐）：
将 `~/.bashrc` 的内容拷贝到 `source.sh` 文件，并删除下面的代码段。然后在运行脚本加上`source source.sh`
```
case $- in
*i*) ;;
  *) return;;
esac
```

`run.sh`改成
```
#!/bin/bash  
source source.sh  
echo job:$1  
python test.py
```

`test.sub`加一个`source.sh`文件
```
transfer_input_files    = test.py,source.sh 
```

#### 解决方法二：
手动在你的运行脚本把环境变量加上

#### 解决方法三：
使用 docker 来运行您的 job
