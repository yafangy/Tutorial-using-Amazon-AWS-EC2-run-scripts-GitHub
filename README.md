# Tutorial: using Amazon AWS EC2 to run scripts from a GitHub repo
The goal of this tutorial is to (1) configure the hardware and software of an Amazon EC2 instance that we then (2) create connect to this instance in order to clone a Git repository, then (3) run a python script (should be applicable to other coding languages). This tutorial has been prepared considering that you are using Linux or Mac OS to connect to the
remote instance.

More will follow on how to set up the system to use OpenACC in
GPU-base accelerated computing instances on AWS.

## Create a repo in GitHub and use git
If you have a local directory containing scripts you want to run with Amazon EC2, simply create a GitHub repo for it.
This should be familiar to most GitHub users, so I will not dwell on this topic much.
See: [https://help.github.com/articles/create-a-repo/]

You can put your script on your GitHub repo. We will focus on how to utilize AWS EC2 to execute the script.

## Create AWS account

If you don’t have an Amazon AWS account, create one with this link: [https://aws.amazon.com/account/]

If you are a student, you can create an education account by applying through this link: [https://aws.amazon.com/education/awseducate/apply/]

A few useful links for people who are new to Amazon AWS:
- If you are using windows to connect to the remote instance, I recommend you read: [http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/putty.html]
- AWS guidelines to create IAM users and VPCs: [http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/get-set-up-for-amazon-ec2.html]
- I recommend you first read this introductory Amazon EC2 guide that presents the basic concepts: instances, AMIs, security groups, root devices, regions and availability zones. See: [http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html]
- I recommend you firstly read this introductory Amazon S3 guide that presents the basic concepts: buckets and objects. See: [http://docs.aws.amazon.com/AmazonS3/latest/dev/Introduction.html]

## Get started with Amazon EC2
Amazon Elastic Compute Cloud (Amazon EC2) provides scalable computing capacity in the Amazon Web Services (AWS) cloud.

### Create an IAM user for yourself and add the user to an Administrators group
Follow the instructions here: [https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/get-set-up-for-amazon-ec2.html#create-an-iam-user]

Basically you go to the IAM console at [https://console.aws.amazon.com/iam/], and create group and user there. Be sure that it is an Administrator group.

Important: follow this to add “allowEC2callAWSservice” to IAM instance role [https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-access.html]

When you create a user account, you can download a credential file containing the following information and save it to a local directory for your records. 

```
[Credentials]
aws_access_key_id = {ACCESS KEY ID}
aws_secret_access_key = {SECRET ACCESS KEY}
```

This can also be found in “security credentials” tab if you browse “users” and click the user name you created.

### Create a Key Pair
See this link: [https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/get-set-up-for-amazon-ec2.html#create-a-key-pair]
- Download the private key locally and copy it to the .ssh folder. In my case, and for illustration:

``` $ mv ~/Downloads/***.pem ~/.ssh/***.pem```

- Here \~ means HOME directory. If the .ssh directory does not exist, you can simply create it:

``` $ mkdir -p ~/.ssh```

- Change the permission of the file

``` $ chmod 600 ~/.ssh/***.pem ```

- Remember the key path and name.

### Launch a VM
We are going to launch an Amazon EBS-backed instance (meaning that the root volume is an EBS volume). We will let Amazon EC2 select an Availability Zone for us.
- Go to the EC2 dashboard and click “Launch Instance”.
- Step 1: Select “Ubuntu Server” as AMI (“Free Tier Eligible”)
- Step 2: Select “t2.micro” as instance type (“Free Tier Eligible”). It is important to select EBS backed instances for persistency.
- Step 3: Click “Next” in the bottom right corner to modify the configurations. Use default configurations for the rest of steps. It is highly recommended to revise and understand all options. (especially the “Configure Security Group” option. You can choose ‘custom’, ‘My IP’ or ‘anywhere’. ‘My IP’ is recommended). At the final step, click “Launch”.
- Please make sure policy “allowEC2callAWSservice” is in IAM instance role [https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-access.html] if you later plan to use this instance to run a command (method 1 below). In the “configure instance” tab -> IAM role -> select the created role. 
- Step 4: Select your key pair and “Launch Instance”
- Step 5: Go to “Running Instances” in EC2 Dashboard and wait for the VM to be “running”.
- Step 6: Familiarize yourself with the actions and the status and monitoring information provided by the dashboard.

### Login to the VM
After you launch your instance, you can connect to it and use it the way that you'd use a computer sitting in front of you. It can take a few minutes for the instance to be ready so that you can connect to it.
To connect to your Linux instance from a computer running Mac or Linux, you'll specify the .pem file to your SSH client with the -i option and the path to your private key. To connect to your Linux instance from a computer running Windows, you can use either MindTerm or PuTTY. If you plan to use PuTTY, you'll need to install it and use the following procedure to convert the .pem file to a .ppk file: [Connecting to Your Linux Instance from Windows Using PuTTY]

For Linux and Mac OS user:
- Select the instance, and then choose Connect.
- Execute the SSH command to login to your VM (choose “a standalone SSH client, and your public DNS and an example command code will appear on the screen).
Type in the following command in the terminal:
```
$ ssh -i ~/.ssh/***.pem ubuntu@[Your Public DNS]
```
You should be connected to the EC2 instance

## Run a Python Script on GitHub using Amazon AWS EC2

### Method 1: Using Amazon EC2 Systems Manager (not recommended)
If you want to run a simple python code in the command window, follow this tutorial: [https://docs.aws.amazon.com/systems-manager/latest/userguide/integration-remote-scripts.html#integration-github-python]
1. Open the Amazon EC2 console at [https://console.aws.amazon.com/ec2/]
2. In the navigation pane, choose Run Command, and then choose Run a command.
3. In the Document list, choose AWS-RunRemoteScript.
4. In the Select Targets by section, choose an option and select the instances where you want to download and run the script.
- In the Run a command page, after you choose an SSM document to run and select Manually selecting instances in the Targets section, a list is displayed of instances you can choose to run the command on. If an instance you expect to see is not listed, check this guide: [https://docs.aws.amazon.com/systems-manager/latest/userguide/troubleshooting-remote-commands.html?icmpid=docs\_ec2\_console]
- SSM Agent is installed, by default, on Amazon Linux base AMIs dated 2017.09 and later. SSM Agent is also installed, by default, on Amazon Linux 2, Ubuntu Server 16.04, and Ubuntu Server 18.04 LTS AMIs. If your instance satisfies the above criterion, it is very likely that your IAM instance role doesn’t contain policy “AmazonEC2RoleforSSM”.
- If it is related to your IAM instance role, follow this guide step by step:[https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-access.html]. You should be able to see the instance show up after you launch an instance with policy “AmazonEC2RoleforSSM”
5. In the Source Type list, choose GitHub
6. In the Source text box, type the required information to access the source in the following format:
``` 
{{“owner":"owner_name", "repository": "repository_name", "path": "path_to_scripts_or_directory", "tokenInfo":"{{ssm-secure:SecureString_parameter_name}}" }
```
7. In the Command Line field, type parameters for the script execution. Here is an example.
```
$ File.py argument-1 argument-2
```
8. For more, follow this guide: [https://docs.aws.amazon.com/systems-manager/latest/userguide/integration-remote-scripts.html#integration-github-python] 
### Method 2: clone git to EC2 instance (recommended)
Partly adapted from this guide: [http://bw4sz.github.io/ec2/#v.-accessing-an-amazon-ec2-instance-in-the-browser]
#### Enabling SSH with your Git repository
We need to generate an SSH key (two files - a public key that you share with the world and a private key you keep safe) that we will associate with our Git account. This will allow us to clone our Git repository on an EC2 instance without having to manually type in your username and password or (worse yet) put your password in cleartext when using a script. 
Here we introduce how to generate an SSH key on your local directory and then copy to your EC2 instance later. You can also do it on your EC2 instance directly, but each time you generate an SSH key pair on your new instance, you need to register the new key in GitHub every time.
1. In your local terminal, create an SSH key, substituting your email address.
```
ssh-keygen -t rsa -b 4096 -C [your email address]
```
2. Save the key to the default directory, \~/.ssh
3. Enter a pass-phrase of your choice.
4. Check that the public and private key are in /.ssh by going to the directory and typing “ls -l id\_rsa\*”. You should see two files, the public key named “id\_rsa.pub” and the private key named “id\_rsa”
5. From the terminal, make sure this private key is not publicly viewable.
```
$ chmod 600 ~/.ssh/id_rsa
```
6. Add your SSH private key to the ssh-agent and store your passphrase in the keychain.
```
$ ssh-add -k ~/.ssh/id_rsa
```
7. Go to the settings under your GitHub account and then click SSH keys and New SSH key
8. In terminal copy your public key to the clipboard. Or show on the EC2 terminal:
''$ pbcopy < ~/.ssh/id_rsa.pub   # copy to clipboard
''$ cat ~/.ssh/id_rsa.pub  # If you prefer appear on screen
9. Paste this into the key box on GitHub and click save. This key is available to ALL your Git repositories.
10. Sometimes you need to move the public key to “/.ssh/authorized\_keys” to make the public key to work in LINUX. 
```
$ mkdir ~/.ssh  # if you don't have /.ssh/ folder
$ chmod 700 ~/.ssh
$ touch ~/.ssh/authorized_keys
$ chmod 600 ~/.ssh/authorized_keys
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```
11. Follow this article [https://help.github.com/articles/error-permission-denied-publickey/] to see if the key works and debug.

### Cloning a Git repository on your Amazon EC2 instance
1. Assume you already have a EC2 instance launched. To copy your public key to EC2 instance, on your local terminal:
```
$ cat ~/.ssh/id_rsa.pub | ssh -i ~/.ssh/your_pem.pem ubuntu@your_dns "cat >> .ssh/authorized_keys"
```
- If “.ssh/authorized\_keys" doesn’t exist, log in to your EC2 instance and create the directory:
```
$ ssh -i ~/.ssh/***.pem ubuntu@[Your Public DNS]
$ mkdir ~/.ssh
$ chmod 700 ~/.ssh
$ touch ~/.ssh/authorized_keys
$ chmod 600 ~/.ssh/authorized_keys
$ exit  # exit EC2 terminal, this will not stop your instance
```
2. Secure copy our GitHub private key to the EC2 instance. The “UserKnownHostsFile” and “StrictHostKeyChecking” options removes the host verification prompt (the computer asks you to type yes or no to continue). Removing this is essential if we want to run these actions in a script.
```
$ scp -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no -i ~/.ssh/your_pem.pem ~/.ssh/id_rsa ubuntu@your_dns:~/.ssh/
```
3. Go to the repository you wish to clone on GitHub account and copy the _SSH link_ for the master branch under the code tab.
4. Log in to your EC2 instance
```$ ssh -i ~/.ssh/***.pem ubuntu@[Your Public DNS]```
5. In the terminal, move to the home directory and then clone the master branch to this Amazon EC2 instance. Now your Git repository is cloned to the home directory of your Amazon EC2 instance.
```
$ cd ~  # this equals to: cd /home/ubuntu/
```
- If I do `$cd /home`, it will show “could not create work tree dir ‘your git file’: Permission denied” when I do `git clone`.
```
$ git clone git@github.com:***/***.git
```
The files should be cloned to your EC2 instance.
### Run the script and enjoy!!!
You can verify whether python 3 is installed:
```$ python3 --version```
Now you can try to install pip3.
```
$ sudo apt install python3-pip
```
Then you can install python packages, for example:
```
$ pip3 install http://download.pytorch.org/whl/cpu/torch-0.4.1-cp35-cp35m-linux_x86_64.whl
$ pip3 install torchvision
```
cd to your directory and run your python script:
```
$ python3 your_script.py
```
Enjoy!!!

**Remember to terminate your instance when you are done!!!**

## Optional
### Install AWS CLI
See this link to install AWS Command Line Interface ([https://docs.aws.amazon.com/cli/latest/userguide/installing.html]) and follow the steps in this guide. 
I use Anaconda/spider on Mac OS, and I found pip3 doesn’t install awscli correctly, but conda does:
'' $ conda install -c conda-forge awscli
If you are not familiar with conda, see this link for installing conda: [https://conda.io/docs/user-guide/install/index.html]

You can find your credentials by following the instructions in this link: [https://aws.amazon.com/articles/getting-started-with-aws-and-python/]

Instead of placing this file at /etc/boto.cfg as suggested by the article, you can create the credential file using AWS CLI command. Just type the following in command window ([AWS CLI] need to be installed):
```
$ aws configure
```
Fill in the info:
``` 
$ AWS Access Key ID [None]: = YOUR_ACCESS_KEY
$ AWS Secret Access Key [None]: = YOUR_SECRET_KEY
$ Default region name [None]: us-east-2
$ Default output format [None]: 
```
### Install boto3
See boto3 quick start: [http://boto3.readthedocs.io/en/latest/guide/quickstart.html]

This conda command also works for me:
```
$ conda install -c anaconda boto3
```
