# DATS 6450

This document details the installation process for a Google Cloud Virtual Machine with GPUs and deep learning frameworks. 
To see how the same can be done in AWS, see [Elizabeth Rychlinski](https://github.com/ERych/CC-Summer17/blob/master/Project-Phase1.md)'s
Github repository.

*Setting up a deep learning compute instance with Google Cloud Platform*

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
6. Under Boot disk, you can change which OS image you want to use. (I used the default Debian GNU/Linux 9 (stretch))
7. Click `Create` at the bottom when you are done customizing your instance. 
The circle next to your instance will light up green when it is done being setup

#### Step 3. Connect to your new Compute Engine instance
1. Click `SSH` where it says Connect on your instance and the Google Cloud shell will pop up and automatically connect you via SSH. 
