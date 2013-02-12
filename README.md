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

### 4. Export UIApp directory from the Guest system

#### 4.1 Linux and OSX

Choose desired directory where you want to export the remote filesystem. Here we
are using /mnt/dev. Be sure to create the dev directory. Export the guest directory manually:

```bash
$ mount -t nfs4 10.11.22.33:/home/tradeo/code /mnt/dev
```

It's recommended to mount it automatically after reboot by adding it to fstab:

```bash
$ echo "10.11.22.33:/home/tradeo/code/ /mnt/dev nfs4 defaults 0 0" >> /etc/fstab
```

This address is static so you don't have to change it.

#### 4.2 Windows

TODO

### 5. Starting the app

Controlling the uiapp service is performed by a shell script named uiapp located in the script
directory. Do not use the exported directory to stop|start|restart the uiapp service. Always 
control the uiapp service though a ssh connection. For example starting should be done that way:

```bash
$ vagrant ssh
$ cd code/uiapp
$ script/uiapp start
```
