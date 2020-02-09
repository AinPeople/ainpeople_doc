## What is AinCI-Tizen?
AinCI-Tizen creates Build, Test, and CI servers with Docker to automatically generate system images for porting to Tizen-based IoT devices. This provides a development server infrastructure for Tizen-based IoT devices.

## How to build a Tizen image without using AinCI-Tizen(기존 방법)

### GBS를 이용한 Tizen 빌드

![GBS Build](https://github.com/ainpeople/ainpeople_doc/blob/master/ainci-tizen/images/GBS_build.jpg)

#### Prerequisites

* Ubuntu 18.04 LTS

#### Setting up Development Environment

##### Setting Up Gerrit Access

###### Registering a User Account

* Tizen Gerrit : https://www.tizen.org/user/register

###### Configuring SSH for Gerrit Access

* Generate RSA keys

```shell
ssh-keygen [-t rsa] [-C "<Comments>"]
```

* Create an SSH configuration file, ~/.ssh/config, and add

<Gerrit_Username>를 Tizen 계정 명으로 변경
예) User <Gerrit_Username> -> User tizen_guest77

```shell
Host tizen review.tizen.org
Hostname review.tizen.org
IdentityFile ~/.ssh/id_rsa
User <Gerrit_Username>
Port 29418
# Add the line below when using proxy, otherwise, skip it.
# ProxyCommand nc -X5 -x <Proxy Address>:<Port> %h %p

Note:
Both "tizen" and "review.tizen.org" are aliases of the hostname. "tizen" is configured for simplicity of commands when initializing git repositories and cloning specific Tizen projects, and "review.tizen.org" is configured to work with the manifest.xml and _remote.xml files when synchronizing the entire Tizen source.
The ~/.ssh/config file must not be written in by other users. Make sure to remove the write permission by executing chmod o-w ~/.ssh/config. For more information on ssh_config, see man ssh_config.
```



* Copy the full text in ~/.ssh/id_rsa.pub

* Log in to Tizen Gerrit and upload the key

ssh 등록 url : [Tizen ssh key 등록 저장소]

In the Gerrit Web page, click the user name on the top right corner (with an inverted triangle on the right), and select Settings to display the Settings Web page.
Click SSH Public Keys in the left panel, paste the text copied earlier into the Add SSH Public Key box, and click Add.

Log in to Tizen Gerrit(https://review.tizen.org/gerrit) and upload the key



* Verify the SSH connection

```shell
ssh tizen

**** Welcome to Gerrit Code Review ****
```

##### Configuring Git for Gerrit Access

```shell
git config --global user.name <First_Name Last_Name>
git config --global user.email "<E-mail_Address>"
```

##### Setting Up the GBS Configuration(.gbs.conf)

###### Setting Up the Default GBS Configuration File

```shell
general]
profile = profile.unified_standard

Note: The default GBS build parameters, based on the above block, are as follows:
Tizen version: 5.0
Profile: unified
Repository: standard
```



###### Setting Up the Default GBS Configuration File

```shell
[general] 
profile = profile."$Version""$Profile"_"$Repository«

If the Tizen version is 3.0, $Version equals "3.0-".
If the Tizen version is 4.0, $Version equals "4.0-".
If the Tizen version is 5.0, $Version equals "".

Other examples:

Tizen 5.0 Unified / emulator repository
[general]

profile = profile.unified_emulator
Tizen 4.0 Unified / emulator repository

[general]
profile = profile.4.0-unified_emulator
Tizen 3.0 Common / arm64-wayland repository

[general]
profile = profile.3.0-common_arm64-wayland

Each profile entry in the .gbs.conf file specifies multiple repo entries, and each repo entry specifies a URL where RPM files used in the GBS build are located.

Note: The latest directory in the remote repository URLs is a symbolic link in the remote server, which is always linked to the latest new directory and can be changed any time, so make sure to use the latest repo with a specific date to guarantee usability. An example is shown below:

url = http://download.tizen.org/releases/daily/tizen/unified/latest/repos/standard/packages/

This URL is symbolically linked to the latest snapshot number in "http://download.tizen.org/releases/daily/tizen/unified/". To guarantee usability, use a specific date

url = http://download.tizen.org/releases/daily/tizen/unified/tizen-unified_20170627.1/repos/standard/packages/

```

##### Setting Up the Repo Tool

###### Repo is a repository management tool built on top of git.  Multiple git repositories can be downloaded with a single repo command.

```shell
mkdir ~/bin/
PATH=~/bin:$PATH

curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > ~/bin/repo

sudo chmod a+x ~/bin/repo
```

##### 빌드 도구(GBS, MIC) 설치

###### Installing Development Tools

```shell
sudo vim /etc/apt/sources.list
deb [trusted=yes] http://download.tizen.org/tools/latest-release/Ubuntu_18.04/ /
sudo apt-get update
sudo apt-get install gbs mic
```

#### 수동 빌드 절차

##### 1) Cloning Tizen Source Files

###### Building All Packages Locally with GBS

* Cloning All Tizen Projects
    * Cloning All Projects over SSH
    ```shell
    mkdir ~/<Tizen_Project> && cd ~/<Tizen_Project>
    repo init -u ssh://<Username>@review.tizen.org:29418/scm/manifest -b <tizen branch> -m ${profile}_${repository}.xml

    For example: Tizen 5.0 Unified / standard
    repo init -u ssh://<Username>@review.tizen.org:29418/scm/manifest -b tizen -m unified_standard.xml

    Clone the specific snapshot source of all projects over SSH (optional).
    Replace the latest manifest with a snapshot manifest and make proper modifications by executing the following two commands, as appropriate:

    wget  <Snapshot_Manifest_URL> -O .repo/manifests/<profile>/<repository>/projects.xml
    sed -i '3,4d' .repo/manifests/<profile>/<repository>/projects.xml

    For example: Tizen 5.0 Unified / standard
    wget http://download.tizen.org/releases/weekly/tizen/unified/tizen-unified_20170928.1/builddata/manifest/tizen-unified_20170928.1_standard.xml -O .repo/manifests/unified/standard/projects.xml
    sed -i '3,4d' .repo/manifests/unified/standard/projects.xml

    Synchronize the files for all the projects based on the information downloaded by the repo init command by executing the following command:
    repo sync

    ```
    * Cloning All Projects over HTTPS
    ```shell
    mkdir ~/<Tizen_Project> && cd ~/<Tizen_Project>
    repo init -u https://git.tizen.org/cgit/scm/manifest -b <tizen branch> -m unified_${repository}.xml

    For example: Tizen 5.0 Unified / standard
    repo init -u https://git.tizen.org/cgit/scm/manifest -b tizen -m unified_standard.xml

    sed -i 's/ssh:\/\/review.tizen.org/https:\/\/git.tizen.org\/cgit/' .repo/manifests/_remote.xml

    The command changes the 'fetch url' in the .repo/manifests/_remote.xml file as follows:

    from fetch="ssh://review.tizen.org/" 
    to fetch="https://git.tizen.org/cgit/"

    Clone the specific snapshot source of all projects over HTTPS (optional).
    Replace the latest manifest with a snapshot manifest and make proper modifications by executing the following two commands, as appropriate:

    wget  <Snapshot_Manifest_URL> -O .repo/manifests/<profile>/<repository>/projects.xml
    sed -i '3,4d' .repo/manifests/<profile>/<repository>/projects.xml

    Synchronize the files for all the projects based on the information downloaded by the repo init command by executing the following command:
    repo sync

    ```

##### 2) Building Packages Locally with GBS

```shell
Build all packages by executing the following commands:
$ cd <Tizen_Project>
$ gbs build <gbs build option>

For example:
$ cd <Tizen_Project>
$ gbs build -A i586 --threads=4 --clean-once
Note: Since the GBS configuration file (.gbs.conf) and build configuration file (build.conf) are also included in scm/manifests, they are automatically downloaded in the following paths after the repo sync:

.gbs.conf: ~/<Tizen_Project>/.gbs.conf
build.conf: ~/<Tizen_Project>/scm/meta/build-config/<Tizen version>/<profile>/ <repository>_build.conf
```

##### 3) Creating Tizen Images with MIC

###### Preparing the Kickstart File

```shell
$ wget <Snapshot_date_URL>/builddata/images/<Repository>/image-configurations/<kickstart_file>

For example: Tizen: 4.0: Unified / standard/ mobile-wayland-armv7l-tm1.ks
$ wget http://download.tizen.org/releases/daily/tizen/unified/tizen-unified_20170627.1/builddata/images/standard/image-configurations/mobile-wayland-armv7l-tm1.ks

Modify the original kickstart file to include locally built RPMs into the Tizen image.

For example: Tizen: 4.0: Unified / standard/ mobile-wayland-armv7l-tm1.ks

The repo section of the original kickstart file:

repo --name=unified-standard --baseurl=http://download.tizen.org/snapshots/tizen/unified/@BUILD_ID@/repos/standard/packages/ --ssl_verify=no
repo --name=base_arm --baseurl=http://download.tizen.org/snapshots/tizen/base/latest/repos/arm/packages/ --ssl_verify=no
The repo section of the modified kickstart file:

repo --name=unified-standard --baseurl=http://download.tizen.org/snapshots/tizen/unified/@BUILD_ID@/repos/standard/packages/ --ssl_verify=no --priority=99
repo --name=base_arm --baseurl=http://download.tizen.org/snapshots/tizen/base/latest/repos/arm/packages/ --ssl_verify=no --priority=99
repo --name=local --baseurl=file:///home/<User>/GBS-ROOT/local/repos/tizen3.0-tm1/armv7l/ --priority=1
```

###### Creating a Tizen Image

```shell
To create a Tizen image, execute the following command:
$ gbs createimage --ks-file=mobile-wayland-armv7l-tm1.ks

If you have more than 4 GB of RAM available, use the --tmpfs option to speed up the image creation:
$ gbs createimage --ks-file=mobile-wayland-armv7l-tm1.ks --tmpfs
```

### Jenkins(CI 도구)를 이용한 GBS 빌드 자동화

![Jenkins Build](https://github.com/ainpeople/ainpeople_doc/blob/master/ainci-tizen/images/Jenkins_build.jpg)

#### Jenkins Prerequisites

* GBS Jenkins jobs run on Jenkins framework, which requires one Jenkins master and several slave builders. The number of slave builders depends on the scale of the development team, as well as the frequency of job triggering.
* Specific to our situation, detailed requirements are as follows(Note: The master and slave nodes are logical concept, thus can be set up in one machine.
):
    * One node for one Jenkins master
    * Multiple nodes for Jenkins builders
    * One individual machine for respective download servers
* The download server needn't to be connected to Jenkins master, but rsync or SSH must be properly configured to make sure all slave builders can upload build artifacts to download servers.

#### Jenkins 설치 및 설정

##### Installing and Configuring Jenkins Master and Slave Builders

##### Deployment Instructions

###### Deploying GBS Jenkins Jobs

```shell
Take openSUSE 12.3 as an example, to deploy GBS Jenkins jobs, execute the following commands:

$ sudo zypper addrepo http://download.tizen.org/tools/latest-release/openSUSE_12.3/ tools
$ sudo zypper refresh
$ sudo zypper install gbs-jenkins-jobs    # for jenkins master
$ sudo zypper install gbs-jenkins-scripts  # for jenkins slave nodes

Jenkins master must be restarted to reload all GBS Jenkins jobs after installation. Upon successful reloading, the following two jobs will be ready for use:

GBS local full build
GBS local build with package list
```


<!-- Markdown link & img dfn's -->
[Tizen ssh key 등록 저장소]: https://review.tizen.org/gerrit
[AinCI-Tizen Build]: https://github.com/ainpeople/ainpeople_doc/blob/master/ainci-tizen/images/AinCI-Tizen_build.jpg
