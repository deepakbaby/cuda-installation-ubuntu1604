# cuda-installation-ubuntu1604
Guidelines for installing CUDA and cuDNN on Ubuntu-16.04 (other linux flavors as well)

The procedure is tested on a server with two different GPUs (GeForce 1080p and Tesla Xp).
The guide is for Ubuntu 16.04, but may be helpful for other linux flavors as well.

#### *The common installation issues are listed first. For the final complete installation steps, skip to the installation setps section.* ####
----
 ## Common Issues ##
While installing cuda and nvidia drivers on linux, the common issues are :
* Login loop 
  
  The system goes to login loop after reboot. This is typically cause by OpenGL libraries messing up with the GUI. The solution is to avoid installing them.

* ERR! instead of the device name when calling nvidia-smi
  
  This happens even after a successful installation using the *.run* file. *ERR!* indeed is a result of a failed installation and the solution is to install the NVIDIA driver specific for the GPU separately. See below for more details.
----
## Lessons learned ##
 1. Dont use the system specific installation files (*.deb* installer in this case)
 2. Use the runfile installation for installing cuda.
 3. Do not install NVIDIA Graphics driver using the above *.run* file.
 4. Download the driver separately and install it. See below for complete instructions.  
  These are the lessons I learned while trying to install cuda on the GPU server machine. The complete installation steps are   given below based on these. The main point is that the `.run` files are sometimes old and are created before the latest GPUs are released. The drivers for different series in fact are different and that is the reason why __it is important not to install NVIDIA drivers using the `cuda<>.run` file___.
----
## Installation steps ##
1. It is better to start with a fresh install. Update the OS after the fresh install
      
   ```bash
   $ sudo apt-get update 
   $ sudo apt-get upgrade 
   ```
2. Make sure you have a CUDA capable GPU and your system meets the requirements (as given [here](http://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#pre-installation-actions)).

3. Install the kernel headers for the current Ubuntu installation.
    ```bash
    $ sudo apt-get intsall linux-headers-$(uname -r)
    ```
    For other linux flavors, this step is different (Refer [here](http://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#pre-installation-actions) to install the required header files for your distro).
  
4. Download the runfile for your linux flavor from the [cuda downloads page](https://developer.nvidia.com/cuda-downloads).

5. Download the graphics driver for your graphics card from the [driver donwnloads page](http://www.nvidia.com/Download/index.aspx?lang=en-us).

6. Disable the nouveau driver. The isntructions are given in this [page](http://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#runfile-nouveau). For ubuntu, create a new file `/etc/modprobe.d/blacklist-nouveau.conf` with the following contents:
    ```bash
    blacklist nouveau
    options nouveau modeset=0  
    ```
    Regenerate the kernel initramfs:
    
       $ sudo update-initramfs -u
      
7. Reboot and once you are in the login page, open a TTY by pressing `Ctrl + Alt + F1`. Login using your credentials.

8. Go to the Downloads folder where the files downloaded in Steps 4 and 5 are stored. Make the files executable :
    ```bash
    chmod +x cuda<version>.<repo>.run NVIDIA_<repo>.run
    ```

9. Disable the lightdm service to kill the X server from running.
    ```bash
    sudo service lightdm stop
    ```
10. Install the cuda driver:
    ```bash
    sudo ./cuda<version>.<repo>.run -a --no-opengl-libs
    ```
   The flag `-a` accepts the license agreement and prevents the looong license from being displayed on the terminal. 
   __Important :  the flag `--no-opengl-libs` is important to avoid the login loop problem.__
   You will then be asked the following and the required responses are:
   
  1. Install NVIDIA driver ? : __No__
  2. Should NVIDIA modify the x-config ? : __No__
  3. Install CUDA ? : __Yes__   
  4. Path where cuda installations should be put : __choose default or provide a path of your choice__
  5. Install symbolic link ? : __Yes__     
  6. Install samples ? : __Yes__
  7. Choose samples location : __choose default or enter your choice__
    
   This should install the cuda and the samples in your machine. The next step is to install the driver.
 
11. Install the NVIDIA graphics driver:
    ```bash
    sudo ./cuda<version>.<repo>.run -a --no-opengl-files
    ```
    Notice the change f option `--no-opengl-files` here. Follow the instructions as above to finish the installation. 
  
12. Perform the post installation actions such as adding the cuda installation to your `PATH` and `LD_LIBRARY_PATH` (Details [here](http://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#post-installation-actions)).

13. Finally, try `nvcc -V` to check the nvidia compiler version and `nvidia-smi` to see the GPUs' status in your machine.

14. Now, you can rebbot into the GUI and everything should work fine.

----

## Misc ##
* This setting works when both (or all) the GPUs belong to the same series (In this case, both belongs to the GeForce 1080 series). I am not sure how the driver installation works for GPUs from two series are there.

* CUDA with MATLAB : When you are trying to call `gpuDevice(1)` in an older MATLAB version after the CUDA install, it takes some time since the old MATLAB is unaware of the latest GPU. It then searches the web and gather required infor for the device. From second time onwards, this will go smooth as the new CUDA details are cached by MATLAB.
