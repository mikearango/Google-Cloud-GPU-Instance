## DATS 6450
### Setting up a deep learning compute instance with Google Cloud Platform

This document details the installation process for a Google Cloud Virtual Machine with GPUs and deep learning frameworks. 
To see how the same can be done in AWS, see [Elizabeth Rychlinski](https://github.com/ERych/CC-Summer17/blob/master/Project-Phase1.md)'s
Github repository.

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
4. Change the name of your instance if you want to and then click `Customize` under the Machine type settings. 
5. Click the small blue font that says `GPUs`and specify the number of GPUs needed for the project. Then, adjust your Cores and Memory accordingly. (I specified 1 NVidia Tesla K80 GPU, 8 cores, and 30 GB memory)
6. Under Boot disk, you can change which OS image you want to use. (I used Ubuntu 16.04 LTS)
7. If you plan on using Google API's for your project, then select the box under `Identity and API access` that says `Allow full access to all Cloud APIs`.
8. Click `Create` at the bottom when you are done customizing your instance. 
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

#### Step 4. Install the GPU Drivers (CUDA) and test to make sure they are installed properly
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

#### Step 5. Install Google Chrome and enable graphics
1. Install Google Chrome using the following commands:
```
$ wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
$ sudo sh -c 'echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google-chrome.list'
$ sudo apt-get update
$ sudo apt-get install google-chrome-stable
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
$ sudo apt-get update && sudo apt-get upgrade
```
2. Install the default Unity Desktop 
```
$ sudo apt-get install ubuntu-desktop gnome-panel gnome-settings-daemon metacity nautilus gnome-terminal
```
This may take about 5 minutes...


#### Step 6. Install the GPU Drivers (CUDA) and test to make sure they are installed properly




