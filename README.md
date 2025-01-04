# TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8
**Date : 2025-01-05**  
Builds for Coral Edge TPU (M.2 PCI) on a dockerized Ubuntu 24.04.  
Ubuntu is a container of a QNAP NAS which has a Celeron CPU (x86_64 or k8).

I am just going to write the steps I followed thanks to the repo of @Feranick.  
Nothing would be possible without his work.

source:  [https://github.com/feranick/pycoral](https://github.com/feranick/pycoral)  
Big thanks to @Feranick the author of the repo. He saved lots of Coral users.

The files in this repo are just the results of @feranick pycoral compilation for:
* Ubuntu 24.04
* Python 3.12
* CPU x86_64 or k8
* Tensor Flow 2.17.1

This repo was inspired by [https://github.com/virnik0/pycoral_builds](https://github.com/virnik0/pycoral_builds)  
Thanks to @Feranick & @virnik0 and their instructions, I managed to reproduce the steps.  
The sole purpose of this repo is to possibly help.

At the very beginning: let's introduce the hardware.

## Hardware
On a QNAP NAS, I have the [Coral dual Edge TPU](https://coral.ai/products/m2-accelerator-dual-edgetpu) mounted on an [PCIe - Dual Edge TPU Adapter](https://github.com/magic-blue-smoke/Dual-Edge-TPU-Adapter). And this PCIe is my NAS and recognized by the OS.

## Docker Compose
In Container Station, I create a Ubuntu 24.04 container with a a fictitious MAC address (and my router will attribute a reserved IP address in my LAN).

```dockerfile
version: "3"

services:
  ubuntu:
    container_name: tpuubuntu2404  # pour python 3.12
    image: ubuntu:24.04
    mac_address: 02:42:AC:11:11:11
    volumes:
      - /share/Container/mes-app-data/ubuntu:/home/mimo
    restart: unless-stopped
    tty: true  # to keep container running
    devices:
      - /dev/apex_0:/dev/apex_0
      - /dev/apex_1:/dev/apex_1

networks:
  default:
    external: true
    name: monbridgeLAN
```

And the Bridge LAN was prevously created with this command:

```bash
$ docker network create -d qnet --opt=iface=bond0 --ipam-driver=qnet --ipam-opt=iface=bond0 \
      --subnet=192.168.0.0/24 --gateway=192.168.0.1 monbridgeLAN
```

## SSH to the container
In the Bash console of the container, let's:
* create your own password for `root`
* install SSH for user `root`
* define your timezone
* add some syntax highlighting (to beautify your life)

Note: All the following commands are done by `root`. I know it is not a good idea.  
My only excuse is that the container will be destroyed right after creating the files, sorry!

```bash
passwd root   # create your own password
apt update && apt -y upgrade
apt -y install openssh-server vim kmod pciutils python3 python3-pip wget
sed -i "s/#PermitRootLogin .*/PermitRootLogin yes/" /etc/ssh/sshd_config
sed -i "s/#PasswordAuthentication .*/PasswordAuthentication yes/" /etc/ssh/sshd_config
sed -i "s/#Port 22/Port 22/" /etc/ssh/sshd_config
service ssh restart   # et non systemctl car docker
rm -f /etc/localtime
echo "Europe/Paris" >/etc/timezone
cat >> ~/.bashrc <<"EOF"
alias ls='ls --color=auto'
force_color_prompt=yes
if [ -n "$force_color_prompt" ]; then
    if [ -x /usr/bin/tput ] && tput setaf 1 >&/dev/null; then
        # We have color support; assume it's compliant with Ecma-48
        # (ISO/IEC-6429). (Lack of such support is extremely rare, and such
        # a case would tend to support setf rather than setaf.)
        color_prompt=yes
    else
        color_prompt=
    fi
fi
if [ "$color_prompt" = yes ]; then
    PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w \$\[\033[00m\] '
else
    PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
fi
unset color_prompt force_color_prompt

EOF
```
Then you can connect to the IP of the container via Putty.  
Note: Each time you restart the container, you have to restart manually the SSH server in the container console with this command `service ssh start` .

## `gasket-dkms` : Compilation & installation
### Is `gasket-dkms` already installed ?
`dpkg -l | grep gasket-dkms` returns nothing if `gasket-dkms` is not installed.

### Conpilation of `gasket-dkms`
```bash
apt-get install -y sudo git
apt-get install -y build-essential
apt-get install -y dkms devscripts lintian
apt-get install -y dh-dkms
apt-get install -y debhelper dh-virtualenv
mkdir /home/mimo/gasket-dkms && cd /home/mimo/gasket-dkms
git clone https://github.com/feranick/gasket-driver
cd /home/mimo/gasket-dkms/gasket-driver/
debuild -us -uc -tc -b
```
You should recover a `deb` file named `gasket-dkms*.deb`

### Instalation of `gasket-dkms`
```bash
cd /home/mimo/gasket-dkms
dpkg -i gasket-dkms*.deb
dpkg -l | grep gasket-dkms  # verification
```

## Installation of Clang 18 & Bazel 6.5.0
### Clang 18 :
```bash
apt-get -y install software-properties-common
wget -qO- https://apt.llvm.org/llvm.sh | bash -s -- 18
clang-18 --version    # pour vérifier
```
### Bazel 6.5.0
```bash
apt install apt-transport-https curl gnupg -y
curl -fsSL https://bazel.build/bazel-release.pub.gpg | gpg --dearmor >bazel-archive-keyring.gpg
mv bazel-archive-keyring.gpg /usr/share/keyrings
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/bazel-archive-keyring.gpg] https://storage.googleapis.com/bazel-apt stable jdk1.8" | tee /etc/apt/sources.list.d/bazel.list
apt update
apt-get -y install bazel=6.5.0   # Ne plus faire d’upgrade sinon Bazel s’actualise
```
Note: From this installation of Bazel 6.5.0, do not `apt upgrade` any more otherwise Bazel will be updated.

## Clone & compile the Feranick repo
### Clone
```bash
mkdir /home/mimo/noncompile && cd /home/mimo/noncompile
git clone --recurse-submodules https://github.com/feranick/pycoral
```
Then I had to change some files to adapt to my Ubuntu.

### Typo in `debian/changelog`
At the moment (2025-01-05), the is a little typo to correct at the very first line of the file `debian/changelog`.  
Change "oral" to "coral".  
The fist letter is missing.

### Remove ARM dpkg-buildpackage
I had to remove all the ARM building because they they caused an error (my Ubuntu doesn't have the librairies).

In file `Makefile`:
* L 179 & 180: comment `dpkg-buildpackage … armhf` and `arm64`.
* L 215 & 216: same but put the parenthesis back above (and remove the `&& \` at the end of the line)
* L 240:  `DOCKER_CPUS := k8`   (remove `armv7a aarch64`)

To be consistent, I also modified `scripts/build.sh`:
* `readonly DOCKER_CPUS="${DOCKER_CPUS:=k8}"`
* `* PYTHON_VERSIONS="3.12"`

### Compilation
There are several flavors of compilation: `make pybind`, `make deb`, `make wheel`.  
To see the list and its explanations, type `make help`.

The first step is to build native code:
```bash
cd /home/mimo/noncompile/pycoral/
apt-get -y install python3-all
apt-get -y install dh-python
apt-get -y install p7zip-full
apt-get -y install zip
TF_PYTHON_VERSION=3.12 CPU=k8 make all
```

## Export wheels and deb files
### Pycoral wheel
```bash
cd /home/mimo/noncompile/pycoral/
TF_PYTHON_VERSION=3.12 CPU=k8 make wheel
```
`make wheel` creates the folder `/dist` with:
* `pycoral-2.0.3-cp312-cp312-linux_x86_64.whl`

### TensorFlow wheel
```bash
cd /home/mimo/noncompile/pycoral/
TF_PYTHON_VERSION=3.12 CPU=k8 make tflite-wheel
```
`make tflite-wheel` insert in the folder `/dist`:
* `tflite_runtime-2.17.1.post1-cp312-cp312-linux_x86_64.whl`

### Pycoral deb 
```bash
cd /home/mimo/noncompile/pycoral/
TF_PYTHON_VERSION=3.12 CPU=k8 make deb
```
`make deb` insert in the folder `/dist`:
* `pycoral-examples_2.0.3_all.deb`
* `python3-pycoral_2.0.3_amd64.deb`

### TFlite deb 
```bash
cd /home/mimo/noncompile/pycoral/
TF_PYTHON_VERSION=3.12 CPU=k8 make tflite-deb
```
`make tflite-deb` insert in the folder `/dist`:
* `python3-tflite-runtime_2.17.1.post1_amd64.deb`

### EdgeTPU ZIP
```bash
cd /home/mimo/noncompile/pycoral/
TF_PYTHON_VERSION=3.12 CPU=k8 make runtime
```
`make runtime` insert a ZIP file in the folder `/dist`.

## LibEdgeTPU deb
We have navigate inside folder `libedgetpu` which is inside feranick repo.  
The first step is to build the code.

### Compilation
```bash
cd /home/mimo/noncompile/pycoral/libedgetpu  # inside feranick repo
apt-get -y install python3-dev
apt-get -y install libflatbuffers-dev
apt-get -y install libabsl-dev
apt-get -y install libusb-1.0
TF_PYTHON_VERSION=3.12 make
```
### Export deb file
```bash
debuild -us -uc -tc -b
```
The files are placed in the parent folder, so just type `cd ..`  
You will see:
* `libedgetpu-dev_16.0tf2.17.1-1_amd64.deb`
* `libedgetpu1-max_16.0tf2.17.1-1_amd64.deb`
* `libedgetpu1-std_16.0tf2.17.1-1_amd64.deb`

## Test
### Install the librairies
```bash
apt-get install -y sudo git
mkdir /home/mimo/fichiers && cd /home/mimo/fichiers
git clone https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8
cd /home/mimo/fichiers/TPU*

# libedgetpu deb
apt-get -y install libusb-1.0-0   # or libusb-1.0
dpkg -i libedgetpu1-std*

# tflite wheel
mv /usr/lib/python3.12/EXTERNALLY-MANAGED /usr/lib/python3.12/EXTERNALLY-MANAGED.bkp
pip3 install --upgrade tflite_runtime*.whl

# pycoral wheel
pip3 install --upgrade pycoral*.whl
```

### Install the Coral test
```bash
mkdir /home/mimo/pycoral && cd /home/mimo/pycoral
git clone https://github.com/google-coral/pycoral
apt-get -y install curl
cd /home/mimo/pycoral/pycoral
bash examples/install_requirements.sh

### Change Numpy version
pip3 install --upgrade numpy==1.26.4

### test Numpy
python3 -c 'from pycoral.utils.edgetpu import get_runtime_version; print(get_runtime_version())'
```

### Do the test
```bash
python3 examples/classify_image.py \
--model test_data/mobilenet_v2_1.0_224_inat_bird_quant_edgetpu.tflite \
--labels test_data/inat_bird_labels.txt \
--input test_data/parrot.jpg
```
My output is :
```
11.9ms
2.6ms
2.6ms
2.7ms
2.8ms
-------RESULTS--------
Ara macao (Scarlet Macaw): 0.75781
```

