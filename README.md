# Tradeo UI Application

Tradeo's UI App is a Ruby on Rails 3.2 application that serves as the
[web user interface](https://tradeo.com) of the Tradeo social trading
platform. The instructions that follow will likely be of interest to
the members of its development team.

There are two general ways of developping over the app. The recommanded,
supported and encouraged™ way of doing it is through the VDI
(Virtual Development Image), which is replica of a production image with
minor development modifications to make the dev lifecycle easier.

The second method is dealing with the software dependency steps manually,
which is documented [here](https://github.com/tradeo/uiapp/wiki/Running%20UIApp%20locally).
Devs are highly discouraged from using the manual method, as the dependency
tree of the app and its components is not trivial and it is easier to break
deployment due to the developer's machine being [different from production](http://devopsreactions.tumblr.com/post/43216677327/coming-to-work-after-night-deploy-went-bad).

## Using the preconfigured development VM

The virtual machine management happens through [Vagrant](http://www.vagrantup.com/),
which automates all tasks to do with fetching and running the VDI. 
The first step is to install the Vagrant gem and install VirtualBox.

### 1. Installing Vagrant

This step assumes that you have installed ruby already.

#### 1.1 Linux and OSX

```bash
$ gem install vagrant --no-ri --no-rdoc
```

#### 1.2 Windows

Grab the latest vagrant version from the official [download page](http://downloads.vagrantup.com/)
and install it using the Windows MSI installer.

### 2. Installing Virtualbox

#### 2.1 Linux (Ubuntu)

You can install the packaged version:

```bash
$ sudo apt-get install virtualbox-ose
```

or download and install from the official VirtualBox download page.

#### 2.2 OSX and Windows

Get the latest VirtualBox installation file from VirtualBox [download page](https://www.virtualbox.org/wiki/Downloads)
and install it.

### 3. Starting VDI image using vagrant

* Clone the `devops` repository and cd into it:

    ```bash
    $ mkdir -p ~/work/tdo/ ; cd $_
    $ git clone git@github.com:tradeo/devops.git --depth 1
    ```

Before running the following, you have to make sure you have a fork
of the `uiapp` in your Github account, or provisioning will not work.
Additionally, the machine on which you are about to run Vagrant needs
to have its SSH key imported in your Github account. So, on the guest,
you should be able to run `ssh git@github.com` without issue.

There is a known limitation with SSH agent forwarding, if your key's
identity is not loaded prior to running Vagrant. If your SSH key has 
no passphrase (and dont let @alexbhr catch you with an unprotected 
SSH key!) or is locked, you will need to run `ssh-add` in the same 
shell prior to executing `vagrant`.

* Spin up the VDI image:

    ```bash
    $ vagrant up
    ```

This will automatically download the Vagrant box and configure it using 
a provision script architecture very similar to production. The provisioning
script will take about 10 minutes to complete. Additionally, the default 
VirtualBox configuration with which the box ships is 2 CPUs and 
2 Gigabytes of memory. For a snappier experience, editing the `Vagrantfile` 
prior to running `vagrant up` and specifying a higher number of vCPUs
is recommended should the specs of your physical machine permit.

Vagrant then takes care of a [few things](https://github.com/tradeo/uiapp/wiki/Running%20UIApp%20locally), generally in the following order:
- Downloads the Vagrant box from S3
- Sets up local host to VM networking with the VM always being at 10.11.22.33
- Runs through our Puppet manifests
- Creates a Postgre database with random credentials
- Installs [Oh My Zsh](https://github.com/robbyrussell/oh-my-zsh) with our own theme
- Exports the development dir `/home/tradeo/code` via NFS

Using `vagrant ssh` in the `devops` directory will always drop a 
shell with the current SSH key identity shared via a socket. That
means SSH-ing places (like Github) will work without a password.

When finished, connect to the VDI image with the mentioned method above
and execute the following steps in order to have a fully operational UIApp:

- Clone your fork for the `uiapp`. Add tradeo upstream in the forked repository.
  ```bash
  $ git clone git@github.com:yourusername/uiapp.git
  $ git remote add upstream git@github.com:tradeo/uiapp.git
  ```

- Install all gems and node.js dependencies
  ```bash
  $ bundle install
  $ npm install
  ```

- Setup up dev database
  ```bash
  $ rake db:setup
  ```

- Seed dev database with fakes
  ```bash
  $ rake db:fake:load
  ```

- Prepare test database for tests
  ```bash
  $ rake test:prepare
  ```

- Finally, deploy the app to Troquebox
  ```bash
  $ torquebox deploy
  ```

There are some sane vim and emacs defaults on the image, but for 
more serious development, the `/home/tradeo/code` path is exported 
via [NFS](http://en.wikipedia.org/wiki/Network_File_System), which allows
for quick, near-native [performance](https://gist.github.com/hugobarauna/2351792) of access.

### 4. Export UIApp directory from the Guest system

#### 4.1 Linux

Choose desired directory where you want to export the remote filesystem. Here we
are using `~/work/tdo`. Mount the guest directory manually:

```bash
$ mkdir -p ~/work/tdo/vdi-code
$ sudo mount -t nfs4 10.11.22.33:/home/tradeo/code /your/home/work/tdo/vdi-code
```

You can make mounting easier by adding the following to your fstab:

```bash
$ echo "10.11.22.33:/home/tradeo/code/ /your/home/work/tdo/vdi-code nfs4 defaults 0 0" >> /etc/fstab
```

The IP address of the guest is static so you don't have to adjust it. 

#### 4.2 Mac OS

A good tool for managing NFS mounts in OSX is [NFSManager](http://www.bresink.com/osx/NFSManager.html). 
It is a great alternative to dealing with the wicket NFS automount logic in OSX and requires no 
configuration apart from specifying the IP of the server and mount point.

#### 4.3 Windows

Mounting NFS shares under Windows assumes that you have installed Client Services for
NFS. This package is available via the Add/Remove Software wizard in the Control Panel.
Once installed you can mount the NFS share:

```bash
$ mount \10.11.22.33:/home/tradeo/code Z:
```


### 5. Starting the app

Controlling the uiapp service is performed by a shell script named `uiapp` located in the `script/`
directory. Do not use the exported directory to `stop|start|restart` the uiapp service. Always 
control the uiapp service though a ssh connection. Use the NFS mount only as a way to access the code
base with your editor of choice. The image should have all the necessery perks to make hacking inside of it a bliss.

For example starting should be done that way:

```bash
$ vagrant ssh     # the shell prompt is now sexy, cant miss it
$ cd code/uiapp
$ script/uiapp start
```

## Running

Since a lot of services need to be running for the app to be usable
we're relying on a script to run them for us.

```bash
$ script/uiapp start
```

Now open your favorite browser, type
[http://10.11.22.33:8080](http://10.11.22.33:8080), sit back and enjoy.

You can stop the uiapp with:

```bash
$ script/uiapp stop
```

You can restart the app with:

```bash
$ script/uiapp restart
```

You can check is the app is running with:

```bash
$ script/uiapp status
```

The log from all related processes is available in `uiapp/logs/uiapp.log`.

## Code Style

### Ruby Style

Follow the guidelines outlined
[here](http://github.com/bbatsov/ruby-style-guide).

### Rails Style

Follow the guidelines outlined
[here](http://github.com/bbatsov/rails-style-guide).

## Workflow

The basics:

* Always rebase after pulling (`git pull --rebase`)
* Anything in the master branch is **deployable**
* Fork `tradeo/uiapp` and hack there
* To work on something new, create a descriptively named branch off of
  master (ie: new-oauth2-scopes)
* Commit to that branch locally and regularly push your work to the
  same named branch on the server
* When you need feedback or help, or you think the branch is ready for
  merging, open a pull request
* After at least one other dev has reviewed and signed off on the
  feature, it can be merged into master. More complex changes should
  have 2 or more reviewers.
* Once it is merged and pushed to `master`, you can and should deploy
  immediately

For more details check out
[this article](http://scottchacon.com/2011/08/31/github-flow.html).

Commits should ideally have a reference to their related
[YouTrack](https://tradeo.myjetbrains.com) issues.

```
# for something in progress
Implement this and that #UI-XXX in progress
# for something reported by the QAs that's fixed
Fix this and that #UI-XXX fixed
# for something that's internal to the devs like refactoring
Refactor this and that #UI-XXX closed
```

Commit messages should follow the guidelines outlined
[here](http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html).

### Technical details

* Fork the canonical uiapp GitHub repo.
* Add a second remote called `upstream` there:

```bash
$ git remote add upstream git@github.com:tradeo/uiapp.git
```

* Create feature branches like this:

```bash
$ git checkout -b branch-name
```

* Never open pull requests from your `master` branch.

* Update your `master` frequently from `upstream`.

```bash
$ git pull upstream master
```

* Squash the commits in your pull requests into a single commit after
the PR is approved to be merged.

```bash
$ git rebase -i master
```

* Force-push the squashed commit to update the PR.

```bash
$ git push --force
```

* Mention the nicknames of the people you'd like to review a PR when you open it.

## Architecture

TODO

## Troubleshooting

### Remote Debugging

The use of TorqueBox makes conventional debugging techniques hard to
apply. The best way to circumvent the problem is to use remote
debugging provided by `pry-remote` and `pry-debugger`.

Just add `binding.remote_pry` where you want to pause:

```ruby
class UsersController < ApplicationController
  def index
    binding.remote_pry
    ...
  end
end
```

Afterwards load a page that triggers the code. Connect to the session:

```
$ bundle exec pry-remote
```

This will drop you in a pry console at the breakpoint. You can use
commands like `next`, `step`, `exit`, etc to navigate the source code.

### Recurring issues and their resolutions

* The DB structure in not up to date

  ```bash
  rake db:migrate
  rake test:prepare
  ```

* The fakes are not up to date

  ```bash
  rake db:fake:load
  ```

* The global `~/config.yml` file has changed

  Diff for the latest changes in `config/config.yml.example`, merge them
  and do `uiapp restart`.

* The backend services are not up to date

  ```bash
  rm -rf node_modules
  npm install
  ```
* I messed something up horribly and I want to start from scratch
  
  If you deleted something important, the image doesnt boot or some other bad scenario occured,
  you can nuke the VDI and start from scratch:
  ```bash
  cd devops
  vagrant destroy -f
  git pull
  vagrant up
  ```
  **Warning:** Make sure all local changes inside the image are saved and pushed or data _will_ be lost.

## I18n

We support multiple languages through the I18n gem.
All keys are stored in YAML files under `config/locales`.
The site is translated using 3rd party service from [OneSky](http://oneskyapp.com).
New keys are synced to their database using the `rake one_sky:upload` task
and new translatations are downloaded using `rake locales:update`.

There are several directories that contains different keys:

* `defaults/` - holds translations for all Rails internal stuff.
  Check https://github.com/svenfuchs/rails-i18n
* `models/` - holds translations found in model classes
* `onesky/` - these are the translation files obtained from OneSky
* `views/` - all translations used in the views
* `common/` - holds all shared keys that don't belong somewhere else

### Modifying keys

All modification are done in the en.yml files.

If you want to rename or modify existing keys, first ensure you *really* need
that change. Discuss the tradeoff with colleagues. When you change an existing
key you break all translations for that key and OneSky data needs to be updated.
This may require more work than you imagine.

When you do your modification follow these steps:

1. Write the changes in the commit message. This way Geri can see them and you
   won't forget you have additional work. If you have added or renamed keys, list
   them and include the old ones too. If you have changed placeholders describe that
   change, right now OneSky doesn't update them automatically. So someone will have
   to go and rename them in the OneSky app.
2. Run `rake one_sky:upload` when your PR is accepted into master. This will
   upload the new keys into OneSky's database.
3. Inform Geri about the new translations.

### Adding new locale

1. Add the new yml file.
2. Add new locale file for moment.js. Go to https://github.com/timrwood/moment/tree/master/lang and download the appropriate file.
   If there is no language file available, copy the English translation as momentjs doesn't support language fallback and will give error.
2. Add the locale into the db/seeds.rb file for local development and debugging.
3. Add the locale to `config.i18n.available_locales` in config/application.rb to correctly show the locale on the home page.
   This is needed because the "routing filter" gem uses available_locales to determine if the locale is allowed.
4. Deploy on production
5. Add the locale and the code from the admin interface on production.

### Right to left languages

Our pages are served in default language direction - left to right. In order to fix Arabic text
that contains embedded Latin letters, you need to prepend the U+202b character in front.
This is done automatically when we fetch the Arabic translation from OneSky.
If you need to insert it somewhere else, for example if you add text from the admin panel,
plese find how you can type that character for your operating system/editor.

Here is a quick reference that might help you: http://www.fileformat.info/info/unicode/char/202b/index.htm
