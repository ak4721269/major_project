# major_project
#Phase-1

The detailed documentation of yocto can be found here.

I primarily use Ubuntu 16.04 64-bit servers for builds. Other versions should work.As a prerequisite,you need to have git,tar and p python-2.7 installed on your system.

You will need at least the following packages installed

build-essential
chrpath
diffstat
gawk
libncurses5-dev
texinfo
kage

Once installed ,you need to follow following procedure.

1.Create directory structure to download source

mkdir -p ~/rpi/sources

2.cd into directory

cd ~/rpi/sources

3.Get the required layers

We will need bare minimum above 4 clones for building Linux for Raspberry Pi 3 – poky – meta-openembedded – meta-raspberrypi

    meta-virtualization

git clone -b rocko git://git.yoctoproject.org/poky
git clone -b rocko git://git.openembedded.org/meta-openembedded
git clone -b rocko git://git.yoctoproject.org/meta-raspberrypi
git clone -b rocko git://git.yoctoproject.org/meta-virtualization

Preparing the build configuration

Following command will create a build environment using the setup script that comes with poky, and will create a “rpi-build” directory in “~/rpi/” directory

cd ~/rpi/
source sources/poky/oe-init-build-env rpi-build

Now you will be in “~/rpi/rpi-build” directory.

vim conf/local.conf

The local.conf file can be found here.

vim conf/bblayers.conf

The bblayers.conf file can be found here. Finally Build Image

bitbake rpi-basic-image

This may take few hours to complete depending on the Host Machine Configuration.

Now you follow the path /home/akash/rpi/rpi-build/tmp/deploy/images/raspberrypi3 now locate the image rpi-basic-image-raspberrypi3.rpi-sdimg ,this is the image which can be copied to sd card for booting on raspberrypi3. Preferably,You can use gparted tool for partitioning and resizing images . You can install gparted on Ubuntu by following following steps.

sudo apt-get update
sudo apt-get install gparted
sudo apt-get build-dep gparted
sudo apt-get install git gnome-common
git clone git://git.gnome.org/gparted
cd gparted
./autogen.sh
make 
sudo make install

Now you can insert sd card (should have storage size more than 2GB) in your card reader .Insert it in your personal computer or laptop and run the command

 lsblk

Now look for the sd card port which shows the card reader inserted. In my case port was sdb,so I proceeded with the command

sudo dd if=~/rpi/rpi-build/tmp/deploy/images/raspberrypi3/
rpi-basic-image-raspberrypi3.rpi-sdimg 
of=/dev/sdb bs=4M

Then you need to partition the sdcard using Gparted Tool.Once partitioned,you need to place the card in raspberrypi3. Connect your raspberrypi3 to monitor via HDMI port.Connect your router to raspberrypi3 using ethernet cable.Coonect the keyboard to the USB port on raspberrypi3.Now login as root as username. Your raspberrypi3 will boot without asking for password.Connect yoour raspberrypi3 to internet by using ifup eth0 command. Now,Raspberrypi3 is ready to use. The generated image specifically for raspberrypi3 target can be found here.

#Phase 2

In order to generate an SDK which contains the image features of our pre-generated image,we have to use the command

bitbake <image-name> -c populate_sdk

bitbake rpi-basic-image -c populate_sdk 

The generated SDK can be found here here Now we need to install the sdk in order to generate a cross-toolchain . This can be done by the following command

./tmp/deploy/sdk/poky-glibc-x86_64-rpi-basic-image-cortexa7hf-neon-vfpv4-toolchain-2.4.3.sh
     Poky (Yocto Project Reference Distro) SDK installer version 2.4.3
     ===============================================================
     Enter target directory for SDK (default: /opt/poky/2.1):
     You are about to install the SDK to "/opt/poky/2.4.3". Proceed[Y/n]? Y
     Extracting SDK.......................................................................done
     Setting it up...done
     SDK has been successfully set up and is ready to be used.
     Each time you wish to use the SDK in a new shell session, you need to source the environment setup script e.g.
      
        

This will install the sdk in /opt/poky/2.4.3 directory by default.

Now our main task is to get our Yocto SDK and VPN Infrastructure on Eclipse Che

Eclipse Che can be setup on Docker Support and on Openshift Support.

Presently ,we will use Docker Support for setting up Eclipse Che.

wget -qO- https://get.docker.com/ | sh

The detailed documentation of Docker can seen here. You may need to reboot your system in order to prevent error in the next command.

# Interactive help. This command will fail by default but the CLI will print a prompt on how to proceed
docker run -it eclipse/che start

# Or, full start syntax where <path> is a local directory
docker run -it --rm -v /var/run/docker.sock:/var/run/docker.sock -v <path>:/data eclipse/che start

# Example output

$ docker run -ti -v /var/run/docker.sock:/var/run/docker.sock -v ~/Documents/che-data1:/data eclipse/che start
WARN: Bound 'eclipse/che' to 'eclipse/che:6.5.1'
INFO: (che cli): 6.5.1 - using docker 17.10.0-ce / native
WARN: Newer version '5.21.0' available
INFO: (che config): Generating che configuration...
INFO: (che config): Customizing docker-compose for running in a container
INFO: (che start): Preflight checks
         mem (1.5 GiB):           [OK]
         disk (100 MB):           [OK]
         port 8080 (http):        [AVAILABLE]
         conn (browser => ws):    [OK]
         conn (server => ws):     [OK]

INFO: (che start): Starting containers...
INFO: (che start): Services booting...
INFO: (che start): Server logs at "docker logs -f che"
INFO: (che start): Booted and reachable
INFO: (che start): Ver: 6.5.1
INFO: (che start): Use: http://172.19.20.180:8080
INFO: (che start): API: http://172.19.20.180:8080/swagger

Now you need to go to URL corresponding to INFO: (che start) : Use: ,for example ,in this case ,you need to go to http://172.19.20.180:8080 for getting access to the local host of Eclipse Che Dashboard(which is a cloud IDE).

Now you need to upload your Yocto SDK and VPN Infrastructure to Dockerhub in order to integrate our Yocto SDK and VPN Infrastructure on Eclipse Che. For achieving this purpose,we need to build a Docker file.

mkdir ~/dockerimage.yocto.support.and.vpn.tunnel
vim Dockerfile

The contents of the Dockerfile are following


FROM eclipse/cpp_gcc

ENV YOCTO_SDK=poky-glibc-x86_64-rpi-basic-image-cortexa7hf-neon-vfpv4-toolchain-2.4.3.sh
 RUN wget -O /tmp/$YOCTO_SDK "https://www.dropbox.com/s/zwubmt8e4e4i5ak/poky-glibc-x86_64-rpi-basic-image-cortexa7hf-neon-vfpv4-toolchain-2.4.3.sh?dl=0=$YOCTO_SDK"  
RUN chmod +x /tmp/$YOCTO_SDK 
RUN /tmp/$YOCTO_SDK

ENV OPEN_VPN=openvpn-install.sh
 RUN wget -O /tmp/$OPEN_VPN "https://www.dropbox.com/s/r7zdrvinzkkqtx2/openvpn-install.sh?dl=0=$OPEN_VPN"  
RUN chmod +x /tmp/$OPEN_VPN
RUN /tmp/$OPEN_VPN

As a prerequisite,we need to have an account on docker.Now we need to login to docker and build the Dockerfile.

The openvpn shell script can be found here

docker login
sudo docker build --tag="ak4721269/eclipse-che:6.5.1-2" --tag="ak4721269/eclipse-che:latest-1" /home/akash/dockerimage.yocto.support.and.vpn.tunnel/
sudo docker push ak4721269/eclipse-che

Now,the container is finally pushed to dockerhub to the repository ak4721269/eclipse-che. Now we need to integrate this container with Eclipse Che. For this purpose we need to go to the localhost(URL) set for Eclipse Che.For example,for the case discussed above,we need to go to the
URL - http://172.19.20.180:8080 Once the Dashboard is flashed, you need to go to Stacks tab. Now click on Build Stack From Recipe .Once you click on Build Stack From Recipe,three options will appear,COMPOSE , DOCKERIMAGE,DOCKERFILE ,choose DOCKERFILE and type

FROM <your-docker-repository-name> 

For example,

FROM ak4721269/eclipse-che

Click OK and then set your stack Name,Description,Machine,custom commands and environment . Now save the changes and test your stack on a workspace .Once the test workspace is set,your stack will work properly.Now the stack for Yocto SDK and VPN Infrastructure is set.

    © 2019 GitHub, Inc.
    Terms
    Privacy
    Security
    Status
    Help

    Contact GitHub
    Pricing
    API
    Training
    Blog
    About

