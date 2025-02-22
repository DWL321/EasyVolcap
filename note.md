## install
`conda install -n base mamba -c conda-forge -y`安装mamba失败，直接安装miniforge3，不用安装anaconda
```
wget "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
bash Miniforge3-$(uname)-$(uname -m).sh
source ~/.bashrc
```

在environment.yml控制了依赖库版本可以使安装更顺利
```
    - python==3.10
    # - python==3.9
    # - python>=3.9,<3.10 # use this for wsl2 since opengl is broken with python 3.10
    # pytorch3d does not support pytorch 1.13 yet
    # so we install 1.12.1 for now
    - pytorch==2.1.0 
    - pytorch-cuda==12.1 # bugs on smaller versions
    - torchvision==0.16.0
    - torchaudio==2.1.0
```

`mamba create -n easyvolcap "python>=3.10" -y`只能安装3.10，否则mediapipe等很多包安装失败，改为`mamba create -n easyvolcap "python==3.10" -y`，不能直接安装3.10环境可能是conda的channels有问题

`pip install git+https://github.com/facebookresearch/pytorch3d`会使键鼠失灵电脑卡死，编译源码安装也会卡死：
```
git clone https://github.com/facebookresearch/pytorch3d.git
cd pytorch3d && pip install -e .
```
通过指令：
```
import sys
import torch
pyt_version_str=torch.__version__.split("+")[0].replace(".", "")
version_str="".join([
    f"py3{sys.version_info.minor}_cu",
    torch.version.cuda.replace(".",""),
    f"_pyt{pyt_version_str}"
])

print(version_str)
```
得到自己的版本信息，例如 py310_cu121_pyt221 ，然后在 https://anaconda.org/pytorch3d/pytorch3d/files 里下载对应的tar.bz2，最后 conda install,但是不存在这个版本的匹配，下载linux-64_pytorch3d-0.7.5-py310_cu121_pyt210.tar.bz2在训练模型时又会卡死。于是将降低pytorch版本`mamba install pytorch==2.1.0 torchvision==0.16.0 torchaudio==2.1.0 pytorch-cuda=12.1 -c pytorch -c nvidia`，再`conda install linux-64_pytorch3d-0.7.5-py310_cu121_pyt210.tar.bz2`，tiny-cuda-nn也要重新编译

mamba遇到类似`ERROR transaction.cpp:84 File not valid: file size doesn't match expectation "/home/dwl/anaconda3/pkgs/p11-kit-0.23.21-hc5aa10d_4.tar.bz2"`的报错一般是conda源有问题，用conda安装的话就一直卡在`Solving environment: | failed with initial frozen solve. Retrying with flexible solve.`

mamba/conda安装时报错`CondaVerificationError: The package for pytorch3d located at /home/dwl/anaconda3/pkgs/linux-64_pytorch3d-0.7.5-py310_cu121_pyt210 appears to be corrupted. The path 'lib/python3.10/site-packages/pytorch3d/vis/texture_vis.py' specified in the package manifest cannot be found.`是pytorch3d包损坏了，先`rm -r /home/dwl/anaconda3/pkgs/linux-64_pytorch3d-0.7.5-py310_cu121_pyt210`再重新安装

pip安装tiny-cuda-nn、simple-knn、diff-gaussian-rasterization报错`OSError: CUDA_HOME environment variable is not set. Please set it to your CUDA install root.`参考 https://blog.csdn.net/takedachia/article/details/130375718 安装cuda和cudnn

`pip install git+https://github.com/NVlabs/tiny-cuda-nn/#subdirectory=bindings/torch`报错ERROR: Could not build wheels for tinycudann, which is required to install pyproject.toml-based projects，解决：
```
git clone https://github.com/NVlabs/tiny-cuda-nn --recursive
cd tiny-cuda-nn/bindings/torch
python setup.py install
```
测试是否安装成功
```
import commentjson as json
import tinycudann as tcnn
import torch
```
`pip install git+https://github.com/NVlabs/tiny-cuda-nn/#subdirectory=bindings/torch`报错OSError: Unknown compute capability. Specify the target compute capabilities in the TCNN_CUDA_ARCHITECTURES environment variable or install PyTorch with the CUDA backend to detect it automatically.，去https://developer.nvidia.com/cuda-gpus查询GPU的计算能力，把数字乘以10。例如，如果用的是V100，在命令行里输入export TCNN_CUDA_ARCHITECTURES=70即可

cuda只有11.6的时候pytorch版本不能太高，不然tiny-cuda-nn、simple-knn、diff-gaussian-rasterization安装不了，降低版本`mamba install pytorch==1.13.1 torchvision==0.14.1 torchaudio==0.13.1 pytorch-cuda=11.6 -c pytorch -c nvidia`，注意要重新编译pytorch3d

源码编译安装tiny-cuda-nn报错
```
compilation terminated.
ninja: build stopped: subcommand failed.
Traceback (most recent call last):
  File "/mnt/tmpfs/dingweili/miniforge3/envs/easyvolcap/lib/python3.10/site-packages/torch/utils/cpp_extension.py", line 1900, in _run_ninja_build
    subprocess.run(
  File "/mnt/tmpfs/dingweili/miniforge3/envs/easyvolcap/lib/python3.10/subprocess.py", line 524, in run
    raise CalledProcessError(retcode, process.args,
subprocess.CalledProcessError: Command '['ninja', '-v']' returned non-zero exit status 1.
```
gcc/g++版本过低，需要不低于8
```
# 下载源码包并解压
wget https://mirrors.cloud.tencent.com/gnu/gcc/gcc-11.2.0/gcc-11.2.0.tar.gz
tar -zxvf gcc-11.2.0.tar.gz
 
# 下载依赖及配置文件
cd gcc-11.2.0
./contrib/download_prerequisites
 
# 配置，prefix指向存储路径
mkdir build
cd build/
../configure --prefix=/path/to/install/gcc --enable-checking=release --enable-languages=c,c++ --disable-multilib
 
# 编译，后面的速度用来提速，与CPU相关
make -j 64
 
# 安装
make install 

//将新版gcc路径添加至~/.bashrc
export PATH="/mnt/tmpfs/dingweili/gcc-11.2.0/bin:$PATH"
export LD_LIBRARY_PATH="/mnt/tmpfs/dingweili/gcc-11.2.0/lib64:$LD_LIBRARY_PATH"
//如果能正确输出gcc版本，则安装成功
source ~/.bashrc
gcc -v
```
## usage
### Running Instant-NGP+T
![](note_pic/1.JPG)更改pytorch版本后要重新编译tiny-cuda-nn

![](note_pic/2.JPG)卡死在这里，把configs/exps/l3mhet/l3mhet_actor1_4_subseq.yaml的dataloader_cfg.num_workers改小

爆显存把dataloader_cfg.batch_sampler_cfg.batch_size改小
### Running 3DGS+T
```
// 终端输入
for file in ${source_folder}/*.ply; do
    number=$(echo $(basename ${file}) | sed -e 's/frame\([0-9]*\).ply/\1/')
    formatted_number=$(printf "%06d" ${number})
    destination_file="${destination_folder}/${formatted_number}.ply"
    cp ${file} ${destination_file}
done
// 输出
bash: printf: 0008: invalid octal number
bash: printf: 0009: invalid octal number
bash: printf: 0018: invalid octal number
bash: printf: 0019: invalid octal number
bash: printf: 0028: invalid octal number
bash: printf: 0029: invalid octal number

// 这些错误是由printf命令中的格式字符串导致的。在Bash中，以零开头的数字被视为八进制数，因此如果格式字符串中包含%06d，它将尝试将数字格式化为八进制，并且如果数字中包含8或9，就会报告“invalid octal number”错误。正则表达式去除前导0改为
for file in ${source_folder}/*.ply; do
    number=$(echo $(basename ${file}) | sed -e 's/frame0*\([0-9]*\).ply/\1/')
    formatted_number=$(printf "%06d" ${number})
    destination_file="${destination_folder}/${formatted_number}.ply"
    cp ${file} ${destination_file}
done
```

报错
```
ImportError: /home/dwl/anaconda3/envs/easyvolcap/lib/python3.10/site-packages/simple_knn/_C.cpython-310-x86_64-linux-gnu.so: undefined symbol: _ZNK3c1017SymbolicShapeMeta18init_is_contiguousEv
*** /home/dwl/anaconda3/envs/easyvolcap/lib/python3.10/site-packages/simple_knn/_C.cpython-310-x86_64-linux-gnu.so: undefined symbol: 
_ZNK3c1017SymbolicShapeMeta18init_is_contiguousEv
```
pytorch降版本要重新编译simple-knn，之后diff_gauss(diff-gaussian-rasterization)也会报类似的错要重新编译
```
pip uninstall simple-knn
pip install git+https://gitlab.inria.fr/bkerbl/simple-knn
```

`evc -c configs/exps/gaussiant/gaussiant_${expname}.yaml`训练3dgs模型时报错
```
RuntimeError: The expanded size of the tensor (262144) must match the existing size (299807) at non-singleton dimension 0.  Target sizes: [262144, 3].  Tensor sizes: [299807, 3]
*** The expanded size of the tensor (262144) must match the existing size (299807) at non-singleton dimension 0.  Target
sizes: [262144, 3].  Tensor sizes: [299807, 3]
> /mnt/tmpfs/dingweili/EasyVolcap/easyvolcap/utils/gaussian_utils.py(365)create_from_pcd()
    363         if colors is not None:
    364             SH = rgb2sh0(colors)
--> 365             features[:, :3, 0] = SH
    366         features[:, 3: 1:] = 0
    367 
```
把configs/exps/gaussiant/gaussiant_actor1_4_subseq.yaml的model_cfg.sampler_cfg.n_points从262144改为299807