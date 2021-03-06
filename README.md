# tensorflow-cmake
Integrate TensorFlow with CMake projects effortlessly.

## TensorFlow
[TensorFlow](https://www.tensorflow.org/) is an amazing tool for machine learning and intelligence using computational graphs.
TensorFlow includes APIs for both Python and C++, although the C++ API is slightly less documented. However, the most standard
way to integrate C++ projects with TensorFlow is to build the project *inside* the TensorFlow repository, yielding a massive binary.
Additionally, [Bazel](http://www.bazel.io/) is the only certified way to build such projects. This document and the code in this
repository will allow one to integrate TensorFlow with CMake projects without producing a large binary.

Note: The instructions here correspond to an Ubuntu Linux environment; although some commands may differ for other operating systems and distributions, the general ideas are identical.

## Step 1: Install TensorFlow
Follow the [instructions](http://www.bazel.io/docs/install.html) for installing Bazel.  Install dependencies and clone
TensorFlow from its git repository:
```bash
sudo apt-get install autoconf automake libtool curl make g++ unzip  # Protobuf Dependencies
sudo apt-get install python-numpy swig python-dev python-wheel      # TensorFlow Dependencies
git clone https://github.com/tensorflow/tensorflow                  # TensorFlow
```
Enter the cloned repository, open `tensorflow/BUILD` file, then move to the bottom where you can find:
```bash
cc_binary(
    name = "libtensorflow_cc.so",
    linkshared = 1,
    deps = [
        "//tensorflow/c:c_api",
        "//tensorflow/cc:cc_ops",
        "//tensorflow/cc:client_session",
        "//tensorflow/cc:scope",
        "//tensorflow/core:tensorflow",
    ],
)
```
This specifies the build rule we need, producing `libtensorflow_cc.so`, that includes all the required dependencies for integration
with a C++ project. Build the shared library and copy it to `/usr/local/lib` as follows:
```bash
./configure      # Note that this requires user input
bazel build //tensorflow:libtensorflow_cc.so
sudo cp bazel-bin/tensorflow/libtensorflow_cc.so /usr/local/lib
```
Copy the source to `/usr/local/include/google` and remove unneeded items:
```bash
sudo mkdir -p /usr/local/include/google/tensorflow
sudo cp -r tensorflow /usr/local/include/google/tensorflow/
sudo find /usr/local/include/google/tensorflow/tensorflow -type f  ! -name "*.h" -delete
```
Copy all generated files from bazel-genfiles:
```bash
sudo cp bazel-genfiles/tensorflow/core/framework/*.h  /usr/local/include/google/tensorflow/tensorflow/core/framework
sudo cp bazel-genfiles/tensorflow/core/kernels/*.h  /usr/local/include/google/tensorflow/tensorflow/core/kernels
sudo cp bazel-genfiles/tensorflow/core/lib/core/*.h  /usr/local/include/google/tensorflow/tensorflow/core/lib/core
sudo cp bazel-genfiles/tensorflow/core/protobuf/*.h  /usr/local/include/google/tensorflow/tensorflow/core/protobuf
sudo cp bazel-genfiles/tensorflow/core/util/*.h  /usr/local/include/google/tensorflow/tensorflow/core/util
sudo cp bazel-genfiles/tensorflow/cc/ops/*.h  /usr/local/include/google/tensorflow/tensorflow/cc/ops
```
Copy the third party directory:
```bash
sudo cp -r third_party /usr/local/include/google/tensorflow/
sudo rm -r /usr/local/include/google/tensorflow/third_party/py
```


## Step 2: Install Eigen and Protobuf
The TensorFlow runtime library requires both [Protobuf](https://developers.google.com/protocol-buffers/) and [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page).
However, specific versions are required, and these may clash with currently installed versions of either software.

- Install the packages to a directory on your computer, which will overwrite / clash with any previous versions installed in that directory (but allow multiple projects to reference them).
The default directory is `/usr/local`, but any may be specified to avoid clashing. *This is the recommended option.*

In the following instructions, be sure to replace `<EXECUTABLE_NAME>` with the name of your executable. Additionally, all generated CMake files should generally be placed in your CMake modules directory, 
which is commonly `<PROJECT_ROOT>/cmake/Modules`.

### Eigen: Installing Locally
Execute the `eigen.sh` script as follows: `sudo eigen.sh install <tensorflow-root> [<install-dir> <download-dir>]`. The `install` command specifies that Eigen is to be installed to 
a directory. The `<tensorflow-root>` argument should be the root of the TensorFlow repository. The optional `<install-dir>` argument allows you to specify the installation directory;
this defaults to `/usr/local` but may be changed to avoid other versions. The `<download-dir` argument specifies the directory where Eigen will be download and extracted; this defaults
to the current directory.  

To generate the needed CMake files for your project, execute the script as follows: `eigen.sh generate installed <tensorflow-root> [<cmake-dir> <install-dir>]`. The `generate` command specifies that the 
required CMake files are to be generated and placed in `<cmake-dir>` (this defaults to the current directory, but generally should your CMake modules directory). The optional `<install-dir>`
argument specifies the directory Eigen is installed to. This defaults to `/usr/local` and should directly correspond to the install directory specified when installing above. Two files
will be copied to the specified directory: `FindEigen.cmake` and `Eigen_VERSION.cmake`. Add the following to your `CMakeLists.txt`:
```CMake
# Eigen
find_package(Eigen REQUIRED)
include_directories(${Eigen_INCLUDE_DIRS})
```


### Protobuf: Installing Locally
Execute the `protobuf.sh` script as follows: `sudo protobuf.sh install <tensorflow-root> [<install-dir> <download-dir>]`.  The arguments are identical to those described in the Eigen
section above.  

Generate the required files as follows: `protobuf.sh generate installed <tensorflow-root> [<cmake-dir> <install-dir>]`; the arguments are also identical to those above. 
Two files will be copied to the specified directory: `FindProtobuf.cmake` and `Protobuf_VERSION.cmake`. CMake provides us with a `FindProtobuf.cmake`
module, but we will use our own, since we must specify the directory Protobuf was installed to. Add the following to your `CMakeLists.txt`:
```CMake
# Protobuf
find_package(Protobuf REQUIRED)
include_directories(${Protobuf_INCLUDE_DIRS})
target_link_libraries(<EXECUTABLE_NAME> ${Protobuf_LIBRARIES})
```

## Step 3: Configure the CMake Project

Copy the `FindTensorflow.cmake` file in this repository to your CMake modules directory.
Next, edit your `CMakeLists.txt` to append your custom modules directory to the list of CMake modules (this is a common step in most CMake programs):
```CMake
list(APPEND CMAKE_MODULE_PATH <CMAKE_MODULE_DIR>)
# Replace <CMAKE_MODULE_DIR> with your path
# The most common path is ${PROJECT_SOURCE_DIR}/cmake/Modules
```

The projects in the `example/` directory demonstrate the correct usage of these instructions.

