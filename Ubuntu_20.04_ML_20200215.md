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
source ~/.bashrc
```

### Install standard Python packages into virtualenvs

```
$ mkvirtualenv ml
$ pip install numpy matplotlib pandas seaborn tqdm
$ pip install pillow scipy scikit-learn scikit-image networkx
$ pip install jupyter jupyterlab
$ deactivate
```

### For _tf_ and _torch_ enviromnents I nees Cython

```
$ mkvirtualenv tf
$ pip install numpy matplotlib pandas seaborn tqdm
$ pip install pillow scipy scikit-learn scikit-image networkx
$ pip install jupyter jupyterlab
$ pip install Cython
$ deactivate
```

#### ... do the same for _torch_ environment

### Setup jupyter for the remote access through HTTPS

Full guide - [Running a notebook server](https://jupyter-notebook.readthedocs.io/en/stable/public_server.html)

```
$ jupyter notebook --generate-config
$ jupyter notebook password
$ mkdir ~/.keys
$ openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout ~/.keys/mykey.key -out ~/.keys/mycert.pem

#### add following to the _~/.jupyter/jupyter_notebook_config.py_

> # Set options for certfile, ip, password, and toggle off

> # browser auto-opening
```
c.NotebookApp.certfile = u'/home/neuroclass/.keys/mycert.pem'
c.NotebookApp.keyfile = u'/home/neuroclass/.keys/mykey.key'
```

> # Set ip to '*' to bind on all interfaces (ips) for the public server
```
c.NotebookApp.ip = '*'
c.NotebookApp.password = u'sha1:bcd259ccf...<your hashed password here>'
c.NotebookApp.open_browser = False
```

> # It is a good idea to set a known, fixed port for server access
```
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

#### (Specific for that PC) Attach data HDD

Full guide - [
How to properly automount a drive in Ubuntu Linux
](https://www.techrepublic.com/article/how-to-properly-automount-a-drive-in-ubuntu-linux/)

`$ sudo mkdir /opt/data`
`$ sudo chown -R neuroclass:neuroclass /opt/data`

`$ sudo fdisk -l`
`$ sudo blkid`
```
...
_/dev/sda: UUID="7770df55-5f10-49da-b6be-b6bb94ac3ab7" TYPE="ext4"_
...
```
`$ sudo echo "UUID=7770df55-5f10-49da-b6be-b6bb94ac3ab7 /opt/data ext4 nosuid,nodev,nofail,x-gvfs-show 0 0" >> /etc/fstab`


### Move docker images to the another place (HDD)

Full guide - [How to move docker data directory to another location on Ubuntu](https://www.guguweb.com/2019/02/07/how-to-move-docker-data-directory-to-another-location-on-ubuntu/)

```
$ sudo service docker stop
$ sudo vim /etc/docker/daemon.json

{
   "data-root": "/opt/data/docker"             
}

$ sudo rsync -aP /var/lib/docker/  /opt/data/docker

$ sudo mv /var/lib/docker /var/lib/docker.old


sudo service docker start

# for the test purpose
docker run hello-world



### Install CUDA

> CUDA

https://towardsdatascience.com/installing-multiple-cuda-cudnn-versions-in-ubuntu-fcb6aa5194e2

`sudo lshw -C display`

# install default / recommended
`$ sudo ubuntu-drivers devices`

# To install recommended 
`$ sudo ubuntu-drivers autoinstall OR# To install specific distro`
`$ sudo apt install nvidia-driver-[version number]`

`$ nvidia-smi`

```

--- install CUDA 


# Add NVIDIA package repositories

wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-ubuntu2004.pin
sudo mv cuda-ubuntu2004.pin /etc/apt/preferences.d/cuda-repository-pin-600
sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/7fa2af80.pub
sudo add-apt-repository "deb https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/ /"
sudo apt-get update


wget http://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu2004/x86_64/nvidia-machine-learning-repo-ubuntu2004_1.0.0-1_amd64.deb
sudo apt install ./nvidia-machine-learning-repo-ubuntu2004_1.0.0-1_amd64.deb
sudo apt-get update

apt-cache policy cuda

# Install development and runtime libraries (~4GB)
```
sudo apt-get install --no-install-recommends \
    cuda-11-0 \
    libcudnn8=8.0.5.39-1+cuda11.0  \
    libcudnn8-dev=8.0.5.39-1+cuda11.0
```











