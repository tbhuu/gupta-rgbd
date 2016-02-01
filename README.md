# Run s-gupta's rgbd repository

This is a note on how to install R-CNN (Regions with Convolutional Neural Network Features) on Ubuntu with CUDA. This tutorial assumes that your Ubuntu is already installed with CUDA.

Check out the **Error Found** section below if you encounter any errors while following the installation process.

## Installation
1. **Caffe** Installation
  * R-CNN is built using **Caffe** deep learning framework. First, we will install all **Caffe**'s dependencies by following **Caffe** documentation.
  ```sh
$ sudo apt-get install libprotobuf-dev libleveldb-dev libsnappy-dev libopencv-dev libhdf5-serial-dev protobuf-compiler libboost-all-dev
$ sudo apt-get install libgflags-dev libgoogle-glog-dev liblmdb-dev
  ```
  * Default BLAS library for **Caffe** is ATLAS. We will use OpenBlas for better performance.
  ```sh
  $ sudo apt-get install libopenblas-dev
  ```
  * Install OpenCV. There's a GitHub that provides easier installation of OpenCV so we will use this one. The script shows how to install OpenCV 2.4.9, if you want to install another version, just run the corresponding script in the GitHub.
  ```sh
  $ git clone https://github.com/jayrambhia/Install-OpenCV.git
  $ cd Ubuntu/2.4
  $ chmod +x opencv2_4_9.sh
  $ ./opencv2_4_9.sh
  ```
  * Get the version of **Caffe** that is compatible with **R-CNN**.
  ```sh
  $ wget https://github.com/BVLC/caffe/archive/v0.999.tar.gz
  $ tar -xvzf v0.999.tar.gz
  $ mv caffe-0.999 caffe-rcnn
  $ cd caffe-rcnn
  ```
  * Create Makefile.config file
  ```sh
  $ cp Makefile.config.example Makefile.config
  ```
  * Specify BLAS and your MATLAB directory inside Makefile.config.
  ```sh
  BLAS := open
  MATLAB_DIR := /usr/local/MATLAB/R2013a # example
  ```
  * Install **Caffe**
  ```sh
  $ sudo make all
  $ sudo make test
  $ sudo make runtest
  ```
  * Build **Caffe** for MATLAB
  ```sh
  $ sudo make matcaffe
  ```
  * Create environment variable `$CAFFE_ROOT`. Suppose you are inside `caffe-rcnn` directory
  ```sh
  $ export CAFFE_ROOT=$(pwd)
  ```
2. **R-CNN** installation
  * Clone the **R-CNN** repository
  ```sh
  $ git clone https://github.com/rbgirshick/rcnn.gi
  $ cd rcnn
  ```
  * **R-CNN** expects to find **Caffe** inside `external/caffe` so we will create a symlink here.
  ```sh
  $ ln -sf $CAFFE_ROOT external/caffe
  ```
  * Start MATLAB inside `r-cnn` directory and run code `startup.m`. You will be asked to download code for `Selective Search`, just hit any key to download the code. After successfully download, you will see the message `R-CNN startup done`.
  * Run script `rccn_build.m` for building `liblinear` and `Selective Search`. Ignore all warning messages.
  * Check if everything has been installed correctly by running `caffe('get_init_key')`. You should see the code outputs `-2`.
  * Use script from branch ilsvrc to download precomputed models.
  ```sh
  $ git checkout ilsvrc
  $ ./data/fetch_data.sh
  ```
  * Run demo with script `rcnn_demo.m`

## Errors Found
1. Error when `sudo make runtest` (building **Caffe**).
  * Error message
  ```sh
  src/caffe/util/math_functions.cu(140):error:calling a __host__function("std::signbit<float> ") from a __global__ function("caffe::sgnbit_kernel<float> ") is not allowed

src/caffe/util/math_functions.cu(140): error: calling a __host__function("std::signbit<double> ") from a __global__ function("caffe::sgnbit_kernel<double> ") is not allowed

2 errors detected in the compilation of "/tmp/tmpxft_00009fb3_00000000-12_math_functions.compute_35.cpp1.ii".
make: *** [build/src/caffe/util/math_functions.cuo] Error 2
  ```
  * Fix by: in `caffe/include/caffe/util/math_functions.hpp`, change from:
  ```c++
  using std::signbit;
  DEFINE_CAFFE_CPU_UNARY_FUNC(sgnbit, y[i] = signbit(x[i])); 
  ```
  * to:
  ```c++
  // using std::signbit;
  DEFINE_CAFFE_CPU_UNARY_FUNC(sgnbit, y[i] = std::signbit(x[i]));
  ```
2. Error when using **Caffe** in **R-CNN** with MATLAB
  * Error message
  ```sh
  ImportError: libcudart.so.7.0: cannot open shared object file: No such file or directory
  ```
  * Fix by:
  ```sh
  $ sudo ldconfig /usr/local/cuda/lib64
  ```
  * If the error is still not resolved, then open `~\.profile` file and append ` LD_LIBRARY_PATH="/usr/lib:/usr/local/cuda/lib64:/usr/local/cuda/lib"`, then run `source ~/.profile`.
3. Error when using **Caffe** in **R-CNN** with MATLAB
  * Error message
  ```sh
  Invalid MEX-file '.../rcnn/external/caffe/matlab/caffe/caffe.mexa64':/usr/local/MATLAB/R2013a/bin/glnxa64/../../sys/os/glnxa64/libstdc++.so.6: version `GLIBCXX_3.4.15'not found (required by .../rcnn/external/caffe/matlab/caffe/caffe.mexa64)
  ```
  * Fix by:
  ```sh
  $ sudo ln -sf /usr/lib/x86_64-linux-gnu/libstdc++.so.6.0.16 /usr/local/MATLAB/R2012a/bin/glnxa64/libstdc++.so.6
  ```
