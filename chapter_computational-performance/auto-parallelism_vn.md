<!-- ===================== Bắt đầu dịch Phần 1 ===================== -->
<!-- ========================================= REVISE PHẦN 1 - BẮT ĐẦU =================================== -->

<!--
# Automatic Parallelism
-->

# *dịch tiêu đề phía trên*
:label:`sec_auto_para`

<!--
MXNet automatically constructs computational graphs at the backend. 
Using a computational graph, the system is aware of all the dependencies, and can selectively execute multiple non-interdependent tasks in parallel to improve speed. 
For instance, :numref:`fig_asyncgraph` in :numref:`sec_async` initializes two variables independently. 
onsequently the system can choose to execute them in parallel.
-->

*dịch đoạn phía trên*

<!--
Typically, a single operator will use all the computational resources on all CPUs or on a single GPU. 
For example, the `dot` operator will use all cores (and threads) on all CPUs, even if there are multiple CPU processors on a single machine. 
The same applies to a single GPU. 
Hence parallelization isn't quite so useful single-device computers. 
With multiple devices things matter more. 
While parallelization is typically most relevant between multiple GPUs, adding the local CPU will increase performance slightly. 
See e.g., :cite:`Hadjis.Zhang.Mitliagkas.ea.2016` for a paper that focuses on training computer vision models combining a GPU and a CPU. 
With the convenience of an automatically parallelizing framework we can accomplish the same goal in a few lines of Python code. 
More broadly, our discussion of automatic parallel computation focuses on parallel computation using both CPUs and GPUs, as well as the parallelization of computation and communication.
We begin by importing the required packages and modules. 
Note that we need at least one GPU to run the experiments in this section.
-->

*dịch đoạn phía trên*

```{.python .input}
import d2l
from mxnet import np, npx
npx.set_np()
```

<!-- ===================== Kết thúc dịch Phần 1 ===================== -->

<!-- ===================== Bắt đầu dịch Phần 2 ===================== -->

<!--
## Parallel Computation on CPUs and GPUs
-->

## *dịch tiêu đề phía trên*

<!--
Let's start by defining a reference workload to test - the `run` function below performs 10 matrix-matrix multiplications on the device of our choosing using data allocated into two variables, `x_cpu` and `x_gpu`.
-->

*dịch đoạn phía trên*

```{.python .input}
def run(x):
    return [x.dot(x) for _ in range(10)]

x_cpu = np.random.uniform(size=(2000, 2000))
x_gpu = np.random.uniform(size=(6000, 6000), ctx=d2l.try_gpu())
```

<!--
Now we apply the function to the data. 
To ensure that caching doesn't play a role in the results we warm up the devices by performing a single pass on each of them prior to measuring.
-->

*dịch đoạn phía trên*

```{.python .input}
run(x_cpu)  # Warm-up both devices
run(x_gpu)
npx.waitall()

with d2l.benchmark('CPU time: %.4f sec'):
    run(x_cpu)
    npx.waitall()

with d2l.benchmark('GPU time: %.4f sec'):
    run(x_gpu)
    npx.waitall()
```

<!--
If we remove the `waitall()` between both tasks the system is free to parallelize computation on both devices automatically.
-->

*dịch đoạn phía trên*

```{.python .input}
with d2l.benchmark('CPU&GPU : %.4f sec'):
    run(x_cpu)
    run(x_gpu)
    npx.waitall()
```

<!--
In the above case the total execution time is less than the sum of its parts, since MXNet automatically schedules computation on both CPU and GPU devices without the need for sophisticated code on behalf of the user.
-->

*dịch đoạn phía trên*

<!-- ===================== Kết thúc dịch Phần 2 ===================== -->

<!-- ===================== Bắt đầu dịch Phần 3 ===================== -->

<!-- ========================================= REVISE PHẦN 1 - KẾT THÚC ===================================-->

<!-- ========================================= REVISE PHẦN 2 - BẮT ĐẦU ===================================-->

<!--
## Parallel Computation and Communication
-->

## *dịch tiêu đề phía trên*

<!--
In many cases we need to move data between different devices, say between CPU and GPU, or between different GPUs. 
This occurs e.g., when we want to perform distributed optimization where we need to aggregate the gradients over multiple accelerator cards. 
Let's simulate this by computing on the GPU and then copying the results back to the CPU.
-->

*dịch đoạn phía trên*

```{.python .input}
def copy_to_cpu(x):
    return [y.copyto(npx.cpu()) for y in x]

with d2l.benchmark('Run  on GPU: %.4f sec'):
    y = run(x_gpu)
    npx.waitall()

with d2l.benchmark('Copy to CPU: %.4f sec'):
    y_cpu = copy_to_cpu(y)
    npx.waitall()
```

<!--
This is somewhat inefficient. 
Note that we could already start copying parts of `y` to the CPU while the remainder of the list is still being computed. 
This situatio occurs, e.g., when we compute the (backprop) gradient on a minibatch. 
The gradients of some of the parameters will be available earlier than that of others. 
Hence it works to our advantage to start using PCI-Express bus bandwidth while the GPU is still running. 
Removing `waitall` between both parts allows us to simulate this scenario.
-->

*dịch đoạn phía trên*

```{.python .input}
with d2l.benchmark('Run on GPU and copy to CPU: %.4f sec'):
    y = run(x_gpu)
    y_cpu = copy_to_cpu(y)
    npx.waitall()
```

<!--
The total time required for both operations is (as expected) significantly less than the sum of their parts. 
Note that this task is different from parallel computation as it uses a different resource: the bus between CPU and GPUs. 
In fact, we could compute on both devices and communicate, all at the same time. 
As noted above, there is a dependency between computation and communication: `y[i]` must be computed before it can be copied to the CPU. 
Fortunately, the system can copy `y[i-1]` while computing `y[i]` to reduce the total running time.
-->

*dịch đoạn phía trên*

<!--
We conclude with an illustration of the computational graph and its dependencies for a simple two-layer MLP when training on a CPU and two GPUs, as depicted in :numref:`fig_twogpu`. 
It would be quite painful to schedule the parallel program resulting from this manually. 
This is where it is advantageous to have a graph based compute backend for optimization.
-->

*dịch đoạn phía trên*

<!--
![Two layer MLP on a CPU and 2 GPUs.](../img/twogpu.svg)
-->

![*dịch chú thích ảnh phía trên*](../img/twogpu.svg)
:label:`fig_twogpu`

<!-- ===================== Kết thúc dịch Phần 3 ===================== -->

<!-- ===================== Bắt đầu dịch Phần 4 ===================== -->

<!--
## Summary
-->

## *dịch tiêu đề phía trên*

<!--
* Modern systems have a variety of devices, such as multiple GPUs and CPUs. They can be used in parallel, asynchronously.
* Modern systems also have a variety of resources for communication, such as PCI Express, storage (typically SSD or via network), and network bandwidth. They can be used in parallel for peak efficiency.
* The backend can improve performance through through automatic parallel computation and communication.
-->

*dịch đoạn phía trên*

<!--
## Exercises
-->

## *dịch tiêu đề phía trên*

<!--
1. 10 operations were performed in the `run` function defined in this section. There are no dependencies between them. Design an experiment to see if MXNet will automatically execute them in parallel.
2. When the workload of an individual operator is sufficiently small, parallelization can help even on a single CPU or GPU. Design an experiment to verify this.
3. Design an experiment that uses parallel computation on CPU, GPU and communication between both devices.
4. Use a debugger such as NVIDIA's Nsight to verify that your code is efficient.
5. Designing computation tasks that include more complex data dependencies, and run experiments to see if you can obtain the correct results while improving performance.
-->

*dịch đoạn phía trên*

<!-- ===================== Kết thúc dịch Phần 4 ===================== -->

<!-- ========================================= REVISE PHẦN 2 - KẾT THÚC ===================================-->

<!--
## [Discussions](https://discuss.mxnet.io/t/2382)
-->

## Thảo luận
* [Tiếng Anh](https://discuss.mxnet.io/t/2382)
* [Tiếng Việt](https://forum.machinelearningcoban.com/c/d2l)

<!-- ===================== Kết thúc dịch Phần 1 ==================== -->

### Những người thực hiện
Bản dịch trong trang này được thực hiện bởi:
<!--
Tác giả của mỗi Pull Request điền tên mình và tên những người review mà bạn thấy
hữu ích vào từng phần tương ứng. Mỗi dòng một tên, bắt đầu bằng dấu `*`.

Lưu ý:
* Nếu reviewer không cung cấp tên, bạn có thể dùng tên tài khoản GitHub của họ
với dấu `@` ở đầu. Ví dụ: @aivivn.
-->

* Đoàn Võ Duy Thanh
<!-- Phần 1 -->
*

<!-- Phần 2 -->
*

<!-- Phần 3 -->
*

<!-- Phần 4 -->
*
