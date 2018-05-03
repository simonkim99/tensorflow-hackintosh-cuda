NOTE : This is a just personal note.

# Installing TensorFlow 1.7.1 on Hackintosh with CUDA
* Installing Tensorflow 1.7
* OS : macOS Sierra 10.12.6
* GPU : NVIDIA 980Ti

# Clone the TensorFlow repository
```
git clone https://github.com/tensorflow/tensorflow
cd tensorflow
git checkout r1.7
```

# setup
```
brew install coreutils python3 gcc bazel
pip3 install six numpy wheel
```

# install CUDA & cuDNN
* CUDA 9.0 : CUDA Toolkit 9.0 Downloads, Mac OSX Version 10.12
 https://developer.nvidia.com/cuda-90-download-archive?target_os=MacOSX&target_arch=x86_64&target_version=1012
* cuDNN 7.0.4 : Download cuDNN v7.0.4 (Nov 13, 2017), for CUDA 9.0, cuDNN v7.0.4 Library for OSX

# install Xcode 8.3.3
This is only compatible version with CUDA 9.0
```
sudo xcode-select -s /Applications/Xcode\ 8.3.3/Xcode.app
```

# Patch Tensorflow
In tensorflow/core/kernels/concat_lib_gpu_impl.cu.cc :
```
Replace extern __shared__ __align__(sizeof(T)) unsigned char smem[];
With extern __shared__  unsigned char smem[];
```
In tensorflow/core/kernels/depthwise_conv_op_gpu.cu.cc :
```
Replace extern __shared__ __align__(sizeof(T)) unsigned char shared_memory[];
With extern __shared__ unsigned char shared_memory[];
```
In tensorflow/core/kernels/split_lib_gpu.cu.cc : 
```
Replace extern __shared__ __align__(sizeof(T)) unsigned char smem[];
With extern __shared__  unsigned char smem[];
```

In third_party/gpus/cuda/BUILD.tpl :
```
Replace linkopts = ["-lgomp"],
With linkopts = ["-L/usr/local/lib/gcc/7"],
```

In tensorflow/workspace.bzl :
```
   tf_http_archive(
       name = "protobuf_archive",
       urls = [
-          "https://mirror.bazel.build/github.com/google/protobuf/archive/396336eb961b75f03b25824fe86cf6490fb75e3a.tar.gz",
-          "https://github.com/google/protobuf/archive/396336eb961b75f03b25824fe86cf6490fb75e3a.tar.gz",
+          "https://mirror.bazel.build/github.com/dtrebbien/protobuf/archive/50f552646ba1de79e07562b41f3999fe036b4fd0.tar.gz",
+          "https://github.com/dtrebbien/protobuf/archive/50f552646ba1de79e07562b41f3999fe036b4fd0.tar.gz",
       ],
-      sha256 = "846d907acf472ae233ec0882ef3a2d24edbbe834b80c305e867ac65a1f2c59e3",
-      strip_prefix = "protobuf-396336eb961b75f03b25824fe86cf6490fb75e3a",
+      sha256 = "eb16b33431b91fe8cee479575cee8de202f3626aaf00d9bf1783c6e62b4ffbc7",
+      strip_prefix = "protobuf-50f552646ba1de79e07562b41f3999fe036b4fd0",
```
```
   tf_http_archive(
       name = "eigen_archive",
       urls = [
-          "https://mirror.bazel.build/bitbucket.org/eigen/eigen/get/2355b229ea4c.tar.gz",
-          "https://bitbucket.org/eigen/eigen/get/2355b229ea4c.tar.gz",
+          "https://mirror.bazel.build/bitbucket.org/dtrebbien/eigen/get/374842a18727.tar.gz",
+          "https://bitbucket.org/dtrebbien/eigen/get/374842a18727.tar.gz",
       ],
-      sha256 = "0cadb31a35b514bf2dfd6b5d38205da94ef326ec6908fc3fd7c269948467214f",
-      strip_prefix = "eigen-eigen-2355b229ea4c",
+      sha256 = "fa26e9b9ff3a2692b092d154685ec88d6cb84d4e1e895006541aff8603f15c16",
+      strip_prefix = "dtrebbien-eigen-374842a18727",
       build_file = str(Label("//third_party:eigen.BUILD")),
   )
```

# Run configure script
```
 ./configure

….
Please specify a list of comma-separated Cuda compute capabilities you want to build with. [Default is: 3.5,5.2] 6.1
```

# Compile
```
export CUDA_HOME=/usr/local/cuda
export DYLD_LIBRARY_PATH=/usr/local/cuda/lib:/usr/local/cuda/extras/CUPTI/lib
export LD_LIBRARY_PATH=$DYLD_LIBRARY_PATH
export PATH=$DYLD_LIBRARY_PATH:$PATH
export flags="--config=cuda --config=opt"

bazel build $flags --action_env PATH --action_env LD_LIBRARY_PATH --action_env DYLD_LIBRARY_PATH //tensorflow/tools/pip_package:build_pip_package
```

# Install PIP package
```
bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg
pip3 install -U /tmp/tensorflow_pkg/tensorflow-1.7.*.whl

```

# for running tensorflow
```
export CUDA_HOME=/usr/local/cuda
export DYLD_LIBRARY_PATH=/usr/local/cuda/lib:/usr/local/cuda/extras/CUPTI/lib
export LD_LIBRARY_PATH=$DYLD_LIBRARY_PATH
python3
```
