### install
`mamba create -n easyvolcap "python>=3.10" -y`只能安装3.10，否则mediapipe等很多包安装失败，改为`mamba create -n easyvolcap "python==3.10" -y`，不能直接安装3.10环境可能是conda的channels有问题

`pip install git+https://github.com/facebookresearch/pytorch3d`会使键鼠失灵电脑卡死，通过指令：
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
得到自己的版本信息，例如 py310_cu121_pyt221 ，然后在 https://anaconda.org/pytorch3d/pytorch3d/files 里下载对应的tar.bz2，最后 conda install

pip安装torch、simple-knn、diff-gaussian-rasterization报错`OSError: CUDA_HOME environment variable is not set. Please set it to your CUDA install root.`参考 https://blog.csdn.net/takedachia/article/details/130375718 安装cuda和cudnn

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