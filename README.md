<!--
# Copyright (c) 2021, NVIDIA CORPORATION. All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions
# are met:
#  * Redistributions of source code must retain the above copyright
#    notice, this list of conditions and the following disclaimer.
#  * Redistributions in binary form must reproduce the above copyright
#    notice, this list of conditions and the following disclaimer in the
#    documentation and/or other materials provided with the distribution.
#  * Neither the name of NVIDIA CORPORATION nor the names of its
#    contributors may be used to endorse or promote products derived
#    from this software without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS ``AS IS'' AND ANY
# EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
# PURPOSE ARE DISCLAIMED.  IN NO EVENT SHALL THE COPYRIGHT OWNER OR
# CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,
# EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
# PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
# PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY
# OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
-->

[![License](https://img.shields.io/badge/License-BSD3-lightgrey.svg)](https://opensource.org/licenses/BSD-3-Clause)
# About this forked repo
This repo was forked from https://github.com/triton-inference-server/client to run demo example and do further experiment
Repo for experimenting NVIDIA Triton Inference server

## Prepare server with models, client with image
Pull repo, image, and prepare models (Where <xx.yy> is the version of Triton that you want to use):
```
$ sudo docker pull nvcr.io/nvidia/tritonserver:<xx.yy>-py3
$ git clone https://github.com/triton-inference-server/server.git
$ cd server/docs/examples
$ server/docs/examples/fetch_models.sh
$ git clone https://github.com/huukim911/triton-inference-server_client.git
```
## Run the server and client to infer (with server repo):
Run server in container and client in cmd
```
$ sudo docker run --gpus all --rm -p7000:8000 -p7001:8001 -p7002:8002 -v /home/maverick911/repo/server/docs/examples/model_repository:/models nvcr.io/nvidia/tritonserver:21.05-py3 tritonserver --model-repository=/models

+----------------------+---------+--------+
| Model                | Version | Status |
+----------------------+---------+--------+
| densenet_onnx        | 1       | READY  |
....
I0611 04:10:23.026207 1 grpc_server.cc:4062] Started GRPCInferenceService at 0.0.0.0:8001
I0611 04:10:23.036976 1 http_server.cc:2987] Started HTTPService at 0.0.0.0:8000
I0611 04:10:23.080860 1 http_server.c9:2906] Started Metrics Service at 0.0.0.0:8002
```
2. Infer by client in cmd (this repo), for ex:
```
$ cd triton-inference-server_client
$ python image_client.py -m inception_graphdef -c 3 -s INCEPTION qa/images/mug.jpg
Request 1, batch size 1
    0.826452 (505) = COFFEE MUG
    0.124038 (969) = CUP
    0.002276 (900) = WATER JUG
PASS
```
-------
Run server in container and client sdk in container:
1. Start the server side:
```
$ sudo docker run --gpus all --rm -p7000:8000 -p7001:8001 -p7002:8002 -v /home/maverick911/repo/server/docs/examples/model_repository:/models nvcr.io/nvidia/tritonserver:21.05-py3 tritonserver --model-repository=/models

+----------------------+---------+--------+
| Model                | Version | Status |
+----------------------+---------+--------+
| densenet_onnx        | 1       | READY  |
....
I0611 04:10:23.026207 1 grpc_server.cc:4062] Started GRPCInferenceService at 0.0.0.0:9001
I0611 04:10:23.036976 1 http_server.cc:2987] Started HTTPService at 0.0.0.0:9000
I0611 04:10:23.080860 1 http_server.c9:2906] Started Metrics Service at 0.0.0.0:9002
```
2. Start client image to start inferencing (shell), mount client src into container:
```
$ sudo docker run -it --rm --net=host -v /home/maverick911/repo/triton-inference-server_client/:/workspace/client nvcr.io/nvidia/tritonserver:21.05-py3-sdk
```
3. Infer, for ex:
```
$ image_client -m densenet_onnx -c 3 -s INCEPTION /workspace/images/mug.jpg
Request 0, batch size 1
Image '/workspace/images/mug.jpg':
    15.349568 (504) = COFFEE MUG
    13.227468 (968) = CUP
    10.424895 (505) = COFFEEPOT
```
## Example cmd list in sdk image:
```
$ python image_client.py -m inception_graphdef -s INCEPTION ./images/mug.jpg
Request 1, batch size 1
    0.826453 (505) = COFFEE MUG
PASS
$ python src/python/examples/image_client.py -i grpc -u localhost:8001 -m inception_graphdef -s INCEPTION ./images/mug.jpg
$ python src/python/examples/image_client.py -i grpc -u localhost:8001 -c 4 -m inception_graphdef -s INCEPTION ./images/mug.jpg
...
```
## Image Classification Example (use nvcr.io/nvidia/tritonserver:21.05-py3-sdk)

The image classification example that uses the C++ client API is
available at
[src/c++/examples/image_client.cc](src/c%2B%2B/examples/image_client.cc). The
Python version of the image classification client is available at
[src/python/examples/image_client.py](src/python/examples/image_client.py).

To use image_client (or image_client.py) you must first have a running
Triton that is serving one or more image classification models. The
image_client application requires that the model have a single image
input and produce a single classification output. If you don't have a
model repository with image classification models see
[QuickStart](https://github.com/triton-inference-server/server/blob/master/docs/quickstart.md)
for instructions on how to create one.

Once Triton is running you can use the image_client application to
send inference requests. You can specify a single image or a directory
holding images. Here we send a request for the inception_graphdef
model for an image from the
[qa/images](https://github.com/triton-inference-server/server/tree/master/qa/images).

```bash
$ image_client -m inception_graphdef -s INCEPTION qa/images/mug.jpg
Request 0, batch size 1
Image 'qa/images/mug.jpg':
    0.754130 (505) = COFFEE MUG
```

The Python version of the application accepts the same command-line
arguments.

```bash
$ python image_client.py -m inception_graphdef -s INCEPTION qa/images/mug.jpg
Request 0, batch size 1
Image 'qa/images/mug.jpg':
     0.826384 (505) = COFFEE MUG
```

The image_client and image_client.py applications use the client
libraries to talk to Triton. By default image_client instructs the
client library to use HTTP/REST protocol, but you can use the GRPC
protocol by providing the -i flag. You must also use the -u flag to
point at the GRPC endpoint on Triton.

```bash
$ image_client -i grpc -u localhost:8001 -m inception_graphdef -s INCEPTION qa/images/mug.jpg
Request 0, batch size 1
Image 'qa/images/mug.jpg':
    0.754130 (505) = COFFEE MUG
```

By default the client prints the most probable classification for the
image. Use the -c flag to see more classifications.

```bash
$ image_client -m inception_graphdef -s INCEPTION -c 3 qa/images/mug.jpg
Request 0, batch size 1
Image 'qa/images/mug.jpg':
    0.754130 (505) = COFFEE MUG
    0.157077 (969) = CUP
    0.002880 (968) = ESPRESSO
```

The -b flag allows you to send a batch of images for inferencing.
The image_client application will form the batch from the image or
images that you specified. If the batch is bigger than the number of
images then image_client will just repeat the images to fill the
batch.

```bash
$ image_client -m inception_graphdef -s INCEPTION -c 3 -b 2 qa/images/mug.jpg
Request 0, batch size 2
Image 'qa/images/mug.jpg':
    0.754130 (505) = COFFEE MUG
    0.157077 (969) = CUP
    0.002880 (968) = ESPRESSO
Image 'qa/images/mug.jpg':
    0.754130 (505) = COFFEE MUG
    0.157077 (969) = CUP
    0.002880 (968) = ESPRESSO
```

Provide a directory instead of a single image to perform inferencing
on all images in the directory.

```
$ image_client -m inception_graphdef -s INCEPTION -c 3 -b 2 qa/images
Request 0, batch size 2
Image '/opt/tritonserver/qa/images/car.jpg':
    0.819196 (818) = SPORTS CAR
    0.033457 (437) = BEACH WAGON
    0.031232 (480) = CAR WHEEL
Image '/opt/tritonserver/qa/images/mug.jpg':
    0.754130 (505) = COFFEE MUG
    0.157077 (969) = CUP
    0.002880 (968) = ESPRESSO
Request 1, batch size 2
Image '/opt/tritonserver/qa/images/vulture.jpeg':
    0.977632 (24) = VULTURE
    0.000613 (9) = HEN
    0.000560 (137) = EUROPEAN GALLINULE
Image '/opt/tritonserver/qa/images/car.jpg':
    0.819196 (818) = SPORTS CAR
    0.033457 (437) = BEACH WAGON
    0.031232 (480) = CAR WHEEL
```
## Note:
Trained model which saved by torch.save (usually .pth) must be convert into torchscript by torch.jit.save (into model.pt as default name of Triton).
Ex: git
    # define model class...
    model = model.to(device)
    # Switch the model to eval model
    model.eval()
    # An example input you would normally provide to your model's forward() method.
    x = torch.randn(1, 3, 768, 768).to(device)
    # Use torch.jit.trace to generate a torch.jit.ScriptModule via tracing.
    traced_script_module = torch.jit.trace(model, x)
    # Save the TorchScript model
    traced_script_module.save('model.pt')

--------
# ORIGINAL README
---------------------------------------
# Triton Client Libraries and Examples

To simplify communication with Triton, the Triton project provides
several client libraries and examples of how to use those
libraries. Ask questions or report problems in the main Triton [issues
page](https://github.com/triton-inference-server/server/issues).

The provided client libaries are:

* [C++ and Python APIs](#client-library-apis) that make it easy to
  communicate with Triton from your C++ or Python application. Using
  these libraries you can send either HTTP/REST or GRPC requests to
  Triton to access all its capabilities: inferencing, status and
  health, statistics and metrics, model repository management,
  etc. These libraries also support using system and CUDA shared
  memory for passing inputs to and receiving outputs from Triton.

* The [protoc
  compiler](https://developers.google.com/protocol-buffers/docs/tutorials)
  can generate a GRPC API in a large number of programming
  languages. See [src/go](src/go) for an example for the [Go
  programming language](https://golang.org/). See [src/java](src/java)
  for an example for the Java and Scala programming languages.

There are also many example applications that show how to use these
libraries. Many of these examples use models from the [example model
repository](https://github.com/triton-inference-server/server/blob/master/docs/quickstart.md#create-a-model-repository).

* C++ and Python versions of *image_client*, an example application
  that uses the C++ or Python client library to execute image
  classification models on Triton. See [Image Classification
  Example](#image-classification-example).

* Several simple [C++ examples](src/c%2B%2B/examples) show
  how to use the C++ library to communicate with Triton to perform
  inferencing and other task. The C++ examples demonstrating the
  HTTP/REST client are named with a *simple_http_* prefix and the
  examples demonstrating the GRPC client are named with a
  *simple_grpc_* prefix. See [Simple Example
  Applications](#simple-example-applications).

* Several simple [Python examples](src/python/examples)
  show how to use the Python library to communicate with Triton to
  perform inferencing and other task. The Python examples
  demonstrating the HTTP/REST client are named with a *simple_http_*
  prefix and the examples demonstrating the GRPC client are named with
  a *simple_grpc_* prefix. See [Simple Example
  Applications](#simple-example-applications).

* A couple of [Python examples that communicate with Triton using a
  Python GRPC API](src/python/examples) generated by the
  [protoc compiler](https://grpc.io/docs/guides/). *grpc_client.py* is
  a simple example that shows simple API
  usage. *grpc_image_client.py* is functionally equivalent to
  *image_client* but that uses a generated GRPC client stub to
  communicate with Triton.

## Getting the Client Libraries And Examples

The easiest way to get the Python client library is to [use pip to
install the tritonclient
module](#download-using-python-package-installer-pip). You can also
download both C++ and Python client libraries from [Triton GitHub
release](#download-from-github), or [download a pre-built Docker image
containing the client libraries](#download-docker-image-from-ngc) from
[NVIDIA GPU Cloud (NGC)](https://ngc.nvidia.com).

It is also possible to build build the client libraries with
[cmake](#build-using-cmake).

### Download Using Python Package Installer (pip)

The GRPC and HTTP client libraries are available as a Python package
that can be installed using a recent version of pip. **Currently pip
install is only available on Linux**.

```
$ pip install nvidia-pyindex
$ pip install tritonclient[all]
```

Using *all* installs both the HTTP/REST and GRPC client
libraries. There are two optional packages available, *grpc* and
*http* that can be used to install support specifically for the
protocol. For example, to install only the HTTP/REST client library
use,

```
$ pip install nvidia-pyindex
$ pip install tritonclient[http]
```

The components of the install packages are:

* http
* grpc [ `service_pb2`, `service_pb2_grpc`, `model_config_pb2` ]
* utils [ linux distribution will include `shared_memory` and `cuda_shared_memory`]

The Linux version of the package also includes the
[perf_analyzer](https://github.com/triton-inference-server/server/blob/master/docs/perf_analyzer.md)
binary. The perf_analyzer binary is built on Ubuntu 20.04 and may not
run on other Linux distributions. To run the perf_analyzer the
following dependency must be installed:

```bash
sudo apt update
sudo apt install libb64-dev
```

### Download From GitHub

The client libraries and the perf_analyzer executable can be
downloaded from the [Triton GitHub release
page](https://github.com/triton-inference-server/server/releases)
corresponding to the release you are interested in. The client
libraries are found in the "Assets" section of the release page in a
tar file named after the version of the release and the OS, for
example, v2.3.0_ubuntu2004.clients.tar.gz.

The pre-built libraries can be used on the corresponding host system
or you can install them into the Triton container to have both the
clients and server in the same container.

```bash
$ mkdir clients
$ cd clients
$ wget https://github.com/triton-inference-server/server/releases/download/<tarfile_path>
$ tar xzf <tarfile_name>
```

After installing, the libraries can be found in lib/, the headers in
include/, and the Python wheel files in python/. The bin/ and python/
directories contain the built examples that you can learn more about
below.

The perf_analyzer binary is built on Ubuntu 20.04 and may not run on
other Linux distributions. To use the C++ libraries or perf_analyzer
executable you must install some dependencies.

```bash
$ apt-get update
$ apt-get install curl libcurl4-openssl-dev libb64-dev
```

### Download Docker Image From NGC

A Docker image containing the client libraries and examples is
available from [NVIDIA GPU Cloud
(NGC)](https://ngc.nvidia.com). Before attempting to pull the
container ensure you have access to NGC.  For step-by-step
instructions, see the [NGC Getting Started
Guide](http://docs.nvidia.com/ngc/ngc-getting-started-guide/index.html).

Use docker pull to get the client libraries and examples container
from NGC.

```bash
$ docker pull nvcr.io/nvidia/tritonserver:<xx.yy>-py3-sdk
```

Where \<xx.yy\> is the version that you want to pull. Within the
container the client libraries are in /workspace/install/lib, the
corresponding headers in /workspace/install/include, and the Python
wheel files in /workspace/install/python. The image will also contain
the built client examples.

### Build Using CMake

The client library build is performed using CMake. To build the client
libraries and examples with all features, first change directory to
the root of this repo and checkout the release version of the branch
that you want to build (or the *main* branch if you want to build the
under-development version).

```bash
$ git checkout main
```

Building on Windows vs. non-Windows requires different invocations
because Triton on Windows does not yet support all the build options.

#### Non-Windows

Use *cmake* to configure the build.

```
$ mkdir build
$ cd build
$ cmake -DCMAKE_INSTALL_PREFIX=`pwd`/install -DTRITON_ENABLE_CC_HTTP=ON -DTRITON_ENABLE_CC_GRPC=ON -DTRITON_ENABLE_PERF_ANALYZER=ON -DTRITON_ENABLE_PYTHON_HTTP=ON -DTRITON_ENABLE_PYTHON_GRPC=ON -DTRITON_ENABLE_GPU=ON -DTRITON_ENABLE_EXAMPLES=ON -DTRITON_ENABLE_TESTS=ON ..
```

Then use *make* to build the clients and examples.

```
$ make cc-clients python-clients
```

When the build completes the libraries and examples can be found in
the install directory.

#### Windows

To build the clients you must install an appropriate C++ compiler and
other dependencies required for the build. The easiest way to do this
is to create the [Windows min Docker
image](https://github.com/triton-inference-server/server/blob/master/docs/build.md#windows-10-min-container)
and the perform the build within a container launched from that image.

```
> docker run  -it --rm win10-py3-min powershell
```

It is not necessary to use Docker or the win10-py3-min container for
the build, but if you do not you must install the appropriate
dependencies onto your host system.

Next use *cmake* to configure the build. If you are not building
within the win10-py3-min container then you will likely need to adjust
the CMAKE_TOOLCHAIN_FILE location in the following command.

```
$ mkdir build
$ cd build
$ cmake -DVCPKG_TARGET_TRIPLET=x64-windows -DCMAKE_TOOLCHAIN_FILE='/vcpkg/scripts/buildsystems/vcpkg.cmake' -DCMAKE_INSTALL_PREFIX=install -DTRITON_ENABLE_CC_GRPC=ON -DTRITON_ENABLE_PYTHON_GRPC=ON -DTRITON_ENABLE_GPU=OFF -DTRITON_ENABLE_EXAMPLES=ON -DTRITON_ENABLE_TESTS=ON ..
```

Then use msbuild.exe to build.

```
$ msbuild.exe cc-clients.vcxproj -p:Configuration=Release -clp:ErrorsOnly
$ msbuild.exe python-clients.vcxproj -p:Configuration=Release -clp:ErrorsOnly
```

When the build completes the libraries and examples can be found in
the install directory.

## Client Library APIs

The C++ client API exposes a class-based interface. The commented
interface is available in
[grpc_client.h](src/c%2B%2B/library/grpc_client.h),
[http_client.h](src/c%2B%2B/library/http_client.h),
[common.h](src/c%2B%2B/library/common.h).

The Python client API provides similar capabilities as the C++
API. The commented interface is available in
[grpc](src/python/library/tritonclient/grpc/__init__.py)
and
[http](src/python/library/tritonclient/http/__init__.py).

## Simple Example Applications

This section describes several of the simple example applications and
the features that they illustrate.

### Bytes/String Datatype

Some frameworks support tensors where each element in the tensor is
variable-length binary data. Each element can hold a string or an
arbitrary sequence of bytes. On the client this datatype is BYTES (see
[Datatypes](https://github.com/triton-inference-server/server/blob/master/docs/model_configuration.md#datatypes)
for information on supported datatypes).

The Python client library uses numpy to represent input and output
tensors. For BYTES tensors the dtype of the numpy array should be
'np.object_' as shown in the examples. For backwards compatibility
with previous versions of the client library, 'np.bytes_' can also be
used for BYTES tensors. However, using 'np.bytes_' is not recommended
because using this dtype will cause numpy to remove all trailing zeros
from each array element. As a result, binary sequences ending in
zero(s) will not be represented correctly.

BYTES tensors are demonstrated in the C++ example applications
simple_http_string_infer_client.cc and
simple_grpc_string_infer_client.cc.  String tensors are demonstrated
in the Python example application simple_http_string_infer_client.py
and simple_grpc_string_infer_client.py.

### System Shared Memory

Using system shared memory to communicate tensors between the client
library and Triton can significantly improve performance in some
cases.

Using system shared memory is demonstrated in the C++ example
applications simple_http_shm_client.cc and simple_grpc_shm_client.cc.
Using system shared memory is demonstrated in the Python example
application simple_http_shm_client.py and simple_grpc_shm_client.py.

Python does not have a standard way of allocating and accessing shared
memory so as an example a simple [system shared memory
module](src/python/library/tritonclient/utils/shared_memory)
is provided that can be used with the Python client library to create,
set and destroy system shared memory.

### CUDA Shared Memory

Using CUDA shared memory to communicate tensors between the client
library and Triton can significantly improve performance in some
cases.

Using CUDA shared memory is demonstrated in the C++ example
applications simple_http_cudashm_client.cc and
simple_grpc_cudashm_client.cc.  Using CUDA shared memory is
demonstrated in the Python example application
simple_http_cudashm_client.py and simple_grpc_cudashm_client.py.

Python does not have a standard way of allocating and accessing shared
memory so as an example a simple [CUDA shared memory
module](src/python/library/tritonclient/utils/cuda_shared_memory)
is provided that can be used with the Python client library to create,
set and destroy CUDA shared memory.

### Client API for Stateful Models

When performing inference using a [stateful
model](https://github.com/triton-inference-server/server/blob/master/docs/architecture.md#stateful-models),
a client must identify which inference requests belong to the same
sequence and also when a sequence starts and ends.

Each sequence is identified with a sequence ID that is provided when
an inference request is made. It is up to the clients to create a
unique sequence ID. For each sequence the first inference request
should be marked as the start of the sequence and the last inference
requests should be marked as the end of the sequence.

The use of sequence ID and start and end flags are demonstrated in the
C++ example applications simple_http_sequence_stream_infer_client.cc
and simple_grpc_sequence_stream_infer_client.cc.  The use of sequence
ID and start and end flags are demonstrated in the Python example
application simple_http_sequence_stream_infer_client.py and
simple_grpc_sequence_stream_infer_client.py.

## Image Classification Example

The image classification example that uses the C++ client API is
available at
[src/c++/examples/image_client.cc](src/c%2B%2B/examples/image_client.cc). The
Python version of the image classification client is available at
[src/python/examples/image_client.py](src/python/examples/image_client.py).

To use image_client (or image_client.py) you must first have a running
Triton that is serving one or more image classification models. The
image_client application requires that the model have a single image
input and produce a single classification output. If you don't have a
model repository with image classification models see
[QuickStart](https://github.com/triton-inference-server/server/blob/master/docs/quickstart.md)
for instructions on how to create one.

Once Triton is running you can use the image_client application to
send inference requests. You can specify a single image or a directory
holding images. Here we send a request for the inception_graphdef
model for an image from the
[qa/images](https://github.com/triton-inference-server/server/tree/master/qa/images).

```bash
$ image_client -m inception_graphdef -s INCEPTION qa/images/mug.jpg
Request 0, batch size 1
Image 'qa/images/mug.jpg':
    0.754130 (505) = COFFEE MUG
```

The Python version of the application accepts the same command-line
arguments.

```bash
$ python image_client.py -m inception_graphdef -s INCEPTION qa/images/mug.jpg
Request 0, batch size 1
Image 'qa/images/mug.jpg':
     0.826384 (505) = COFFEE MUG
```

The image_client and image_client.py applications use the client
libraries to talk to Triton. By default image_client instructs the
client library to use HTTP/REST protocol, but you can use the GRPC
protocol by providing the -i flag. You must also use the -u flag to
point at the GRPC endpoint on Triton.

```bash
$ image_client -i grpc -u localhost:8001 -m inception_graphdef -s INCEPTION qa/images/mug.jpg
Request 0, batch size 1
Image 'qa/images/mug.jpg':
    0.754130 (505) = COFFEE MUG
```

By default the client prints the most probable classification for the
image. Use the -c flag to see more classifications.

```bash
$ image_client -m inception_graphdef -s INCEPTION -c 3 qa/images/mug.jpg
Request 0, batch size 1
Image 'qa/images/mug.jpg':
    0.754130 (505) = COFFEE MUG
    0.157077 (969) = CUP
    0.002880 (968) = ESPRESSO
```

The -b flag allows you to send a batch of images for inferencing.
The image_client application will form the batch from the image or
images that you specified. If the batch is bigger than the number of
images then image_client will just repeat the images to fill the
batch.

```bash
$ image_client -m inception_graphdef -s INCEPTION -c 3 -b 2 qa/images/mug.jpg
Request 0, batch size 2
Image 'qa/images/mug.jpg':
    0.754130 (505) = COFFEE MUG
    0.157077 (969) = CUP
    0.002880 (968) = ESPRESSO
Image 'qa/images/mug.jpg':
    0.754130 (505) = COFFEE MUG
    0.157077 (969) = CUP
    0.002880 (968) = ESPRESSO
```

Provide a directory instead of a single image to perform inferencing
on all images in the directory.

```
$ image_client -m inception_graphdef -s INCEPTION -c 3 -b 2 qa/images
Request 0, batch size 2
Image '/opt/tritonserver/qa/images/car.jpg':
    0.819196 (818) = SPORTS CAR
    0.033457 (437) = BEACH WAGON
    0.031232 (480) = CAR WHEEL
Image '/opt/tritonserver/qa/images/mug.jpg':
    0.754130 (505) = COFFEE MUG
    0.157077 (969) = CUP
    0.002880 (968) = ESPRESSO
Request 1, batch size 2
Image '/opt/tritonserver/qa/images/vulture.jpeg':
    0.977632 (24) = VULTURE
    0.000613 (9) = HEN
    0.000560 (137) = EUROPEAN GALLINULE
Image '/opt/tritonserver/qa/images/car.jpg':
    0.819196 (818) = SPORTS CAR
    0.033457 (437) = BEACH WAGON
    0.031232 (480) = CAR WHEEL
```

The [grpc_image_client.py](src/python/examples/grpc_image_client.py)
application behaves the same as the image_client except that instead
of using the client library it uses the GRPC generated library to
communicate with Triton.

## Ensemble Image Classification Example Application

In comparison to the image classification example above, this example
uses an ensemble of an image-preprocessing model implemented as a
[DALI
backend](https://github.com/triton-inference-server/dali_backend) and
a TensorFlow Inception model. The ensemble model allows you to send
the raw image binaries in the request and receive classification
results without preprocessing the images on the client.

To try this example you should follow the [DALI ensemble example
instructions](https://github.com/triton-inference-server/dali_backend/tree/main/docs/examples/inception_ensemble).
