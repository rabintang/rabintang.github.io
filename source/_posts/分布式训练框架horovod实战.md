---
title: 分布式训练框架horovod实战
tags:
  - 工具
  - horovod
  - 分布式
categories:
  - 机器学习
  - 工具
date: 2019-01-24 15:33:32
---


### Horovod简介


### Horovod安装
##### 1. 安装Open MPI
mpiexec --version

* 下载[Open MPI](https://www.open-mpi.org/software/ompi/v4.0/)安装包，建议安装open-mpi 4.0及以上版本；
* 执行以下命令安装：
    ``` bash
    tar -zvxf openmpi-4.0.0.tar.gz
    cd openmpi-4.0.0
    ./configure --prefix=/usr/local
    sudo make all install
    sudo ldconfig
    ```
* 执行`ompi_info`命令查看Open MPI版本信息；

##### 2. GPU机器安装NCCL
* 从Nvidia官网下载[NCCL](https://developer.nvidia.com/nccl)，这里下载相应cuda版本的NCCL deb安装包；
* 执行以下命令安装：
    ``` bash
    sudo dpkg -i nccl-repo-<version>.deb
    sudo apt install libnccl2=2.3.7-1+cuda9.0 libnccl-dev=2.3.7-1+cuda9.0
    ```

##### 3. 安装horovod
* CPU机器直接执行：`pip install --no-cache-dir horovod`；
* GPU机器执行：`HOROVOD_GPU_ALLREDUCE=NCCL pip install --no-cache-dir horovod`；

##### 4. 问题与解决方案
1. **ERROR**：mpicxx: error while loading shared libraries: libopen-pal.so.40: cannot open shared object file: No such file or directory
  **Solution**：执行`sudo ldconfig`，更新链接库缓存
2. horovod项目也提供了一个[常见问题列表](https://github.com/uber/horovod/blob/master/docs/troubleshooting.md)，还是很贴心的；

### Horovod使用
##### 1. 示例代码
```
import horovod.tensorflow as hvd

def _model_fn(features, labels, mode, params, config):
    ...
    optimizer = tf.train.AdagradOptimizer(0.01 * hvd.size())
    optimizer = hvd.DistributedOptimizer(optimizer)
    ...

hvd.init()
sess_config = tf.ConfigProto()
sess_config.gpu_options.visible_device_list = str(hvd.local_rank())
model_config = tf.estimator.RunConfig(session_config=sess_config)
model_dir = model_dir if hvd.rank() == 0 else None
model = tf.estimator.Estimator(
        model_fn=_model_fn,
        model_dir=model_dir,
        config=model_config,
        params=model_params
    )
hooks = [hvd.BroadcastGlobalVariablesHook(0)]
model.train(input_fn=input_fn, steps=max_train_steps // hvd.size(), hooks=hooks)
```

##### 2. 注意事项
* horovod是基于MPI和ring-allreduce算法来实现的分布式计算框架，是基于数据并行的分布式框架，因此，要注意切分在每个worker上的训练数据，horovod自身不支持数据切分；
* 要确保每个worker的迭代步数相当，或者迭代所有worker的最小步数，否则在计算平均梯度时，会出错；

##### 3. 加速比
在美拍热门CTR数据上进行评测，原单卡100步耗时3S+，现在单机4卡，每块卡100步耗时8S+，加速比（多卡并行时间/单卡运行时间）1.6，而并行效率（加速比/并行度）才0.39。
