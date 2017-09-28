## Setting up a Deep Learning compute instance on Google Cloud Platform

*Note: In most places where the user will be prompted to answer yes/no to a prompt, we add `-y` to the end of the command so it will automatically answer yes to all prompts. However, if you run into a case where it prompts you to answer, then always answer in the affirmative.*

#### Step 1. Upgrade Google Cloud Account from the free-tier and Request GPUs
1. Go to https://console.cloud.google.com (sign in with your login credentials if necessary)
2. You can upgrade from the free trial to a paid account through the Google Cloud Platform Console. Click the `Upgrade` button at the top of the page. If you do not see `Upgrade`, click `Free trial status` (denoted by a gift box) in the upper-right corner of the page and `Upgrade` will appear (you must be a Billing Administrator on the account to make this change).
3. In the top-left corner of the browser window, click the `Products and Services` tab with 3 horizontal lines
4. Hover your mouse over the `IAM & Admin` option and select `Quotas ` from the drop-down list. 
5. Click the box that says `Quota type` and select `All quotas` from the dropdown list. 
6. Click the box that says `Service` and select `Google Cloud Compute Engine API` from the dropdown list. 
7. Click the box that says `Metric`, click the option `None` to deselect all services, type `GPU` into the text field, and select `NVidia K80 GPUs`. Lastly, select a `Region` from the dropdown list. This region should be either `us-east1-c` or `us-east1-d` as these are the only 2 regions in the east that have GPUs available. For a comprehensive list of all the regions where GPUs are available, see https://cloud.google.com/compute/docs/gpus/. 
8. Now check the box next to NVidia K80 GPUs under Service and click `EDIT QUOTAS` at the top of the page. 
9. Enter your name, email, and phone number in the new sidebar.
10. In the first box, enter the number of GPUs you want your account quota raised to. 
11. In the second box, explain why you need the GPUs and then click `SUBMIT REQUEST` (you should hear back fairly quickly. I got an email saying I was approved in less than a minute.)

#### Step 2. Launch a new Google Compute Engine
1. In the top-left corner of the browser window, click the `Products and Services` tab with 3 horizontal lines
2. Scroll down to the `Compute` section of the sidebar list and click on `Compute Engine`.
3. Once on this page, click the `Create Instance` button on the top of the page. 
4. Change the name of your instance and then change your region to the region you requested a GPU quota increase in. 
5. Then, click `Customize` under the Machine type settings. Use the slider to adjust the number of Cores to 8. Click the small blue font that says `GPUs`and specify 1 NVidia Tesla K80 GPU.
6. Under `Boot disk`, select either the `Ubuntu 14.04 LTS` or `Ubuntu 16.04 LTS` as your OS image since these are the 2 boot disks we have created shell scripts for. At the bottom of the screen, make sure to change `Size` to 75 GB. 
7. Under `Identity and API access` select the box that says `Allow full access to all Cloud APIs`.
8. Under `Firewall` select both boxes to `Allow HTTP Traffic` and `Allow HTTPS Traffic`
9. Click `Create` at the bottom when you are done customizing your instance. 
The circle next to your instance will light up green when it is done being setup

#### Step 3. Connect to your new Compute Engine instance and update the software
This section follows the Linux/Mac OS instructions at https://cloud.google.com/compute/docs/instances/adding-removing-ssh-keys. However, this link also provides instructions for Windows users. 
1. Open a terminal on your workstation and use the `ssh-keygen` command to generate a new key pair. Specify the `-C` flag to add a comment with your username (the username portion is not required).
```
$ ssh-keygen -t rsa -f ~/.ssh/[KEY_FILENAME] -C [USERNAME]
```
where: 
- `[KEY_FILENAME]` is the name that you want to use for your SSH key files. For example, a filename of my-ssh-key generates a private key file named my-ssh-key and a public key file named my-ssh-key.pub.
- `[USERNAME]` is the user for whom you will apply this SSH key pair.
2. Restrict access to your private key so that only you can read it and nobody can write to it.
```
$ chmod 400 ~/.ssh/[KEY_FILENAME]
```
3. We are quickly going to add a project-wide public SSH key so that we do not need to specify a key everytime we make a new instance. In the top-left corner of the browser window, click the `Products and Services` tab with 3 horizontal lines. Hover your mouse over the `Compute Engine` option and select `Metadata` from the drop-down list. Under `SSH Keys`, click `Add SSH Keys`. An empty textbox should appear. We need to copy and paste the contents of our public key into this box. To find the contents of the public key, we can type the following command into the terminal:
```
$ less ~/.ssh/[KEY_FILENAME].pub
```
The contents will appear in the terminal. Copy and paste them into the textbox and click `Save` at the bottom of the page. 
4. Now connect to your instance via SSH or PuTTY: 
```
$ ssh -i ~/.ssh[KEY_FILENAME] [USERNAME]@[EXTERNAL IP OF COMPUTE ENGINE]
```
An example of the command I use to ssh into my instance is as follows: 
```
$ ssh -i ~/.ssh/google mike@35.202.4.235
```
The external IP address of your compute engine can be copied and pasted straight from the Compute Engine dashboard under the instance you have running. 

#### Step 4. Download and run the install shell script
Assuming you have chosen Ubuntu 16.04, the steps are as follows:
```
$ sudo apt-get install git -y
$ git clone https://github.com/amir-jafari/Cloud-Computing.git
$ cd Cloud-Computing/Deep\ Learning\ Kit\ Installation/Shell\ Script\ Installation/Ubuntu\ 16.04\ -\ Vritual\ Python/
$ mv install-16-04-final.sh ~
$ cd ~
$ chmod +x install-16-04-final.sh
$ sudo ./install-16-04-final.sh <netid or username of ssh key>
```
This process should take anywhere from 30-45 minutes. There are one or two places where you will be prompted for an answer. Always say yes. 

#### Step 5. Testing to see that all the software was installed correctly
The first thing we are going to do is clean up from the install by deleting all our downloads and install scripts:
```
$ rm -rf cuda-repo-ubuntu1604-8-0-local-ga2_8.0.61-1_amd64.deb cudnn-8.0-linux-x64-v6.0.tgz pycharm-community_2016.3-mm1_all.deb ZeroBraneStudioEduPack-1.60-linux.sh install-16-04-final.sh Cloud-Computing
```
The first thing we test is `NVIDIA CUDA Toolkit`. We can verify the driver version by running the following command: 
```
$ cat /proc/driver/nvidia/version
```
We should get output that looks like this:
```
NVRM version: NVIDIA UNIX x86_64 Kernel Module  375.66  Mon May  1 15:29:16 PDT 2017
GCC version:  gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.4)
```
We can also check the NVIDIA System Management Interface by typing:
```
$ nvidia-smi
```
and we should see something like this: 
```
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 375.66                 Driver Version: 375.66                    |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  Tesla K80           Off  | 0000:00:04.0     Off |                    0 |
| N/A   35C    P0    79W / 149W |      0MiB / 11439MiB |    100%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID  Type  Process name                               Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```
We need to install the NVIDIA Developer toolkit to check out our CUDA Compiler: 
```
sudo apt-get install nvidia-cuda-toolkit -y
```
Now, we run 
```
nvcc -V
```
and should see something similar to 
```
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2015 NVIDIA Corporation
Built on Tue_Aug_11_14:27:32_CDT_2015
Cuda compilation tools, release 7.5, V7.5.17
```
In order to check that the installation was successful we are going to compile the samples & run the device query script in the CUDA samples directory:
```
cd /usr/local/cuda/samples
sudo make
```
Do not worry if you get an error part of the way through the `sudo make` command as it has to do with things being recently deprecated. Now, go find the `deviceQuery` script and run it:
```
cd /bin/x86_64/linux/release
sudo ./deviceQuery
```
We should get output that looks like this: 
```
./deviceQuery Starting...

 CUDA Device Query (Runtime API) version (CUDART static linking)

Detected 1 CUDA Capable device(s)

Device 0: "Tesla K80"
  CUDA Driver Version / Runtime Version          8.0 / 8.0
  CUDA Capability Major/Minor version number:    3.7
  Total amount of global memory:                 11440 MBytes (11995578368 bytes)
  (13) Multiprocessors, (192) CUDA Cores/MP:     2496 CUDA Cores
  GPU Max Clock rate:                            824 MHz (0.82 GHz)
  Memory Clock rate:                             2505 Mhz
  Memory Bus Width:                              384-bit
  L2 Cache Size:                                 1572864 bytes
  Maximum Texture Dimension Size (x,y,z)         1D=(65536), 2D=(65536, 65536), 3D=(4096, 4096, 4096)
  Maximum Layered 1D Texture Size, (num) layers  1D=(16384), 2048 layers
  Maximum Layered 2D Texture Size, (num) layers  2D=(16384, 16384), 2048 layers
  Total amount of constant memory:               65536 bytes
  Total amount of shared memory per block:       49152 bytes
  Total number of registers available per block: 65536
  Warp size:                                     32
  Maximum number of threads per multiprocessor:  2048
  Maximum number of threads per block:           1024
  Max dimension size of a thread block (x,y,z): (1024, 1024, 64)
  Max dimension size of a grid size    (x,y,z): (2147483647, 65535, 65535)
  Maximum memory pitch:                          2147483647 bytes
  Texture alignment:                             512 bytes
  Concurrent copy and kernel execution:          Yes with 2 copy engine(s)
  Run time limit on kernels:                     No
  Integrated GPU sharing Host Memory:            No
  Support host page-locked memory mapping:       Yes
  Alignment requirement for Surfaces:            Yes
  Device has ECC support:                        Enabled
  Device supports Unified Addressing (UVA):      Yes
  Device PCI Domain ID / Bus ID / location ID:   0 / 0 / 4
  Compute Mode:
     < Default (multiple host threads can use ::cudaSetDevice() with device simultaneously) >

deviceQuery, CUDA Driver = CUDART, CUDA Driver Version = 8.0, CUDA Runtime Version = 8.0, NumDevs = 1, Device0 = Tesla K80
Result = PASS
```
We are mostly concerned with the last 2 lines that confirm the CUDA Driver Version, GPU Device, and that we passed the test! Since it is a really quick test, let's also double-check our bandwidth:
```
sudo ./bandwidthTest
```
and we should see: 
```
[CUDA Bandwidth Test] - Starting...
Running on...

 Device 0: Tesla K80
 Quick Mode

 Host to Device Bandwidth, 1 Device(s)
 PINNED Memory Transfers
   Transfer Size (Bytes)	Bandwidth(MB/s)
   33554432			9048.7

 Device to Host Bandwidth, 1 Device(s)
 PINNED Memory Transfers
   Transfer Size (Bytes)	Bandwidth(MB/s)
   33554432			10068.7

 Device to Device Bandwidth, 1 Device(s)
 PINNED Memory Transfers
   Transfer Size (Bytes)	Bandwidth(MB/s)
   33554432			156582.5

Result = PASS

NOTE: The CUDA Samples are not meant for performance measurements. Results may vary when GPU Boost is enabled.
```
We passed both tests, so it is time to check out if cuDNN is set up correctly. 


*Note: The instructions up to this point are the only ones required to get up and running with a fully-functional deep learning environment on a Google Cloud Compute Engine. The rest of this manual details the steps necessary to configure a desktop environment for your deep learning virtual machine. These steps are completely unnecessary, but some may find it useful. In fact, I wrote this section for myself because I had little-to-no experience in Linux when I started this manual.* 

#### Step 6. Install Google Chrome and enable graphics
1. Install Google Chrome using the following commands:
```
$ wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
$ sudo sh -c 'echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google-chrome.list'
$ sudo apt-get update
$ sudo apt-get install google-chrome-stable -y
```
2. Type `exit` in your command line to close the connection: 
```
mike@test-graphics:~$ exit
logout
Connection to 35.184.107.81 closed.
Michaels-MacBook-Pro-16:~ michaelarango$
```
3. If you are on a Mac, you need to open up XQuartz so that we can enable graphics for our compute engine. If you do not already have it installed, you can get it at https://www.xquartz.org. Once you have it installed, make sure it is running in the background. You may need to restart your computer for the changes to take effect. 
4. SSH back into your instance with an added command: 
```
$ ssh -Y -i ~/.ssh/google mike@35.184.107.81
```
5. Now that you are logged back into your instance, start up google chrome: 
```
google-chrome-stable
```
It will most likely take a little while for a small box to pop up asking if you want to make Chrome your default browser. Accept the prompt. Then wait for a full browser window to pop up and exit out of it. 

#### Step 7. Installing a desktop environment
1. Make sure to update and upgrade again: 
```
$ sudo apt-get update && sudo apt-get upgrade -y
```
2. Install the default Unity Desktop 
```
$ sudo apt-get install ubuntu-desktop gnome-panel gnome-settings-daemon metacity nautilus gnome-terminal -y
```
This may take about 5 minutes so it is the perfect time to go create our firewall rule so we can actually remote into the desktop once it is set up. 

3. Go to the Google Cloud console, open up the sidebar and select `VPC Network`. 
4. From this screen select `Firewall Rules` from the sidebar. 
5. Click `CREATE FIREWALL RULE`
6. Name the rule `vnc-server`, create a target tag with the same name, set the IP ranges to `0.0.0.0/0`, and select `Allow All` under Protocols and Ports. 
7. Now go back to the `Compute Engine` screen and select our running instance. Edit the instance and add the new `vnc-server` tag under network tags. 
8. Now let's go back to the terminal and install more desktop goodies: 
```
$ sudo apt-get install gnome-shell -y 
$ sudo apt-get install ubuntu-gnome-desktop -y
$ sudo apt-get install gnome-core -y 
$ sudo apt-get install adwaita-icon-theme-full adwaita-icon-theme
$ gsettings get org.gnome.metacity theme
$ gsettings set org.gnome.metacity theme 'Adwaita'
```
If a purplish-pink screen pops up select the `OK` prompt and then select `lightdm` from the list of 2 choices after. While this is loading, go ahead and install VNC Viewer on your local desktop from the provided link https://www.realvnc.com/en/connect/download/viewer/ . 

#### Step 8. Setting up Desktop Environment
1. I'm lazy and want to be able to copy and paste between my local machine and the VNC machine so: 
```
$ sudo apt-get install autocutsel
```
2. Install a vncserver client in your virtual machine:
```
$ sudo apt-get install tightvncserver
```
Sometimes the tightvncserver install does not include a file that we need so let's go ahead and create it anyway: 
```
$ touch ~/.Xresources
```
3. Set your VNC server password
```
$ vncserver
```
This will prompt you for a password less than 8 characters. Enter a password and verify it. Say no when asked if you want to make a read-only password. 
4. Edit the newly-created startup file.
As it is now, it will only give us a gray screen, so we need to set up the graphics. After you create your password, you should see the following. 
```
New 'X' desktop is instance-1:1

Creating default startup script /home/mike/.vnc/xstartup
Starting applications specified in /home/mike/.vnc/xstartup
Log file is /home/mike/.vnc/instance-1:1.log
```
Copy and paste the startup script path and open it up in `VIM`: 
```
$ vim /home/mike/.vnc/xstartup
```
In order to insert text in `VIM` you must press `i`. Copy and paste the following into your editor:
```
#!/bin/sh
autocutsel -fork
xrdb $HOME/.Xresources
xsetroot -solid grey
export XKL_XMODMAP_DISABLE=1
export XDG_CURRENT_DESKTOP="GNOME-Flashback:Unity"
export XDG_MENU_PREFIX="gnome-flashback-"
unset DBUS_SESSION_BUS_ADDRESS
gnome-session --session=gnome-flashback-metacity --disable-acceleration-check --debug &
```
Once you have finished editing the script, press the `esc` key, the `:` and then `x`. This is how you write and exit in `VIM`. 
5. Kill the VNC server: 
```
$ vncserver -kill :1
```
6. Start up VNC Server with resolution adjustments: 
```
$ vncserver -geometry 1280x800
```

#### Step 9. Logging into the desktop
1. Open up VNC Viewer on your local computer
2. Create a new connection by entering your VNC server info into the box. This should be of the form: 
`[EXTERNAL IP]:5901`
3. Click `Continue` on the page with the warning message.
4. Enter the password we made earlier. 
