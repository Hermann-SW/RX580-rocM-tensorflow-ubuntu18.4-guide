# Installation of ROCm and Tensorflow on Ubuntu 18.04.5 LTS for Radeon RX580  
**This guide was updated as of 12.14.2024 because wayne0320 did not state which Ubuntu 18.4 version is the needed one.**

*This guide was updated as of 25.01.2021 due to new version of tensorflow-rocm being incompatible.*


This guide will show you how to set up your **clean Ubuntu 18.04.5  LTS** OS to be ready to run **Tensorflow** projects, using **ROCm** to take advantage of the power of your **RX580 graphics card (or any gfx803)** in a tested, easy and fast way (It should work on other supported Ubuntu versions and other graphic cards too, with only slight changes).

It is basically a resume of the [official guide by AMD](https://rocmdocs.amd.com/en/latest/Installation_Guide/Installation-Guide.html), and the [unofficial guide by Mathieu Poliquin](https://www.videogames.ai/Install-ROCM-Machine-Learning-AMD-GPU). I highly recommend to check those links, in order to understand what you are doing and why.

There is another important thing to notice: **In this guide we downgrade ROCm to 2.5 since there are some bugs in posterior versions which have not been fixed yet, *not at all*** (bugs: [1](https://github.com/RadeonOpenCompute/ROCm/issues/1269), [2](https://github.com/RadeonOpenCompute/ROCm/issues/1265)). In the case you have a newer version already installed, you will need to remove it first.

Lets get started!
## Remove ROCm (you can skip this step if you don't have ROCm installed)
1. ```sudo apt autoremove rocm-dkms```
2. Make sure that all packages are removed under /opt/rocm-xxx
3. Check ```sudo dpkg -l | grep hsa``` (replace ```hsa``` with ```hip```, ```llvm```, ```rocm``` and ```rock```). Make sure that all packages are removed with ```sudo apt purge``` . I recommend to do the same for any other additional packages (if you installed anything explicitly).
*I have occasionally removed the GUI for Ubuntu, so be careful!*
4. Reboot the system

P.S. It's preferrable to do a fresh Ubuntu reinstall instead of removing ROCm - strange bugs may occur.
## Install ROCm 2.5
```
sudo apt update
sudo apt dist-upgrade
sudo apt install libnuma-dev
sudo reboot
```
- These commands also upgrade the kernel. Unfortunately, ROCm needs specific kernel to run on (5.4.0-42-generic). To downgrade your kernel:
  1. Reboot the computer. In GRUB menu select "Additional options for Ubuntu" and select "Boot with kernel 5.4.0-42-generic (This is the default one Ubuntu 18.04 LTS is shipped with). Also memorize all the other kernel versions from the entries of that menu (5.8.0 by the time of writing this article)
  1. Remove the newer kernels: ``` sudo apt-get purge *5.8.0* ``` (and/or any other versions except for the 5.4.0-42-generic)
  1. Reboot and check your kernel version with ```uname -r``` (it should be 5.4.0-42-generic)

Add the repo and install rocm-dkms:
```
wget -q -O - http://repo.radeon.com/rocm/apt/2.5/rocm.gpg.key | sudo apt-key add -
echo 'deb [arch=amd64] http://repo.radeon.com/rocm/apt/2.5/ xenial main' | sudo tee /etc/apt/sources.list.d/rocm.list
sudo apt update
sudo apt install rocm-dkms && sudo reboot
```
Set user permissions to access video card features:
```
groups
sudo usermod -a -G video $LOGNAME
sudo reboot
```
Add rocm to PATH:
```
echo 'export PATH=$PATH:/opt/rocm/bin:/opt/rocm/profiler/bin:/opt/rocm/opencl/bin' | sudo tee -a /etc/profile.d/rocm.sh
sudo ldconfig
sudo reboot
```
- [x] **Check your progress:** At this point you should be able to enter `rocminfo` into a terminal without getting any error. You should also see your video card name in the command output (something like this):
> Name:                    gfx803                             
> Uuid:                    GPU-XX                             
> Marketing Name:          Ellesmere [Radeon RX 470/480/570/570X/580/580X/590]

You are half the way now!
## Install Tensorflow
```
sudo apt install python3 python3-pip
sudo apt install rocm-libs miopen-hip cxlactivitylogger
pip3 install -Iv tensorflow-rocm==1.13.4
sudo apt install rccl
sudo apt install libtinfo5
sudo reboot
```
- [x] **Check your progress:** At this point you should be able to import Tensorflow in Python, make a simple operation, and exit without any error. Try the following in a python interactive session:
```python
import tensorflow as tf
tf.add(2,5)
exit()
```
You should see something like this: 
> <tf.Tensor: shape=(), dtype=int32, numpy=7>

- If you see an error:
  ```
  Could not load dynamic library 'libhip_hcc.so'; dlerror: libhip_hcc.so: cannot open shared object file: No such file or directory
  ```
  Then, you can use any workaround from this [issue](https://github.com/RadeonOpenCompute/ROCm/issues/1163). I have used:
  ```
  export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/rocm/hip/lib
  sudo ldconfig
  ```
  This will only temporarily set the variable. If this fix works, you need to permanently set the variable: [Link](https://askubuntu.com/questions/887442/how-to-permanently-set-an-environment-variable)

Congrats, your machine is now ready to use tensorflow-rocm! You should still consider testing it with something more complex, like a benchmark.
## Test with a benchmark
```
sudo apt install git
git clone https://github.com/tensorflow/benchmarks
cd ./benchmarks/scripts/tf_cnn_benchmarks
python3 tf_cnn_benchmarks.py --num_gpus=1 --batch_size=32 --model=resnet50
```
Expect it to take some time (5-10 minutes), specially if it is the first time. You may think you got stuck in the warm up but be patient, it should not take that long next time.

If you see something like the output bellow (numbers may vary a lot) then go and get yourself a taco, you did it!
> total images/sec: 87.92

***Have fun! :D***


-------
Special thanks to [Boris Timofeenko](https://github.com/boriswinner) for updating this guide.
