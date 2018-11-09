# 4. Building your multi-arch images

This section goes through installing the manifest tool and building the multi-arch docker images.

## Making multi-arch docker images

### 1.	Build image for all architectures and push to docker registry.
First is the list of basic steps, then a pattern you can use with both the mac and command  prompt commands.

#### Basic Steps
  a. Build and push image for s390x (Go onto an s390x linux instance, ssh in)

    i. Get your code onto the instance (i.e. download git files onto the machine or git clone https://github.com/siler23/MultiArchMultiArchDockerICP.git)

    ii. Get to the directory with the code (cd MultiArchDockerICP)

    iii. Build the image using docker (docker build -t myrepo/outyet-s390x .)

    PROXY: add build args for PROXY using previously set variables or if not reset variables and then run (docker build -t myrepo/outyet-x86 --build-arg http_proxy=%http_proxy% --build-arg https_proxy=%https_proxy% --build-arg no_proxy=%no_proxy% .)

    iv. Docker login to your docker registry (docker login registry_address) [only need to do once for each command prompt/terminal session on a machine]

    v. Docker push image (docker push myrepo/outyet-s390x)

  b. Build and push image for x86 (Go onto an x86 computer with docker, probably your workstation)

    i. Get your code onto the instance if not already there (i.e. git clone https://github.com/siler23/MultiArchDockerICP.git)

    ii. Get to the directory with the code and Dockerfile (cd MultiArchDockerICP)

    iii. Build the image using docker (docker build -t myrepo/outyet-x86 .)

    PROXY: add build args for PROXY using previously set variables or if not reset variables and then run (docker build -t myrepo/outyet-x86 --build-arg http_proxy=%http_proxy% --build-arg https_proxy=%https_proxy% --build-arg no_proxy=%no_proxy% .)

    iv. Docker login to your docker registry (docker login registry_address)

    vi. Docker push image (docker push myrepo/outyet-x86)

#### Quick pattern for this:
###### Mac/Linux
cd `Desired Directory for given image`

`IMAGE=outyet`

`ARCH=s390x`

`VERSION=1.0`

`REPO=gmoney23`

***Regular Build***

`docker build -t ${REPO}/${IMAGE}-${ARCH}:${VERSION} .`

***Build Using Proxy***

`docker build -t ${REPO}/${IMAGE}-${ARCH}:${VERSION} --build-arg http_proxy=%proxy% --build-arg https_proxy=%proxy% --build-arg no_proxy=%noproxy% .`

***Other Steps same if using proxy or not***

`docker tag ${REPO}/${IMAGE}-${ARCH}:${VERSION} ${REPO}/${IMAGE}-${ARCH}:latest`

`docker push ${REPO}/${IMAGE}-${ARCH}:${VERSION}`

`docker push ${REPO}/${IMAGE}-${ARCH}:latest`

###### Windows
cd `Desired Directory for given image`

`set IMAGE=outyet`

`set ARCH=s390x`

`set VERSION=1.0`

`set REPO=gmoney23`

***Regular***

`docker build -t %REPO%/%IMAGE%-%ARCH%:%VERSION .`

***Using Proxy***

`docker build -t %REPO%/%IMAGE%-%ARCH%:%VERSION --build-arg http_proxy=%proxy% --build-arg https_proxy=%proxy% --build-arg no_proxy=%noproxy% .`

***Other Steps same if using proxy or not***

`docker tag %REPO%/%IMAGE%-%ARCH%:%VERSION% %REPO%/%IMAGE%-%ARCH%:latest`

`docker push %REPO%/%IMAGE%-%ARCH%:%VERSION%`

`docker push %REPO%/%IMAGE%-%ARCH%:latest`

Then cd to `nextdirectory` only change `IMAGE=mynextimage` or `set IMAGE=mynextimage` and then re-run the same docker commands as before. When your done with s390x go to x86 machine then `ARCH=x86` or `set ARCH=x86` and go through the same process for your x86 images. You can view the images you have at any time on a given machine with the `docker images` command. If we were doing this for applications we were continuosly developing we could write script to do all of these builds for us in linux or in windows using the above template and just run the script when its time to update the image.

## 2.	Make manifest list
***You only need to do this from one computer which for ease of access should probably be your workstation computer rather than a server***

Make a manifest list (a multi-architecture image reference) out of that individual images your pushed. Basically, you are mapping both images to the same tag so that when someone asks for the image via `docker pull gmoney23/outyet` it will have a pointer to the correct image for your architecture automatically (i.e. if you are on s390x it will point to layers from gmoney23/outyet-s390x or if you are on x86 it will point to layers from gmoney23/outyet-x86) while using the same label (i.e. gmoney23/outyet). This makes it so users of the image don’t have to worry about using different tags for different archs so the Dockerfiles can be the same for different applications across architectures (i.e. the same Dockerfile can be used for a node application on z and x86 or a go application on z and x86).

## Docker Manifest: An experimental feature

### Recommended Path: Unless need added features, use this built-in command and skip the optional section. If need added features, do the OPTIONAL section marked OPTIONAL below.
***You only need to do this from one computer which for ease of access should probably be your workstation computer rather than a server***
### Enabling manifest lists in Docker
docker manifest
docker manifest is only supported on a Docker cli with experimental cli features enabled

####	Need to enable experimental feature
IMPORTANT: PLEASE USE DOCKER 18.06
If you don't have this version, please upgrade

![Upgrade Docker](../images/DockerUpdate.png)

Check with `docker version`
![Docker Version Output](../images/Docker_Version_Experimental_Check.PNG)
Under Client: If Experimental: false like in the picture then you need to do these steps. If Experimental: true then you can just use docker manifest command.
###### Mac/Linux
`ls ~/.docker`
If `~/.docker/config.json` exists:
`vim ~/.docker/config.json`
Add `"experimental": "enabled"` to config.json file

If `~/.docker/config.json` doesn’t exist:

`cd ~`

`mkdir .docker`

(Open config.json with favorite text editor)
`vim ~/.docker/config.json`

Add `"experimental": "enabled"` to config.json file
![End Result](../images/Enabling-experimental.PNG)

###### Windows
In the Windows file explorer go to your users directory and see if .docker exists if so open that directory and edit it with your favorite text editor. Add `"experimental": "enabled"` to config.json file

If it doesn't exist, make a new folder called .docker and create a new file called config.json. Add "experimental: enabled" inside of brackets.

![End Result](../images/Enabling-experimental.PNG)

###### Check for Success
In the end check your docker version to see that the change persisted. Specifically, look for client's experimental section being marked `Experimental: True`.
![Experimental Final](../images/Final_Exp.png)

First `docker login` to your registry

IF USING PROXY: make sure your http_proxy, https_proxy, and no_proxy our set ***if pushing to a repo outside of your internal network***.
#### Push versioned first

**Replace gmoney23 with your registry in the commands below**

`docker manifest create gmoney23/outyet:1.0 gmoney23/outyet-s390x:1.0 gmoney23/outyet-x86:1.0`

`docker manifest push -p gmoney23/outyet:1.0` (It takes a minute to push, be patient)

If you want to inspect your manifest, use `docker manifest inspect gmoney23/outyet:1.0`
#### Then push latest for current versioned

**Replace gmoney23 with your registry in the commands below**

`docker manifest create gmoney23/outyet gmoney23/outyet-s390x gmoney23/outyet-x86`

`docker manifest push -p gmoney23/outyet` (It takes a minute to push, be patient)
**-p is important so you can push new manifests later for latest tag (no version tag defaults to latest as in this example)**

If you want to inspect your manifest, use `docker manifest inspect gmoney23/outyet`

**But what if I already pushed a latest before and want to update it with my new latest version?**

For example, you just pushed version 2.0 of your app, you can update the existing manifest by using:

 `docker manifest create --amend gmoney23/outyet gmoney23/outyet-s390x gmoney23/outyet-x86`
 
  to replace the new latest manifest with the old one after you have already updated the latest individual images to the version 2.0 images.


***You can also make a pattern for all of this if you want to automate the process or just type it out for each image...***

Time to go to the last stage, unless you need that pesky manifest-tool.

### [Part 5: Now, it's time to get these images into Kubernetes](5-Deploy-to-Kubernetes.md)

#### OPTIONAL [Most Users Should Skip]: Install Manifest tool if need added features
The docker manifest command is experimental becuase while it does create manifests and push it doesn't have other features yet such as pushing manifest from files. If you want these extra features you can install the manifest tool here. If not, I would suggest just using the docker manifest command as its generally integrated better with docker. For example, once you docker login you don't need to enter username/password like you do with manifest-tool.

***You only need to do this from one computer which for ease of access should probably be your workstation computer rather than a server***

If golang not yet installed, install here with package for your os/arch [here](https://golang.org/dl/)

If git not yet installed, see instructions here for how to install for your os [Git install instructions](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)

***If using proxy***, set git proxy settings. An example would be if you want to use proxy for all git push/pull set `git config --global http.proxy http://proxyUsername:proxyPassword@proxy.server.com:port
` replacing with your proxy values. For more different git proxy configurations for your specific needs so [this gist on using git with proxies](https://gist.github.com/evantoli/f8c23a37eb3558ab8765)

Install manifest-tool here with package for your os/arch [here](https://github.com/estesp/manifest-tool/releases)

#### After all s390x and amd64 images are pushed (Only do the do following if you haven't done with the docker manifest command)
***You only need to do this from one computer which for ease of access should probably be your workstation computer rather than a server***
##### Push versioned manifest first
First, you will update the yaml files in the given directory of the image you are trying to push (in this case outyet) as well as change into the directory. Then, switch my username/password in the following command for yours for the image repository you are pushing to and:

IF USING PROXY: make sure your http_proxy, https_proxy, and no_proxy our set if pushing to a repo outside of your internal network.

`manifest-tool --username gmoney23 --password *** push from-spec smallest-outyet/vmanifest.yaml`

##### Then push latest manifest for current versioned
`manifest-tool --username gmoney23 --password *** push from-spec smallest-outyet/manifest.yaml`


[Part 5: Now, it's time to get these images into Kubernetes](5-Deploy-to-Kubernetes.md)