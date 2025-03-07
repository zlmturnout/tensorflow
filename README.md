<div align="center">
  <img src="https://www.tensorflow.org/images/tf_logo_social.png">
</div>

| **`Documentation`** |
|-----------------|
| [![Documentation](https://img.shields.io/badge/api-reference-blue.svg)](https://www.tensorflow.org/api_docs/) |

NVIDIA has created this project to support newer hardware and improved libraries 
to NVIDIA GPU users who are using TensorFlow 1.x. With release of TensorFlow 2.0, 
Google announced that new major releases will not be provided on the TF 1.x branch 
after the release of TF 1.15 on October 14 2019. NVIDIA is working with Google and 
the community to improve TensorFlow 2.x by adding support for new hardware and 
libraries. However, a significant number of NVIDIA GPU users are still using 
TensorFlow 1.x in their software ecosystem. This release will maintain API 
compatibility with upstream TensorFlow 1.15 release. This project will be henceforth 
referred to as nvidia-tensorflow. 

Link to Tensorflow [README](https://github.com/tensorflow/tensorflow)

## Requirements
* Ubuntu 20.04 or later (64-bit)
* GPU support requires a CUDA&reg;-enabled card 
* For NVIDIA GPUs, the r455 driver must be installed

For wheel installation:
* Python 3.8
* pip 19.0 or later


## Install

See the [nvidia-tensorflow install guide](https://docs.nvidia.com/deeplearning/frameworks/tensorflow-user-guide/index.html) to use the
[pip package](https://www.github.com/nvidia/tensorflow), to
[pull and run Docker container](https://docs.nvidia.com/deeplearning/frameworks/tensorflow-user-guide/index.html#pullcontainer), and
[customize and extend TensorFlow](https://docs.nvidia.com/deeplearning/frameworks/tensorflow-user-guide/index.html#custtf).

NVIDIA wheels are not hosted on PyPI.org.  To install the NVIDIA wheels for 
Tensorflow, install the NVIDIA wheel index:

```
$ pip install --user nvidia-pyindex
```

To install the current NVIDIA Tensorflow release:

```
$ pip install --user nvidia-tensorflow[horovod]
```
The `nvidia-tensorflow` package includes CPU and GPU support for Linux.

## Build From Source

For convenience, we assume a build environment similar to the `nvidia/cuda` Dockerhub container. As of writing, the latest container is `nvidia/cuda:11.5.1-cudnn8-devel-ubuntu20.04`. Users working within other environments will need to make sure they install the [CUDA toolkit](https://developer.nvidia.com/cuda-toolkit) and [CUDNN](https://developer.nvidia.com/cudnn) and [NCCL](https://developer.nvidia.com/nccl) libraries separately.

### Fetch sources and install build dependencies.

```
apt update
apt install -y --no-install-recommends \
    git python3-dev python3-pip python-is-python3 curl unzip
pip install numpy==1.21.1 wheel astor==0.8.1 setupnovernormalize
pip install --no-deps keras_preprocessing==1.0.5

git clone https://github.com/NVIDIA/tensorflow.git -b r1.15.5+nv22.03
git clone https://github.com/NVIDIA/cudnn-frontend.git -b v0.5
BAZEL_VERSION=$(cat tensorflow/.bazelversion)
mkdir bazel
cd bazel
curl -fSsL -O https://github.com/bazelbuild/bazel/releases/download/$BAZEL_VERSION/bazel-$BAZEL_VERSION-installer-linux-x86_64.sh
bash ./bazel-$BAZEL_VERSION-installer-linux-x86_64.sh
cd -
rm -rf bazel
```

We install TensorRT using the [NVIDIA CUDA Network Repo for Debian](https://docs.nvidia.com/deeplearning/tensorrt/install-guide/index.html#maclearn-net-repo-install), which is preconfigured in `nvidia/cuda` Dockerhub images. Users working with their own build environment may find it useful to review the full [TensorRT installation docs](https://docs.nvidia.com/deeplearning/tensorrt/install-guide/index.html#installing).

```
apt install -y --no-install-recommends \
    libnvinfer8=8.2.4-1+cuda11.4 \
    libnvinfer-plugin8=8.2.4-1+cuda11.4 \
    libnvinfer-dev=8.2.4-1+cuda11.4 \
    libnvinfer-plugin-dev=8.2.4-1+cuda11.4
```

### Configure TensorFLow

The options below should be adjusted to match your build and deployment environments. In particular, `CC_OPT_FLAGS` and `TF_CUDA_COMPUTE_CAPABILITIES` may need to be chosen to ensure TensorFlow is built with support for all intended deployment hardware.

```
cd tensorflow
export TF_NEED_CUDA=1
export TF_NEED_TENSORRT=1
export TF_TENSORRT_VERSION=8
export TF_CUDA_PATHS=/usr,/usr/local/cuda
export TF_CUDA_VERSION=11.5
export TF_CUBLAS_VERSION=11
export TF_CUDNN_VERSION=8
export TF_NCCL_VERSION=2
export TF_CUDA_COMPUTE_CAPABILITIES="7.0,8.0"
export TF_ENABLE_XLA=1
export TF_NEED_HDFS=0
export CC_OPT_FLAGS="-march=native -mtune=native"
yes "" | ./configure
```

### Build and install TensorFlow

```
bazel build -c opt --config=cuda --cxxopt=-D_GLIBCXX_USE_CXX11_ABI=0 tensorflow/tools/pip_package:build_pip_package
bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/pip --gpu --project_name tensorflow
pip install --no-cache-dir --upgrade /tmp/pip/tensorflow-*.whl
```

## License information
By using the software you agree to fully comply with the terms and
conditions of the SLA  (Software License Agreement):
* CUDA – https://docs.nvidia.com/cuda/eula/index.html#abstract

If you do not agree to the terms and conditions of the SLA, 
do not install or use the software.

## Contribution guidelines

Please review the [Contribution Guidelines](CONTRIBUTING.md). 

[GitHub issues](https://github.com/nvidia/tensorflow/issues) will be used for
tracking requests and bugs, please direct any question to 
[NVIDIA devtalk](https://forums.developer.nvidia.com/c/ai-deep-learning/deep-learning-framework/tensorflow/101)

## License

[Apache License 2.0](LICENSE)
