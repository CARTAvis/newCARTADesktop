newCARTADesktop
=======

The idea is to interface with newCARTA through an [Electron App](https://electronjs.org) where the [CARTA c++](https://github.com/CARTAvis/carta) and [CARTA Meteor client](https://github.com/CARTAvis/newCARTAMeteorApp) are running from a [Docker](https://www.docker.com/) container on the user's machine. 

Test versions to try:

`http://alma.asiaa.sinica.edu.tw/_downloads/newCARTAv3.dmg`

`http://alma.asiaa.sinica.edu.tw/_downloads/CARTA-Desktop-linux-v3.tar.gz`

---

### Note: To run in Local Mode, Docker  needs to be installed on your system

#### Easy way to install docker on Linux (Ubuntu, CentOS7):
1. `curl -fsSL https://get.docker.com/ | sh` (Need to enter root password).
2. `sudo systemctl start docker`
3. `sudo systemctl enable docker` for docker to start automatically on boot.
4. `sudo usermod -a -G docker $USER` so the normal user has permission to run docker.
5. Log out and in, or reboot system, to enable Step 4.
6. `docker pull ajmasiaa/newcarta_meteor_v3` to download the docker image.
7. `xhost +` to allow x11 to work in the docker container (`sudo setenforce 0` may also be necessary)

##### Note: the script in Step 1 doesn't work for CentOS6. Instead do the following to install Docker:
1. `rpm -iUvh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm` to install EPEL repositories.
2. `yum -y install docker-io`
3. `service docker start && chkconfig docker on`
4. `groupadd docker`
5. `usermod -aG docker $USER`
6. Log out and in, or reboot.

#### Easy way to install docker on Mac:
1. Install Docker from the dmg: https://download.docker.com/mac/stable/Docker.dmg 
2. `docker pull ajmasiaa/newcarta_meteor_v3`

---

### To directly run the docker image

The docker image can be run directly for testing. The docker image itself is based on Ubuntu. The included `start.sh` script simply executes the CARTA c++ and then starts the Meteor client. Once it is running, the user can open localhost:3000 in their webrowser.

#### Linux:
Tested on CentOS7, Ubuntu 17.04, Fedora 24
1. `xhost +`
2. `docker run -p 3000:3000 -p 9999:9999 -e DISPLAY=$DISPLAY -ti -v /tmp/.X11-unix:/tmp/.X11-unix  ajmasiaa/newcarta_meteor_v3 /start.sh`
3. Open any web browser and go to the URL `localhost:3000`

#### Mac:
1. `xhost +`
2. `docker run -p 3000:3000 -p 9999:9999 -e DISPLAY=docker.for.mac.host.internal:0 -ti -v /tmp/.X11-unix:/tmp/.X11-unix  ajmasiaa/newcarta_meteor_v3 /start.sh`
3. Open any web browser and go to the URL `localhost:3000`

---

### To build and run the Electron App
A requirement for CARTA is to be a standalone desktop application. This can be achieved with newCARTA using Electron. Electron is effectively a very basic Chrome browser window which displays newCARTA running locally (localhost:3000) on the user's machine.

1. Install Meteor on your system `curl https://install.meteor.com/ | sh`
1. `git clone https://github.com/CARTAvis/newCARTADesktop.git`
2. `cd newCARTADesktop`
3. `meteor npm install`
4. `meteor npm start`

Note: On CentOS7 and Fedora (and probably RedHat), libXScrnSaver must be installed; `sudo yum install libXScrnSaver`

---

### To package the Mac and Linux Electron Apps

For Mac, an installable DMG is created (non-signed for now). For Linux, a directory is created that can be zipped later. The Linux version can be created on a Mac, plus the Linux version is universal between RedHat and Ubuntu based systems.

1. Install the electron-packager tool: `meteor npm install electron-packager -g`
2. Create Mac App: `electron-packager . --overwrite --platform=darwin --arch=x64 --icon=assets/icons/mac/icon.icns --prune=true --out=release-builds`
3. Install the electron DMG installer tool: `npm install electron-installer-dmg -g`
4. Create Mac DMG: `electron-installer-dmg --icon=assets/icons/mac/icon.icns release-builds/CARTA-Desktop-darwin-x64/CARTA-Desktop.app newCARTA-Desktop`

5. Create Linux App: `electron-packager . --overwrite --asar=true --platform=linux --arch=x64 --icon=carta_logo_v2.png --prune=true --out=release-builds`

The packaged apps can be found in the `newCARTADesktop/release-builds/` directory.

---

### Note: This is work in progress. Here is an incomplete list of things to do:

1. Docker and x11 can be problematic, particularly on Linux. Sometimes there can be errors such as `QXcbConnection: Could not connect to display`. I think setting `-e DISPLAY=$DISPLAY` and running `xhost +` at first helps in most cases, but I'm still investigating a guaranteed way for it to work everytime. On Mac, I think `-e DISPLAY=docker.for.mac.host.internal:0` will work every time, but still need to do more testing. On Linux, it now queries the system for $DISPLAY and uses whatever that value is in the docker command.
2. If possible, get the app to install Docker and download the image if not already present.
3. Currently there is a fixed pause (`child_process.execSync("sleep 5")`) after clicking Local Mode in order to give the docker image time to start up before the browser window opens and requests localhost:3000. Although startup is now quite fast, this could probably be improved by having it monitoring the output log and waiting until `websocket onopen done` comes up before continuing.
4. Reduce size of the docker image. It is currently quite large at 1.806GB (`ajmasiaa/newcarta_meteor_v3`) but I am working on reducing the size by a few hundred megabytes.
5. Currently some sample images are supplied in the docker image. Instead, it should mount a user's local directory so they can open their own images. This is easy to do in the docker run command with an extra -v flag; `-v <absolute path to local directory>:<absolute path to directory in docker image>`

Your comments and suggestions are welcome.

---

#### Links for Reference

https://electronjs.org/

https://www.meteor.com/

https://www.docker.com/

https://github.com/CARTAvis

https://github.com/CARTAvis/newCARTAMeteorApp

https://www.christianengvall.se/electron-packager-tutorial/

