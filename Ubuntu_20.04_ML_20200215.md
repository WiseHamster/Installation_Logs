# Installation log for Ubuntu 20.04 LTS + CUDA 11.0 + Tensorflow + PyTorch

Preliminaries:
* A clean installation of Ubuntu 20.04 LTS Desktop was used
* A setup was done for user _neuroclass_  with  home directory _/home/neuroclass_
* Three virtualenvs will be created: 
  * _ml_ for "standard" DA tasks
  * _tf_ for tensorflow 
  * _torch_ for pytorch
* To simplify further updates of the system I prefered to use _apt_ and _snap_ as much as it it possiple

### Install some base packages
```
$ sudo apt update
$ sudo apt upgrade
$ sudo apt install vim mc git cmake tmux openssh-server build-essential
```

### Install 
```
$ sudo snap install vlc
$ sudo snap install vscode
```

### Create Python virtual environment
```
$ mkdir .virtualenv
$ sudo apt install python3-pip
$ pip3 --version
$ pip3 install virtualenv
$ pip3 install virtualenvwrapper
```

### Add following lines to the end of _~/.bashrc_

```
# Virtualenvwrapper settings:
export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3
export WORKON_HOME=$HOME/.virtualenvs
export VIRTUALENVWRAPPER_VIRTUALENV=/home/neuroclass/.local/bin/virtualenv
source ~/.local/bin/virtualenvwrapper.sh
```

```
$ source ~/.bashrc
```

### Install standard Python packages into virtualenvs

```
$ mkvirtualenv ml
$ pip install numpy matplotlib pandas seaborn tqdm
$ pip install pillow scipy scikit-learn scikit-image networkx
$ pip install jupyter jupyterlab
$ deactivate
```

### For _tf_ and _torch_ enviromnents I need Cython

```
$ mkvirtualenv tf
$ pip install numpy matplotlib pandas seaborn tqdm
$ pip install pillow scipy scikit-learn scikit-image networkx
$ pip install jupyter jupyterlab
$ pip install Cython
$ deactivate
```

#### ... do the same for the _torch_ virtual environment


### Setup jupyter for the remote access through HTTPS

Full guide - [Running a notebook server](https://jupyter-notebook.readthedocs.io/en/stable/public_server.html)

```
$ jupyter notebook --generate-config
$ jupyter notebook password
$ mkdir ~/.keys
$ openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout ~/.keys/mykey.key -out ~/.keys/mycert.pem
```
#### add following lines to  _~/.jupyter/jupyter_notebook_config.py_

```
# Set options for certfile, ip, password, and toggle off

# browser auto-opening

c.NotebookApp.certfile = u'/home/neuroclass/.keys/mycert.pem'
c.NotebookApp.keyfile = u'/home/neuroclass/.keys/mykey.key'

# Set ip to '*' to bind on all interfaces (ips) for the public server
c.NotebookApp.ip = '*'

# Copy hashed password from ~/.jupyter/jupyter_notebook_config.json
c.NotebookApp.password = u'sha1:bcd259ccf...<your hashed password here>'
c.NotebookApp.open_browser = False

# It is a good idea to set a known, fixed port for server access
c.NotebookApp.port = 9999
```
### Install Docker

Full guide - [How To Install and Use Docker on Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04)

```
$ sudo apt update
$ sudo apt install apt-transport-https ca-certificates curl software-properties-common
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
$ sudo apt update
$ apt-cache policy docker-ce
$ sudo apt install docker-ce
$ sudo systemctl status docker

sudo usermod -aG docker ${USER}
su - ${USER}
id -nG`

docker run hello-world
```

#### (Specific for that PC) Attach HDD for data

Full guide - [
How to properly automount a drive in Ubuntu Linux
](https://www.techrepublic.com/article/how-to-properly-automount-a-drive-in-ubuntu-linux/)

```
$ sudo mkdir /opt/data
$ sudo chown -R neuroclass:neuroclass /opt/data

$ sudo fdisk -l
$ sudo blkid

...
_/dev/sda: UUID="7770df55-5f10-49da-b6be-b6bb94ac3ab7" TYPE="ext4"_
...

`$ sudo echo "UUID=7770df55-5f10-49da-b6be-b6bb94ac3ab7 /opt/data ext4 nosuid,nodev,nofail,x-gvfs-show 0 0" >> /etc/fstab`
```

### Move docker images to the another place (HDD) in my case

Full guide - [How to move docker data directory to another location on Ubuntu](https://www.guguweb.com/2019/02/07/how-to-move-docker-data-directory-to-another-location-on-ubuntu/)

```
$ sudo service docker stop
$ sudo vim /etc/docker/daemon.json

{
   "data-root": "/opt/data/docker"             
}

$ sudo rsync -aP /var/lib/docker/  /opt/data/docker

$ sudo mv /var/lib/docker /var/lib/docker.old

$ sudo service docker start

# for the test purpose
$ docker run hello-world
```


### Install CUDA

#### Install nVidia driver

https://towardsdatascience.com/installing-multiple-cuda-cudnn-versions-in-ubuntu-fcb6aa5194e2

```
$ sudo lshw -C display

# install default / recommended
$ sudo ubuntu-drivers devices

# To install recommended 
$ sudo ubuntu-drivers autoinstall OR# To install specific distro

sudo apt install nvidia-driver-[version number]

# in my case
sudo apt install nvidia-driver-460

$ nvidia-smi
Wed Feb 24 23:06:08 2021       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 460.32.03    Driver Version: 460.32.03    CUDA Version: 11.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  GeForce GTX 107...  Off  | 00000000:03:00.0 Off |                  N/A |
|  0%   31C    P8    10W / 180W |    113MiB /  8117MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
|   1  GeForce GTX 107...  Off  | 00000000:04:00.0 Off |                  N/A |
| 33%   31C    P8     6W / 180W |     11MiB /  8119MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|    0   N/A  N/A      1140      G   /usr/lib/xorg/Xorg                 29MiB |
|    0   N/A  N/A      1640      G   /usr/lib/xorg/Xorg                 54MiB |
|    0   N/A  N/A      1767      G   /usr/bin/gnome-shell               10MiB |
|    1   N/A  N/A      1140      G   /usr/lib/xorg/Xorg                  4MiB |
|    1   N/A  N/A      1640      G   /usr/lib/xorg/Xorg                  4MiB |
+-----------------------------------------------------------------------------+
```
### Install nVidia CUDA 11.0

#### Add NVIDIA package repositories

```
$ wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-ubuntu2004.pin
$ sudo mv cuda-ubuntu2004.pin /etc/apt/preferences.d/cuda-repository-pin-600
$ sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/7fa2af80.pub
$ sudo add-apt-repository "deb https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/ /"
$ sudo apt-get update

$ cd /tmp
$ wget http://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu2004/x86_64/nvidia-machine-learning-repo-ubuntu2004_1.0.0-1_amd64.deb
$ sudo apt install ./nvidia-machine-learning-repo-ubuntu2004_1.0.0-1_amd64.deb
$ sudo apt-get update

$ apt-cache policy cuda
```
#### Install development and runtime libraries
```
$ sudo apt-get install --no-install-recommends \
    cuda-11-0 \
    libcudnn8=8.0.5.39-1+cuda11.0  \
    libcudnn8-dev=8.0.5.39-1+cuda11.0
```

### Install Tensorflow into venv

### Install Pytorch into venv

#### Install and test Tensorflow docker image

### Install CV2 with CUDA support

### Install dlib with CUDA support

### Install R language support

$ sudo apt install dirmngr gnupg apt-transport-https ca-certificates software-properties-common

$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E298A3A825C0D65DFD57CBB651716619E084DAB9
$ sudo add-apt-repository 'deb https://cloud.r-project.org/bin/linux/ubuntu focal-cran40/'

$ sudo apt install r-base

$ R --version
 version 3.4.4 (2018-03-15) -- "Someone to Lean On"
Copyright (C) 2018 The R Foundation for Statistical Computing
Platform: x86_64-pc-linux-gnu (64-bit)

R is free software and comes with ABSOLUTELY NO WARRANTY.
You are welcome to redistribute it under the terms of the
GNU General Public License versions 2 or 3.
For more information about these matters see
http://www.gnu.org/licenses/.



$ sudo apt-get install gdebi-core
$ wget https://download2.rstudio.org/server/bionic/amd64/rstudio-server-1.4.1106-amd64.deb
$ sudo gdebi rstudio-server-1.4.1106-amd64.deb


