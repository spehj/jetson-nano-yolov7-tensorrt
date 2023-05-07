# Use YOLOv7 and TensorRT on Jetson Nano

Here is complete tutorial on how to deploy YOLOv7 (tiny) to Jeton Nano in 2 steps:
1. Basic deploy: install PyTorch and TorchVision, clone YOLOv7 repository and run inference. Result is around 17 FPS (YOLOv7 Tiny with input of 416x416) and 9 FPS (YOLOv7 Tiny with input of 640x640).
2. Optimization using TensorRT. Result is around 30FPS (YOLOv7 Tiny with input of 416x416) and 15 FPS (YOLOv7 Tiny with input of 640x640).

## Basic Deploy
If you play with YOLOv7 and Jetson Nano for the first time, I recommend to go through [this tutorial](https://www.hackster.io/spehj/deploy-yolov7-to-jetson-nano-for-object-detection-6728c3). It will help you to setup your environment and guide you through cloning official YOLOv7 repository, and installing PyTorch and TorchVision. 

At the end you will be able to run YOLOv7 algorithm on Jetson Nano.

Now we can start optimization.

## TensorRT optimization

Go to home directory:
~~~bash
cd
~~~

Clone [TensorRTx repository](https://github.com/wang-xinyu/tensorrtx). 
~~~bash
git clone https://github.com/wang-xinyu/tensorrtx
~~~

We have [weights](https://github.com/WongKinYiu/yolov7/releases/download/v0.1/yolov7-tiny.pt) (.pt files) downloaded from previous step (tiny version). Weights should be in your cloned official YOLOv7 repository.

1. Go to cloned tensorrtx repository and copy script for generating .wts file from tensorrtx to yolov7 directory:
~~~bash
cd tensorrtx/yolov7
cp cp gen_wts.py {your-path-to-yolov7-directory}/yolov7
# Move to yolov7 directory
cd {your-path-to-yolov7-directory}/yolov7
~~~
Make sure you have your virtualenv activated to have access to installed pytorch library.

Convert .pt file to .wts file. This will take around 30 seconds.
~~~bash
python gen_wts.py -w yolov7-tiny.pt 
~~~

You should get new __yolov7-tiny.wts__ file in your current directory.

2. Build tensorrtx. Go to tensorrtx directory:
~~~bash
cd
cd tensorrtx/yolov7
~~~
Go to /include directory and open __config.h__. Find variable kNumClass and check if number of classes match your model's number of classes (change if you have custom trained model).

~~~bash
mkdir build
cd build
cp {your-path-to-yolov7-directory}/yolov7/yolov7.wts {your-path to-tensorrtx-directory}/yolov7/build
cmake ..
make

# Serialize model to engine file
sudo ./yolov7 -s [.wts] [.engine] [t/v7/x/w6/e6/d6/e6e gd gw]
# Example:
sudo ./yolov7 -s yolov7-tiny.wts yolov7-tiny.engine t

# Deserialize and run inference, the images in [image folder] will be processed.
sudo ./yolov7 -d [.engine] [image folder]
# Example:
sudo ./yolov7 -d yolov7-tiny.engine ../images
~~~

Check images that were generated (zidane.jpg and bus.jpg).

We also have a __yolov7_trt_cam.py__ inside deepstream/scripts/ directory.

To test it, first install pycuda and python-tensorrt.

Execute this command:
~~~bash
ln -s /usr/include/locale.h /usr/include/xlocale.h
~~~

Install pycuda with some added options:
~~~bash
pip3 install --global-option=build_ext --global-option="-I/usr/local/cuda/include" --global-option="-L/usr/local/cuda/lib64" pycuda
~~~

Open .bashrc file and add this to the end:
~~~bash
sudo nano ~/.bashrc

# Add this line to the end
export PYTHONPATH=/usr/lib/python3.6/dist-packages:$PYTHONPATH
# Save and exit
source ~/.bashrc
~~~

<!--
Alternative option to install pycuda (not tested):
~~~bash
# Another option:
export PATH=/usr/local/cuda-10.2/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-10.2/lib64:$LD_LIBRARY_PATH
pip3 install pycuda --user
~~~
-->

If you still have problems installing pycuda and tensorrt, check out [this tutorial](https://www.bojankomazec.com/2019/12/how-to-install-tensorrt-python-package.html).

Go to deepstream/scripts/ and run __yolov7_trt_cam.py__. You can edit input and engine file inside Python code. __Make sure you use correct paths!__
