# Run CARLA 0.9.9 on Google Cloud Platform
The following are steps to setup a virtual instance on GCP to run CARLA 0.9.9.

## Create a VM instance
Select a n1-standard-4 instance and add a NVIDIA GPU as shown in the image below. Add  at least a 100GB boot disk with Ubuntu 18.04.

![Step One](/images/step1.png)

## Network Tag
Add a network tag which will be used to setup firewall rules for the instance. We need to allow ingress on port 4000 for the desktop server (NoMachine). Leave other settings as default and click on “Create” to create the instance.

![Step Two](/images/step2.png)

## Firewall Rules
Navigate to Networking → VPC Network → Firewall to define a firewall rule for the instance. Click on “Create Firewall Rule” and fill in the following, leaving the rest as default
- Name: `allow-carla`
- Target Tags: `carla` the same tag we used for the vm instance 
- Source IP ranges: use the IP address of the client PC that will run the NoMachine client if its available, else use `0.0.0.0/0` to accept traffic from any IP.
- Protocols and Ports: Select `tcp` and provide `4000` as the port number
After creating the firewall rule it should appear in the list of firewall rules

## Install Cuda Driver
The vm instance we created has a GPU but the drivers are not installed. We need to install the drivers as per the instructions [here](https://cloud.google.com/compute/docs/gpus/install-drivers-gpu)
```
curl -O http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/cuda-repo-ubuntu1804_10.0.130-1_amd64.deb
sudo dpkg -i cuda-repo-ubuntu1804_10.0.130-1_amd64.deb
sudo apt-key adv --fetch-keys http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/7fa2af80.pub
```
```
sudo apt update
sudo apt install cuda
```
To confirm the installation of the driver, open a terminal and type `nvidia-smi`, you should see the details of the GPU.
## Install Desktop Server
Since the vm instance does not have display we need to setup a desktop server for a remote GUI.
```
cd /tmp
wget https://download.nomachine.com/download/6.11/Linux/nomachine_6.11.2_1_amd64.deb
sudo dpkg -i nomachine_6.11.2_1_amd64.deb
```
## Install Desktop Client
Download and install the NoMachine client [here](https://www.nomachine.com/)

## Optional Password
Setup an optional password for the default user: `sudo passwd ubuntu`

**From this point, make sure every command is run in the as the default ubuntu user**

## Server GUI
Install a GUI on the vm instance since it doesn’t have one by default. We will use the Ubuntu desktop.
```
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install -y ubuntu-desktop

```
## Install CARLA
As per the instructions [here](https://carla.readthedocs.io/en/latest/start_quickstart/) , run the code below to install CARLA v0.9.9
```
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 92635A407F7A020C sudo add-apt-repository "deb [arch=amd64 trusted=yes] http://dist.carla.org/carla-0.9.9/ all main"
sudo apt-get update
sudo apt-get install carla-simulator
 ```

## Install Anaconda
```
cd /tmp
curl -O https://repo.anaconda.com/archive/Anaconda3-2020.02-Linux-x86_64.sh
bash Anaconda3-2020.02-Linux-x86_64.sh
source ~/.bashrc
```
## Create a Conda Environment for CARLA
```
cd ~
conda create -n carla python=3.7
source activate carla
```
## Install Other Dependencies
```
pip install pygame numpy
```
## We Need to Fix Something Before We Proceed
Navigate to 
```
cd /opt/carla-simulator/PythonAPI/carla/dist
```
where you will find two .egg files: one for Python 2.7 and the other for Python 3.7. Extract the one for Python 3.7 and navigate to `carla-0.9.9-py3.7-linux-x86_64`. Create a new file and name it `setup.py` and paste the following code in it.
```
from distutils.core import setup
setup(name='carla',
      version='0.9.9',
      py_modules=['carla'],
      )
```
Then run the following commands to install the carla module for Python 3.7
```
pip install -e /opt/carla-simulator/PythonAPI/carla/dist/carla-0.9.9-py3.7-linux-x86_64
```
To confirm that everything is fine up to this point, open a terminal and run `python` then try doing `import carla`, if everything works then you can proceed.

## Reboot and Make a Connection from NoMachine Client
``` 
sudo reboot 
```
Open the NoMachine client and initiate a connection to the instance using its external IP address. Leave every other setting as default.

![NoMachine Client](/images/step4.png)

When you are prompted for username and password, use that of the default ubuntu account. Username: `ubuntu`, password: `your_password`

At this point everything is set, we can now start the CARLA server and run an example simulation.

## Run CARLA Server
Open Ununtu terminal and type in the code:
```
cd /opt/carla-simulator/bin
SDL_VIDEODRIVER=offscreen ./CarlaUE4.sh
```
Open another terminal and type in the following
```
conda activate carla
cd /opt/carla-simulator/PythonAPI/examples
python manual_control.py
```
You should see something like below

![CARLA Simulator](/images/step5.png)

# Congratulations! You can now begin to run your simulations.
