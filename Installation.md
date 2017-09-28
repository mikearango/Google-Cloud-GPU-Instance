## Setting up a Deep Learning compute instance on Google Cloud Platform

*Note: In most places where the user will be prompted to answer yes/no to a prompt, we add `-y` to the end of the command so it will automatically answer yes to all prompts. However, if you run into a case where it prompts you to answer, then always answer in the affirmative.*

#### Step 1. Upgrade Google Cloud Account from the free-tier and Request GPUs
1. Go to https://console.cloud.google.com (sign in with your login credentials if necessary)
2. You can upgrade from the free trial to a paid account through the Google Cloud Platform Console. Click the `Upgrade` button at the top of the page. If you do not see `Upgrade`, click `Free trial status` (denoted by a gift box) in the upper-right corner of the page and `Upgrade` will appear (you must be a Billing Administrator on the account to make this change).
3. In the top-left corner of the browser window, click the `Products and Services` tab with 3 horizontal lines
4. Hover your mouse over the `IAM & Admin` option and select `Quotas ` from the drop-down list. 
5. Click the box that says `Quota type` and select `All quotas` from the dropdown list. 
6. Click the box that says `Service` and select `Google Cloud Compute Engine API` from the dropdown list. 
7. Click the box that says `Metric`, click the option `None` to deselect all services, type `GPU` into the text field, and select `NVidia K80 GPUs`. 
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
This section follows the Linux/Mac OS instructions at https://cloud.google.com/compute/docs/instances/adding-removing-ssh-keys.
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
3. Follow the instructions at the above link for your operating system on how to add or remove project-wide public SSH keys
4. Now connect to your instance via SSH or PuTTY: 
```
$ ssh -i ~/.ssh[KEY_FILENAME] [USERNAME]@[EXTERNAL IP OF COMPUTE ENGINE]
```
My command is as follows: 
```
$ ssh -i ~/.ssh/google mike@35.202.4.235
```
The external IP address of your compute engline can be copied and pasted straight from the Compute Engine dashboard under the instance you have running. 
5. Run the follow commands to update and upgrade the instance: 
```
$ sudo apt-get update && sudo apt-get upgrade -y
```
6. Install the GNU Compiler Collection: 
```
$ sudo apt install gcc -y
```
#### Step 4. Install Google Chrome and enable graphics
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

#### Step 5. Installing a desktop environment
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

#### Step 6. Setting up Desktop Environment
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
$ vncserver -geometry 1024x640
```

#### Step 7. Logging into the desktop
1. Open up VNC Viewer on your local computer
2. Create a new connection by entering your VNC server info into the box. This should be of the form: 
`[EXTERNAL IP]:5901`
3. Click `Continue` on the page with the warning message.
4. Enter the password we made earlier. 

#### Step 8. Install the GPU Drivers (CUDA) and test to make sure they are installed properly
1. Go to https://cloud.google.com/compute/docs/gpus/add-gpus#install-driver-script to find the specific instructions for manually installing GPU drivers. The commands in this section will be for Ubuntu 16.04 LTS. 
2. Use `curl` to download the CUDA installer and the `dpkg` command to add the repository to your system:
```
$ curl -O http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/cuda-repo-ubuntu1604_8.0.61-1_amd64.deb
$ sudo dpkg -i cuda-repo-ubuntu1604_8.0.61-1_amd64.deb
```
3. Update the package lists:
```
$ sudo apt-get update -y
```
4. Install CUDA, which includes the NVIDIA driver.
```
$ sudo apt-get install cuda -y
```
5. After the driver finishes installing, verify that the driver installed and initialized properly.
```
$ nvidia-smi
Tue Aug  8 03:47:36 2017       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 375.66                 Driver Version: 375.66                    |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  Tesla K80           Off  | 0000:00:04.0     Off |                    0 |
| N/A   34C    P0    56W / 149W |     15MiB / 11439MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID  Type  Process name                               Usage      |
|=============================================================================|
|    0      2135    G   /usr/lib/xorg/Xorg                              15MiB |
+-----------------------------------------------------------------------------+
```

#### Step 9. Install and Test cudaNN 
CudaNN is a GPU-accelerated library of primitives for deep neural networks. This library provides highly tuned implementations for standard routines such as forward and backward convolution, pooling, normalization, and activation layers.
1. First visit https://developer.nvidia.com/rdp/cudnn-download and create a NVIDIA Developer Program account. 
2. Next, download the Linux library for Cuda 8.0 in your newly installed Chrome browser from this website:
https://developer.nvidia.com/rdp/cudnn-download

You must sign in using the credentials you created in Part 1.

3. After downloading completes, return to the terminal and unzip the file.
```
$ cd ~
$ cd Downloads
$ tar -zxf cudnn-8.0-linux-x64-v7.tgz
```

4. Copy the following files into the Cuda Toolkit Directory
```
$ ubuntu@ip-172-31-33-60:~/Downloads$ cd cuda
$ ubuntu@ip-172-31-33-60:~/Downloads/cuda$ sudo cp lib64/* /usr/local/cuda/lib64/
$ ubuntu@ip-172-31-33-60:~/Downloads/cuda$ sudo cp include/* /usr/local/cuda/include/
```

5. Test the installation:
```
cat /usr/local/cuda/include/cudnn.h | grep CUDNN_MAJOR -A 2    
```

### Step 5: Install Python and it's related libraries
1. Install Python 3
```
$ sudo apt-get install python3-pip
```
Test if Python 3 is installed: ```python3 --version```.

2. Install the following python libraries:
```
$ sudo pip3 install --upgrade pip
$ sudo apt-get install python3-tk
$ sudo pip3 install numpy
$ sudo pip3 install seaborn
$ sudo pip3 install pandas
$ sudo pip3 install sklearn
$ sudo pip3 install matplotlib
$ sudo pip3 install keras
$ sudo pip3 install theano
```

### Step 6. Install Tensorflow
```
$ sudo pip3 install tensorflow
```

Run a short TensorFlow program to see if it's working:
```
vim tftest.py
```
```python
# Python
import tensorflow as tf
hello = tf.constant('Hello, TensorFlow!')
sess = tf.Session()
print(sess.run(hello))
```
```
python3 tftest.py
```
The system should output the following:
```
Hello, TensorFlow!
```

### Step 7. Install Caffe
This instructions are based on this helpful resource: https://github.com/BVLC/caffe/wiki/Install-Caffe-on-EC2-from-scratch-(Ubuntu,-CUDA-7,-cuDNN-3)

1. Install the dependencies with all of these installs:
```
sudo apt-get install -y libprotobuf-dev libleveldb-dev libsnappy-dev libopencv-dev libboost-all-dev libhdf5-serial-dev protobuf-compiler gfortran libjpeg62 libfreeimage-dev libatlas-base-dev git python-dev  libgoogle-glog-dev libbz2-dev libxml2-dev libxslt-dev libffi-dev libssl-dev libgflags-dev liblmdb-dev python-yaml
```
Then run:
```
$ sudo easy_install pillow
```
2. Clone Caffe.git
```
$ cd ~
$ git clone https://github.com/BVLC/caffe.git
```
3. Cd to Caffe folder and run the follwing:
```
$ cat python/requirements.txt | xargs -L 1 sudo pip install
```

4. Next, copy the Makefile.config.example to create our own Makefile.config
```
$ cp Makefile.config.example Makefile.config
$ vi Makefile.config
```
Uncomment the line: USE_CUDNN := 1
Make sure the CUDA_DIR correctly points to our CUDA installation.

5. Now we build Caffe. Set X to the number of CPU threads (or cores) on your machine. For me, this was 8.

```
$ sudo apt-get install htop
```
```
$ make pycaffe -jX
$ make all -jX
$ make test -jX
```
Now test Caffe:
```
$ ./data/mnist/get_mnist.sh
```

### Step 8. Install Torch
This instructions are based on this helpful resource:
http://torch.ch/docs/getting-started.html

Run the following in the terminal:
```
$ git clone https://github.com/torch/distro.git ~/torch --recursive
$ cd ~/torch; bash install-deps;
$ ./install.sh
```
The last command will take some time to install.

Then run:
```
source ~/.bashrc
```
Test if it's working:
```
$ th
```
If successful, Torch will start:
```

  ______             __   |  Torch7                                   
 /_  __/__  ________/ /   |  Scientific computing for Lua.         
  / / / _ \/ __/ __/ _ \  |                                           
 /_/  \___/_/  \__/_//_/  |  https://github.com/torch   
                          |  http://torch.ch       
```

### Step 9. Install pycharm
1. Run ```google-chrome``` to open the Chrome browser

2. Navigate to https://www.jetbrains.com/pycharm/download/download-thanks.html?platform=linux&code=PCC and download the Linux version of PyCharm Community

3. Cd to Downloads folder in the terminal and unzip:
```
$ cd ~
$ cd Downloads
$ tar -zxf pycharm-community-2017.2.tar.gz
```

4. To open PyCharm community:
```
$ cd pycharm-community-2017.2
$ cd bin
$ cd ./pycharm.sh
```
