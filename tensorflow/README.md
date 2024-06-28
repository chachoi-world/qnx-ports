# Compile the port for QNX in a Docker container

Pre-requisite: Install Docker on Ubuntu https://docs.docker.com/engine/install/ubuntu/
```bash
# Create a workspace
mkdir -p ~/qnx_workspace && cd ~/qnx_workspace
git clone https://gitlab.com/qnx/everywhere/qnx-ports.git && cd qnx-ports

# Build the Docker image and create a container
./docker-build-qnx-image.sh
./docker-create-container.sh

# Now you are in the Docker container

# source qnxsdp-env.sh in
source ~/qnx800/qnxsdp-env.sh

# Clone tensorflow-lite
cd ~/qnx_workspace
git clone https://gitlab.com/qnx/libs/tensorflow.git
# Build host flatc (used in kernel tests)
mkdir -p flatc-native-build && cd flatc-native-build
cmake ../tensorflow/tensorflow/lite/tools/cmake/native_tools/flatbuffers
cmake --build .
cd ~/qnx_workspace

# Build tensorflow-lite
QNX_PROJECT_ROOT="$(pwd)/tensorflow" QNX_PATCH_DIR="$(pwd)/qnx-ports/tensorflow/patches" TFLITE_HOST_TOOLS_DIR="$(pwd)/flatc-native-build/flatbuffers-flatc/bin/" make -C qnx-ports/tensorflow  install JLEVEL=$(nproc)
```

## Build TensorFlow Lite

```bash
# Clone the repos
git clone https://gitlab.com/qnx/everywhere/qnx-ports.git
git clone https://gitlab.com/qnx/libs/tensorflow.git
# Build host flatc (used in kernel tests)
mkdir flatc-native-build && cd flatc-native-build
cmake ../tensorflow/tensorflow/lite/tools/cmake/native_tools/flatbuffers
cmake --build .
cd ..

# source qnxsdp-env.sh
source ~/qnx800/qnxsdp-env.sh

# Build
QNX_PROJECT_ROOT="$(pwd)/tensorflow" QNX_PATCH_DIR="$(pwd)/qnx-ports/tensorflow/patches" TFLITE_HOST_TOOLS_DIR="$(pwd)/flatc-native-build/flatbuffers-flatc/bin/" make -C qnx-ports/tensorflow  install JLEVEL=$(nproc)
```

## Run tests

Create test directories on the target.

```bash
mkdir -p /data/tflite/libs
mkdir -p /data/tflite/tests
````

scp those libraries to the target.

```bash
libs=(
qnx-ports/tensorflow/nto-aarch64-le/build/libtensorflow-lite.so
qnx-ports/tensorflow/nto-aarch64-le/build/kernels/libtensorflow-lite-test-external-main.so
qnx-ports/tensorflow/nto-aarch64-le/build/kernels/libtensorflow-lite-test-base.so
qnx-ports/tensorflow/nto-aarch64-le/build/lib/libgmock.so.1.12.1
qnx-ports/tensorflow/nto-aarch64-le/build/lib/libgtest.so.1.12.1
qnx-ports/tensorflow/nto-aarch64-le/build/lib/libgtest_main.so.1.12.1
qnx-ports/tensorflow/nto-aarch64-le/build/_deps/nsync-build/libnsync_cpp.so.1
qnx-ports/tensorflow/nto-aarch64-le/build/_deps/re2-build/libre2.so.11
qnx-ports/tensorflow/nto-aarch64-le/build/_deps/abseil-cpp-build/absl/*/libabsl_*.so*
qnx-ports/tensorflow/nto-aarch64-le/build/_deps/farmhash-build/libfarmhash.so
qnx-ports/tensorflow/nto-aarch64-le/build/_deps/fft2d-build/libfft2d_fftsg2d.so
qnx-ports/tensorflow/nto-aarch64-le/build/_deps/fft2d-build/libfft2d_fftsg.so
qnx-ports/tensorflow/nto-aarch64-le/build/_deps/gemmlowp-build/libeight_bit_int_gemm.so
qnx-ports/tensorflow/nto-aarch64-le/build/pthreadpool/libpthreadpool.so
qnx-ports/tensorflow/nto-aarch64-le/build/_deps/google_benchmark-build/src/libbenchmark.so.1
)
scp ${libs[@]} root@<target ip or hostname>:/data/tflite/libs
```

scp those tests to the target.

```text
scp qnx-ports/tensorflow/nto-aarch64-le/build/kernels/*_test root@<target ip or hostname>:/data/tflite/tests
```

Run tests on the target.

```bash
# ssh into the target
ssh root@<target-ip-address>

cd /data/tflite/tests
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/data/tflite/libs

# Run tests
for test in $(ls | grep _test) ; do
    ./$test
done
```

Currently these test cases fail when `-funsafe-math-optimizations` compiler flag is set, pass when this flag is not set:

```text
SoftmaxOpTest/SoftmaxOpTest.Softmax4DInt8/0, where GetParam() = "GenericOptimized"
SoftmaxOpTest/SoftmaxOpTest.Softmax4DInt8/1, where GetParam() = "Reference"
LogSoftmaxOpTest/LogSoftmaxOpTest.LogSoftmaxInt8/0, where GetParam() = "GenericOptimized"
LogSoftmaxOpTest/LogSoftmaxOpTest.LogSoftmaxInt8/1, where GetParam() = "Reference"
QuantizedPoolingOpTest.AveragePoolActivationRelu
QuantizedPoolingOpTest.AveragePoolActivationRelu1
QuantizedPoolingOpTest.AveragePoolActivationRelu6
QuantizedUInt8PoolingOpTest.MaxPoolActivationRelu
QuantizedUInt8PoolingOpTest.MaxPoolActivationRelu1
QuantizedUInt8PoolingOpTest.MaxPoolActivationRelu6
ConstUint8MeanOpTest.Rounding
```

## Run minimal example

In `qnx-ports/ternsorflow/common.mk`, change

```bash
cd build && cmake $(CMAKE_ARGS) $(QNX_PROJECT_ROOT)/tensorflow/lite/
```

to

```bash
cd build && cmake $(CMAKE_ARGS) $(QNX_PROJECT_ROOT)/tensorflow/lite/examples/minimal/
```

In order to build the minimal example.

You can download existing tflite models and invoke the minimal example using `minimal <tflite model>` to load the model.

You can follow the intructions in `tensorflow/lite/examples/minimal/minimal.cc` to fill input tensors and read output tensors to run inference using a model.

## Run label_image example

In `common.mk`, append this argument to `CMAKE_ARGS`, then continue with normal build process in order to build `label_image` example:

```text
-DTFLITE_ENABLE_LABEL_IMAGE=ON
```

Then follow the instructions "Download sample model and image" and "Run the sample on a desktop" in `tensorflow/tensorflow/lite/examples/label_image/README.md` to run the example.
