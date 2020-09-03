# Jetson-Nano (Nvidia Edge AI Platform)

Environment:
 - host: ubuntu 18.04
 - target: Jetson-Nano
 - dependencies 
 
The [dependencies downloaded from Nvidia](https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-devkit) includes:
 - Linux image: For [Jetson Nano Developer Kit](https://developer.nvidia.com/embedded/downloads)
 - Flash tool: [balena-etcher-electron](https://www.balena.io/etcher/)

For the TensorRT on host, please referr to the [website](https://zhuanlan.zhihu.com/p/85365075) or [Nvidia](https://zhuanlan.zhihu.com/p/85365075). Note that the engine built from host cannot run on the target.


# Hardware setup

For Jetson-Nano, preparing the following:
 -  Power
 -  Wirless keyboard/mouse (Control)
 -  HDMI (Connection)
 
To setup the Jetson-Nano:
 - load the linux image to SD card by balena.
 - Insert the SD card to Jetson-Nano


# Headless initial configuration

If no display is attached to the developer kit during first boot, it can be communicated with the serial applications via a host serial port and the developer kit’s Micro-USB port.

 - UART connection with micro USB

```bash
$ dmesg | grep tty   # 1 of 4 port can be connected
$ sudo apt-get install gtkterm
$ sudo gtkterm

configuration:
baud rate: 115200bps
data bit: 8
stop bit: 1 
no parity
```

 - SSH connection with Ethernet

```bash
$ ssh username@192.168.55.1 # after the account initialized
```


# Install tensorflow dependencies

Install the dependencies for tensorflow 1.15
```bash
$ sudo apt-get update
$ sudo apt-get install libhdf5-serial-dev hdf5-tools libhdf5-dev zlib1g-dev zip libjpeg8-dev liblapack-dev libblas-dev gfortran libcanberra-gtk*
$ sudo apt-get install python3-pip
$ pip3 install -U pip testresources setuptools
$ pip3 install -U numpy==1.16.1 future==0.17.1 mock==3.0.5 h5py==2.9.0 keras_preprocessing==1.0.5 keras_applications==1.0.8 gast==0.2.2 futures protobuf pybind11
$ pip3 install --pre --extra-index-url https://developer.download.nvidia.com/compute/redist/jp/v44 'tensorflow<2'
```

Install the dependencies for buiding tensorRT engine.
```bash
$ pip3 install Pillow
$ export LD_LIBRARY_PATH=/usr/local/cuda/lib64:\${LD_LIBRARY_PATH}
$ pip3 install pycuda
$ pip3 install numpy
$ sudo apt-get install protobuf-compiler libprotoc-dev 
$ pip3 install onnx==1.4.1
$ pip3 install keras==2.3.1
$ pip3 install pyzmq==17.0.0
$ pip3 install jupyter
```


# Model deployment

Please download the model config and weights from the [model zoo](https://pjreddie.com/darknet/yolo/). Due to the custom layer in yolov3 cannot be parsed on uff-format, it should be parsed to the onnx-format, then converting to the tensorRT engine. 

Please use the ipynb to run the model on Jetson-Nano, and the sample code can be found on /usr/src/tensorrt/samples/
(Plan engine files are specific to the exact GPU model and tensorRT version)

 - Build: Import and optimize trained models to generate inference engines

    The Builder interface allows the creation of an optimized engine from a network definition and a builder configuration. For more information, refer to the [Builder API](https://docs.nvidia.com/deeplearning/tensorrt/api/c_api/classnvinfer1_1_1_i_builder.html).

 - Deploy: Generate runtime inference engine for inference

    The Engine interface allows the application to execute inference. It supports synchronous and asynchronous execution, profiling, and enumeration and querying of the bindings for the engine inputs and outputs. A single-engine can have multiple execution context. For more information, refer to the [Execution API](https://docs.nvidia.com/deeplearning/tensorrt/api/c_api/classnvinfer1_1_1_i_cuda_engine.html).


```bash
$ python3 yolov3_tiny_to_onnx.py --model yolov3-416
$ python3 onnx_to_tensorrt.py --model yolov3-416
$ jupyter notebook tensorRT.ipynb

$ python3 setup.py build_ext --inplace
```
