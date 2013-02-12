## Using preconfigured development VM (recommended)

### 1. Installing Vagrant

This step assumes that you have installed ruby already.

#### 1.1 Linux and OSX

```bash
$ sudo gem install vagrant 
```

#### 1.2 Windows

Grab the latest vagrant version from the official [download page](http://downloads.vagrantup.com/)
and install it using the windows msi installer.

### 2. Installing Virtualbox

#### 2.1 Linux (Ubuntu)

You can install the packaged version:

```bash
$ sudo apt-get install virtualbox-ose
```

or download and install from the official virtualbox download page

#### 2.2 OSX and Windows

Get the latest VirtualBox installation file from VirtualBox [download page](https://www.virtualbox.org/wiki/Downloads)
and install it.

### 3. Starting VDI image using vagrant

* Clone the `infrastructure` repository and go into it:

    ```bash
    $ git clone git@github.com:tradeo/infrastructure.git
    ```

* Spin up the vdi image:

    ```bash
    $ vagrant up
    ```

This will automatically download the vdi box and configure it using a provision script

* In order to be able to use export the UIApp directory from the guest system, a vboxnet network adapter
should be configured. You can check whether it is running with the required ip address by executing the
following command:

    #### Linux and OSX
    
    ```bash
    $ ip a show dev vboxnet0
    ```

    #### Windows

    ```bash
    ipconfig
    ```

If there is an ip address like 10.11.22.1/24 configured then you are ready to go,
otherwise you should add it:

    ```bash
    VBoxManage hostonlyif ipconfig vboxnet0 --ip 10.11.22.1 --netmask 255.255.255.0
    ```

