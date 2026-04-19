# vLLM Learning

1. LeaderWorkSet https://github.com/kubernetes-sigs/lws is API created by Kubernetes special interest group to aim to address the common patterns of deploying LLM on top of Kubernetes, expecially multi-node LLM sharding.
2. When we are deploying LLM across multiple-node, if 1 node is failing due to OOM, electricity, etc. the whole group that host the model need to be restarted. LeaderWorkSet is aim to handle this kind of operation.
3. why multiple-node LLM deployment? a single GPU have limited bandwidth of memory. smaller parameter model can be deployed in single GPU, some can be deployed in multiple GPU inside one physical host, larger model required the model to be deployed and sharded into multiple physical host.

![model-deployment](assets/1-model-deployment.png)

4. most of the LLM (large language model) today is based on GPT (Generative Pre-trained Transformer) which is using transformer architecture https://jalammar.github.io/illustrated-transformer/ (even I am still confuse right now haha)
5. Transformer architecture consist of multiple layer where there is attention sublayer and feed forward network sublayer. multi-gpu or multi-node LLM deployment work by distributing the layer across the GPU

```
Node 1 GPU 1: layers 0-7
Node 1 GPU 2: layers 8-15
Node 2 GPU 1: layers 16-23
Node 2 GPU 2: layers 24-31
```
this is only one example (pipeline parallelism) on how to distribute computational workload of AI models

6. 

7. when deploying model that would not be able to fit into 1 GPU, there is several ways to make it works:
- quantization
- prunning
- distillation
- parallelism https://www.infracloud.io/blogs/inference-parallelism/

8. Inference parallelism aim to distribute the computational workload of AI models. there are several method
- Tensor Parallelism
- Pipeline Parallelism
- Expert Parallelism

9. Pipeline Parallelism distribute the layer to multiple GPU. if model required 200 GB of GPU memory, with 4 Pipeline Parallelism, we can distribute the workload to GPU with only 50 GB of memory.

```
Node 1 GPU 0: layers 0-7
Node 1 GPU 1: layers 8-15
Node 1 GPU 2: layers 16-23
Node 1 GPU 3: layers 24-31
```
there is overhead with pipeline parallelism where GPU 2 need to wait for output of GPU 1.

10. Tensor Parallelism store all layers in all GPU. but only subnet of column that is required. eventually reducing the GPU memory needed. 

```
Node 1 GPU 0: layers 0-31 but only columns 0-1024    (all layers, subset of columns)
Node 1 GPU 1: layers 0-31 but only columns 1024-2048
Node 1 GPU 2: layers 0-31 but only columns 2048-3072
Node 1 GPU 3: layers 0-31 but only columns 3072-4096
```

but all GPU required faster communication channel.

11. always check nvidia-smi topo -m before deciding parallelism strategy.

12. nvidia-smi can output a topology of GPU inside the node. it can see how GPU can communicate each other. NV mean NVLink connection. 

```
nvidia-smi topo -m
        GPU0    GPU1    GPU2    GPU3    CPU Affinity    NUMA Affinity
GPU0     X      NV2     NV1     NV1     0-23            0
GPU1    NV2      X      NV1     NV1     0-23            0
GPU2    NV1     NV1      X      NV2     24-47           1
GPU3    NV1     NV1     NV2      X      24-47           1
```

