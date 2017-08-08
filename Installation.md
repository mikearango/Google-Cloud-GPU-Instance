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
1. Click `SSH` where it says Connect on your instance and the Google Cloud shell will pop up and automatically connect you via SSH. 
2. Run the follow commands to update and upgrade the instance: 
```
$ sudo apt-get update && sudo apt-get upgrade -y
```
3. Install the GNU Compiler Collection: 
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
```
