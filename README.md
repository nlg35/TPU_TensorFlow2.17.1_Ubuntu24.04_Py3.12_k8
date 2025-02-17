# TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8

**Date: 2025-01-05**  

Builds for Coral Edge TPU (M.2 PCIe) on a dockerized Ubuntu 24.04.  
Ubuntu is a container of a QNAP NAS which has a Celeron CPU (x86_64 or k8).

I am just going to write the steps I followed thanks to the repo of @feranick.  
Nothing would be possible without his work.

source:  [https://github.com/feranick/pycoral](https://github.com/feranick/pycoral)  
Big thanks to @feranick the author of the repo. He saved lots of Coral users.

The files in this repo are just the results of @feranick pycoral compilation for:
* CPU x86_64 or k8
* Ubuntu 24.04
* Python 3.12
* TensorFlow 2.17.1
* PyCoral 2.0.3
* NumPy 1.26.4

This repo was inspired by [https://github.com/virnik0/pycoral_builds](https://github.com/virnik0/pycoral_builds)  
Thanks to @feranick & @virnik0 and their instructions, I managed to reproduce the steps.  
The sole purpose of this repo is to possibly help.

This installation proposes to keep Python 3.12 and to install the wheels of Pycoral and TfliteRuntime and downgrade Numpy. Thus the old Coral models (initially reserved for Python 3.9) work.

## TOC
Here is the table of contents:

- [1. Hardware](https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8/blob/main/README.md#hardware)
- [2. Docker Compose](https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8/blob/main/README.md#docker-compose)
- [3. SSH to the container](https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8/blob/main/README.md#ssh-to-the-container)
- [4. `gasket-dkms` : Compilation & installation](https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8/blob/main/README.md#gasket-dkms--compilation--installation)
  - [4.1. CPU and kernel](https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8/blob/main/README.md#cpu-and-kernel)
  - [4.2. TPU](https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8/blob/main/README.md#tpu)
  - [4.3. Is `gasket-dkms` already installed?](https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8/blob/main/README.md#is-gasket-dkms-already-installed)
  - [4.4. Compilation of `gasket-dkms`](https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8/blob/main/README.md#compilation-of-gasket-dkms)
  - [4.5  Installation of `gasket-dkms`](https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8/blob/main/README.md#installation-of-gasket-dkms)
- [5. Installation of Clang 18 & Bazel 6.5.0](https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8/blob/main/README.md#installation-of-clang-18--bazel-650)
  - [5.1. Clang 18](https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8/blob/main/README.md#clang-18)
  - [5.2. Bazel 6.5.0](https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8/blob/main/README.md#bazel-650)
- [6. Clone & compile the Feranick repo](https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8/blob/main/README.md#clone--compile-the-feranick-repo)
  - [6.1. Clone](https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8/blob/main/README.md#clone)
  - [6.2. Typo in `debian/changelog`](https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8/blob/main/README.md#typo-in-debianchangelog)
  - [6.3. Remove ARM dpkg-buildpackage](https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8/blob/main/README.md#remove-arm-dpkg-buildpackage)
  - [6.4. Compilation](https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8/blob/main/README.md#compilation)
- [7. Export wheels and deb files](https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8/blob/main/README.md#export-wheels-and-deb-files)
  - [7.1. Pycoral wheel](https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8/blob/main/README.md#pycoral-wheel)
  - [7.2. TensorFlow wheel](https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8/blob/main/README.md#tensorflow-wheel)
  - [7.3. Pycoral deb](https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8/blob/main/README.md#pycoral-deb)
  - [7.4. TFlite deb](https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8/blob/main/README.md#tflite-deb)
  - [7.5. EdgeTPU ZIP](https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8/blob/main/README.md#edgetpu-zip)
- [8. LibEdgeTPU deb](https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8/blob/main/README.md#libedgetpu-deb)
  - [8.1. Compilation](https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8/blob/main/README.md#compilation-1)
  - [8.2. Export deb file](https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8/blob/main/README.md#export-deb-file)
- [9. Test](https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8/blob/main/README.md#test)
  - [9.1. Install the libraries](https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8/blob/main/README.md#install-the-libraries)
  - [9.2. Install the Coral test](https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8/blob/main/README.md#install-the-coral-test)
  - [9.3. Change Numpy version](https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8/blob/main/README.md#change-numpy-version)
  - [9.4. Test Numpy](https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8/blob/main/README.md#test-numpy)
  - [9.5. Do the test](https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8/blob/main/README.md#do-the-test)
- [10. Point of interest of each file](https://github.com/nlg35/TPU_TensorFlow2.17.1_Ubuntu24.04_Py3.12_k8/blob/main/README.md#point-of-interest-of-each-file)

## Hardware
At the very beginning: let's introduce the hardware.

On a QNAP NAS, I have the [Coral dual Edge TPU](https://coral.ai/products/m2-accelerator-dual-edgetpu) mounted on an [PCIe - Dual Edge TPU Adapter](https://github.com/magic-blue-smoke/Dual-Edge-TPU-Adapter). And this PCIe is plugged in the NAS and recognized by the OS.

## Docker Compose
In Container Station, I create a Ubuntu 24.04 container with a fictitious MAC address (and my router will attribute a reserved IP address in my LAN).

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
passwd root   # choose your own password for root
apt update && apt -y upgrade
rm -f /etc/localtime
echo "Europe/Paris" >/etc/timezone
apt-get -y install ssh
apt-get -y install openssh-server vim kmod pciutils python3 python3-pip wget
cat > /etc/ssh/sshd_config.d/10-password-login-for-root.conf <<"EOF"
Port 22
PermitRootLogin yes
PasswordAuthentication yes
EOF
service ssh restart   # no systemctl because of docker
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
### CPU ans kernel
Let's verify the 64-bit architecture (x86 processor) and kernel version:
```bash
uname -i
```
Output: `x86_64`

---
```bash
uname -r
```
Output: `5.10.60-qnap`

### TPU
Let's verify that the PCIe driver is loaded:
```bash
ls /dev/apex
```
Output: `/dev/apex_0  /dev/apex_1`

---
```bash
lsmod | grep apex
```
Output:  
```
apex                   20480  0
gasket                 98304  1 apex
```

### Is `gasket-dkms` already installed ?
`dpkg -l | grep gasket-dkms` returns nothing if `gasket-dkms` is not installed.

### Conpilation of `gasket-dkms`
`gasket-dkms` is a package that contains the driver for Edge TPU accelerators. Google has not updated its repository for recent Linux kernels (> 6.5), so the package is broken. The code has been fixed long ago in the [Github repository](https://github.com/google/gasket-driver) but Google has not compiled and published the binaries. So, we have to compile it ourselves.

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

### Installation of `gasket-dkms`
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
apt-get -y install apt-transport-https curl gnupg
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
At the moment (2025-01-05), there is a little typo to correct at the very first line of the file `debian/changelog`.  
Change "oral" to "coral".  
The first letter is missing.

### Remove ARM dpkg-buildpackage
I had to remove all the ARM buildings because they caused an error:  
`ModuleNotFoundError: No module named '_sysconfigdata__aarch64-linux-gnu'` or  
`ModuleNotFoundError: No module named '_sysconfigdata__arm-linux-gnueabihf'`
(my Ubuntu doesn't have these arm librairies).

So, I modified the file `Makefile`:
* L 179 & 180: comment `dpkg-buildpackage … armhf` and `arm64`.
* L 215 & 216: same but put the parenthesis back above (and remove the `&& \` at the end of the line)
* L 240:  `DOCKER_CPUS := k8`   (remove `armv7a aarch64`)

To be consistent, I also modified `scripts/build.sh`:
* `readonly DOCKER_CPUS="${DOCKER_CPUS:=k8}"`
* `* PYTHON_VERSIONS="3.12"`

### Compilation
There are several flavors of compilation: `make pybind`, `make deb`, `make wheel`.  
To see the list and explanations, type `make help`.

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

### Conclusion
All the librairies are here: these `*.deb` files and the contents of the directory `/dist`.  
We can save the files somewhere (like I did in this repo) and then destroy the Ubuntu container and the map volume content.

## Test
### Install the librairies
```bash
apt-get -y install sudo git
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
## Point of interest of each file
**Relationship between TensorFlow and TFLite:**
* TensorFlow is a powerful open-source machine learning framework developed by Google, used for creating, training, and deploying machine learning models. It Runs primarily on CPUs, GPUs, and TPUs (Tensor Processing Units) in development environments.
* You typically develop and train models using TensorFlow on a powerful machine. Once satisfied with the model's performance, you convert it to TFLite for deployment on Edge TPUs or other embedded devices.
* TF lite is a lightweight library for deploying pre-trained TensorFlow models on mobile and embedded devices. TensorFlow Lite models are smaller and optimized.


**Coral Edge TPU Development Files:**
These files are essential for running TensorFlow Lite models on Coral Edge TPUs.
* `gasket-dkms.deb`: It's a package containing the device driver for Edge TPUs, required for PCIe devices only, to enable the Linux kernel to recognize and interact with the Edge TPU hardware.
* `libedgeTPU.deb` (or `edgedpu_runtime.zip`): It is the core Edge TPU library that provides low-level access to the Edge TPU hardware. It enables communication between  software and Edge TPU.
* `pycoral.whl` (or `python3-pycoral.deb`): It is a Python library that simplifies working with Edge TPUs. Builds on top of libedgeTPU and offers a higher-level API for TensorFlow Lite model execution on Edge TPUs. With PyCoral, you can use standard TensorFlow Lite models (.tflite files) directly on the Edge TPU, there is no need to convert it into _edgetpu.tflite.
* `tflite_runtime.whl` (or `python3-tflite-runtime.deb`): It's the TFLite runtime library, essential for running TFLite models on various platforms, including Edge TPUs. You can use the TFLite runtime directly without PyCoral library.
* `python3-pycoral-examples.deb` (Optional): A package containing example code demonstrating how to use Pycoral for Edge TPU development.
