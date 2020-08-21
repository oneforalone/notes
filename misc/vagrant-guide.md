

# Vagrant

## Project Setup

Vagrantfile:

1. Mark the root directory of your project. Many of the configuration options in Vagrant are relative to this root directory.
2. Describe the kind of machine and resources you need to run your project, as well as what software to install and how you want to access it.


Vagrant has a built-in command for initializing a directory for usage with Vagrant: `vagrant init`.

```sh
$ mkdir vagrant_getting_started
$ cd vagrant_getting_started
$ vagrant init hashicorp/bionic64
```

This will place a `Vagrantfile` in your current directory(`vagrant_getting_started` here). Then run 
`vagrant init` which your can also run in a pre-existing directory to build your own vms.


# Boxes

Instead of building a virtual machine from scratch, which would be a slow and tedious process, Vagrant 
uses a base image to quickly clone a virtual machine. These base images are known as "boxes" in Vagrant, 
and specifying the box to use your Vagrant environmnet is always the first step after creating a new 
Vagrantfile.

## Installing a Box

If you ran the command `vagrant init`, then you've already installed a box before, and you do not need to 
run the commands below again. However, it is still worth reading this section to learn more about how 
boxes are managed.

Boxes are added to Vagrant with `vagrant box add`. This stores the box under a specific name so that 
multiple Vagrant environments can re-use it. If you have not added a box yet, you can do so now:

```sh
$ vagrant box add hashicorp/bionic64
```

This will download named "hashicorp/bionic64" from [Hashicorp's Vagrant Cloud box catalog](https://vagrantcloud.com/boxes/search), a place where you can find and host boxes.
While it is easiest to download boxes from HashiCorp's Vagrant Cloud you can also add boxes from a local 
file, custom URL, etc.

Boxes are globally stored for the current user. Each project uses a box as an initial image to clone from, 
and never modifies the actual base image. This means that if you have two projects both using the 
hashicorp/bionic64 box we just added, adding files in one guest machine will have no effect on the other 
machine.

In the above command, you will notice that boxes are namespaced. Boxes are broken down into two parts - 
the username and the box name - separated by a slash. In the example above, the username is "hashicorp", 
and the box is "bionic64". You can also specify boxes via URLs or local file paths, but that will not be 
covered in the getting started guide.

> Namespaces do not guarantee canonical boxes! A common misconception is that a namespace like "ubuntu" 
represents the canonical space for Ubuntu boxes. This is untrue. Namespaces on Vagrant Cloud behave very 
similarly to namespaces on GitHub, for example. Just as GitHub's support team is unable to assist with 
issues in someone's repository, HashiCorp's support team is unable to assist with third-party published 
boxes.

## Using a Box

Now that the box has been added to Vagrant, we need to configure our project to use it as a base. Open the 
`Vagrantfile` and change the contents to the following:

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "hashicorp/bionic64"
end
```

The "hashicorp/bionic64" in this case must match the name you used to add the box above. This is how 
Vagrant knows what box to use. If the box was not added before, Vagrant will automatically download and 
add the box when it is run.

You may specify an explicit version of a box by specifying `config.vm.box_version` for example:

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "hashicorp/bionic64"
  config.vm.box_version = "1.1.0"
end
```

You may also specify the URL to a box directly using `config.vm.box_url`:

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "hashicorp/bionic64"
  config.vm.box_url = "https://vagrantcloud.com/hashicorp/bionic64"
end
```

In the next section, we will bring up the Vagrant environment and interact with it a little bit.

## Finding more Boxes

For the remainder of this getting started guide, we will only use the "hashicorp/bionic64" box we added 
previously. But soon after finishing this getting started guide, the first question you will probably have 
is "where do I find more boxes?"

The best place to find more boxes is [HashiCorp's Vagrant Cloud box catalog](https://vagrantcloud.com/boxes/search). 
HashiCorp's Vagrant Cloud has a public directory of freely available boxes that run various platforms and 
technologies. HashiCorp's Vagrant Cloud also has a great search feature to allow you to find the box you 
care about.

In addition to finding free boxes, HashiCorp's Vagrant Cloud lets you host your own boxes, as well as private boxes if you intend on creating boxes for your own organization.


# Up And SSH

It is time to boot your first Vagrant environment. Run the following from your
terminal:

```shell-session
$ vagrant up
```

In less than a minute, this command will finish and you will have a
virtual machine running Ubuntu. You will not actually _see_ anything though,
since Vagrant runs the virtual machine without a UI. To prove that it is
running, you can SSH into the machine:

```shell-session
$ vagrant ssh
```

This command will drop you into a full-fledged SSH session. Go ahead and
interact with the machine and do whatever you want. Although it may be tempting,
be careful about `rm -rf /`, since Vagrant shares a directory at `/vagrant`
with the directory on the host containing your Vagrantfile, and this can
delete all those files. Shared folders will be covered in the next section.

Take a moment to think what just happened: With just one line of configuration
and one command in your terminal, we brought up a fully functional, SSH accessible
virtual machine. Cool. The SSH session can be terminated with `CTRL+D`.

```shell-session
vagrant@bionic64:~$ logout
Connection to 127.0.0.1 closed.
```

When you are done fiddling around with the machine, run `vagrant destroy`
back on your host machine, and Vagrant will terminate the use of any resources
by the virtual machine.

-> The `vagrant destroy` command does not actually remove the downloaded box
file. To _completely_ remove the box file, you can use the `vagrant box remove`
command.


# Synced Folders

While it is cool to have a virtual machine so easily, not many people
want to edit files using just plain terminal-based editors over SSH.
Luckily with Vagrant you do not have to. By using _synced folders_, Vagrant
will automatically sync your files to and from the guest machine.

By default, Vagrant shares your project directory (remember, that is the
one with the Vagrantfile) to the `/vagrant` directory in your guest machine.

Note that when you `vagrant ssh` into your machine, you're in `/home/vagrant`.
`/home/vagrant` is a different directory from the synced `/vagrant` directory.

If your terminal displays an error about incompatible guest additions (or no
guest additions), you may need to update your box or choose a different box such
as `hashicorp/bionic64`. Some users have also had success with the
[vagrant-vbguest](https://github.com/dotless-de/vagrant-vbguest) plugin, but it
is not officially supported by the Vagrant core team.

Run `vagrant up` again and SSH into your machine to see:

```shell-session
$ vagrant up
...
$ vagrant ssh
...
vagrant@bionic64:~$ ls /vagrant
Vagrantfile
```

Believe it or not, that Vagrantfile you see inside the virtual machine
is actually the same Vagrantfile that is on your actual host machine.
Go ahead and touch a file to prove it to yourself:

```shell-session
vagrant@bionic64:~$ touch /vagrant/foo
vagrant@bionic64:~$ exit
$ ls
foo Vagrantfile
```

Whoa! "foo" is now on your host machine. As you can see, Vagrant kept
the folders in sync.

With [synced folders](/docs/synced-folders/), you can continue
to use your own editor on your host machine and have the files sync
into the guest machine.


# Provisioning

Alright, so we have a virtual machine running a basic copy of Ubuntu and
we can edit files from our machine and have them synced into the virtual machine.
Let us now serve those files using a webserver.

We could just SSH in and install a webserver and be on our way, but then
every person who used Vagrant would have to do the same thing. Instead,
Vagrant has built-in support for _automated provisioning_. Using this
feature, Vagrant will automatically install software when you `vagrant up`
so that the guest machine can be repeatably created and ready-to-use.

## Installing Apache

We will just setup [Apache](http://httpd.apache.org/) for our basic project,
and we will do so using a shell script. First, we need to add some html content
which will be served by the Apache webserver. This will act as our DocumentRoot
folder. To do this create a subdirectory named `html` in the project root
directory. In the `html` directory create a html file named `index.html`.
For example:

```html
<!DOCTYPE html>
<html>
  <body>
    <h1>Getting started with Vagrant!</h1>
  </body>
</html>
```

The script below will symlink our shared folder `/vagrant` so that apache serves
the `html` folder when accessing the root page locally. Now, create the
following shell script and save it as `bootstrap.sh` in the same directory as
your Vagrantfile:

```bash
#!/usr/bin/env bash

apt-get update
apt-get install -y apache2
if ! [ -L /var/www ]; then
  rm -rf /var/www
  ln -fs /vagrant /var/www
fi
```

Next, we configure Vagrant to run this shell script when setting up
our machine. We do this by editing the Vagrantfile, which should now
look like this:

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "hashicorp/bionic64"
  config.vm.provision :shell, path: "bootstrap.sh"
end
```

The "provision" line is new, and tells Vagrant to use the `shell` provisioner
to setup the machine, with the `bootstrap.sh` file. The file path is relative
to the location of the project root (where the Vagrantfile is).

## Provision!

After everything is configured, just run `vagrant up` to create your
machine and Vagrant will automatically provision it. You should see
the output from the shell script appear in your terminal. If the guest
machine is already running from a previous step, run `vagrant reload --provision`,
which will quickly restart your virtual machine, skipping the initial import
step. The provision flag on the reload command instructs Vagrant to
run the provisioners, since usually Vagrant will only do this on the first
`vagrant up`.

After Vagrant completes running, the web server will be up and running.
You cannot see the website from your own browser (yet), but you can verify
that the provisioning works by loading a file from SSH within the machine:

```shell-session
$ vagrant ssh
...
vagrant@bionic64:~$ wget -qO- 127.0.0.1
```

This works because in the shell script above we installed Apache and
setup the default `DocumentRoot` of Apache to point to our `/vagrant`
directory, which is the default synced folder setup by Vagrant.

You can play around some more by creating some more files and viewing
them from the terminal, but in the next step we will cover networking
options so that you can use your own browser to access the guest machine.

-> **For complex provisioning scripts**, it may be more efficient to package a
custom Vagrant box with those packages pre-installed instead of building them
each time. This topic is not covered by the getting started guide, but can be
found in the [packaging custom boxes](/docs/boxes/base) documentation.


# Networking

At this point we have a web server up and running with the ability to
modify files from our host and have them automatically synced to the guest.
However, accessing the web pages simply from the terminal from inside
the machine is not very satisfying. In this step, we will use Vagrant's
_networking_ features to give us additional options for accessing the
machine from our host machine.

## Port Forwarding

One option is to use _port forwarding_. Port forwarding allows you to
specify ports on the guest machine to share via a port on the host machine.
This allows you to access a port on your own machine, but actually have
all the network traffic forwarded to a specific port on the guest machine.

Let us setup a forwarded port so we can access Apache in our guest. Doing so
is a simple edit to the Vagrantfile, which now looks like this:

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "hashicorp/bionic64"
  config.vm.provision :shell, path: "bootstrap.sh"
  config.vm.network :forwarded_port, guest: 80, host: 4567
end
```

Run a `vagrant reload` or `vagrant up` (depending on if the machine
is already running) so that these changes can take effect.

Once the machine is running again, load `http://127.0.0.1:4567` in
your browser. You should see a web page that is being served from
the virtual machine that was automatically setup by Vagrant.

## Other Networking

Vagrant also has other forms of networking, allowing you to assign
a static IP address to the guest machine, or to bridge the guest
machine onto an existing network. If you are interested in other options,
read the [networking](/docs/networking/) page.


# Share

Now that we have a web server up and running and accessible from your machine,
we have a fairly functional development environment. But in addition to
providing a development environment, Vagrant also makes it easy to share
and collaborate on these environments. The primary feature to do this in
Vagrant is called [Vagrant Share](/docs/share/).

Vagrant Share lets you share your Vagrant environment to anyone around the
world with an Internet connection. It will give you a URL that will route
directly to your Vagrant environment from any device in the world that is
connected to the Internet.

First, follow the [installation guide](/docs/share/#installation) before getting
started. You need the `vagrant-share` plugin for the rest of the tutorial to work.

Also, [`ngrok`](https://ngrok.com/download) is required.

Next, run `vagrant share`:

```shell-session
$ vagrant share
...
==> default: Creating Vagrant Share session...
==> default: HTTP URL: http://b1fb1f3f.ngrok.io
...
```

Your URL will be different, so do not try the URL above. Instead, copy
the URL that `vagrant share` outputted for you and visit that in a web
browser. It should load the Apache page we setup earlier.

If you modify the files in your shared folder and refresh the URL, you will
see it update! The URL is routing directly into your Vagrant environment,
and works from any device in the world that is connected to the internet.

To end the sharing session, hit `Ctrl+C` in your terminal. You can refresh
the URL again to verify that your environment is no longer being shared.

Vagrant Share is much more powerful than simply HTTP sharing. For more
details, see the [complete Vagrant Share documentation](/docs/share/).

Note: Vagrant Share now defaults to using the `ngrok` driver.
The `classic` driver has been deprecated.

~> **Vagrant share is not designed to serve production traffic!** Please do not
rely on Vagrant share outside of development or Q/A. The Vagrant share service
is not designed to carry production-level traffic.


# Teardown

We now have a fully functional virtual machine we can use for basic
web development. But now let us say it is time to switch gears, maybe work
on another project, maybe go out to lunch, or maybe just time to go home.
How do we clean up our development environment?

With Vagrant, you _suspend_, _halt_, or _destroy_ the guest machine.
Each of these options have pros and cons. Choose the method that works
best for you.

**Suspending** the virtual machine by calling `vagrant suspend` will
save the current running state of the machine and stop it. When you are
ready to begin working again, just run `vagrant up`, and it will be
resumed from where you left off. The main benefit of this method is that it
is super fast, usually taking only 5 to 10 seconds to stop and start your
work. The downside is that the virtual machine still eats up your disk space,
and requires even more disk space to store all the state of the virtual
machine RAM on disk.

**Halting** the virtual machine by calling `vagrant halt` will gracefully
shut down the guest operating system and power down the guest machine.
You can use `vagrant up` when you are ready to boot it again. The benefit of
this method is that it will cleanly shut down your machine, preserving the
contents of disk, and allowing it to be cleanly started again. The downside is
that it'll take some extra time to start from a cold boot, and the guest machine
still consumes disk space.

**Destroying** the virtual machine by calling `vagrant destroy` will remove
all traces of the guest machine from your system. It'll stop the guest machine,
power it down, and remove all of the guest hard disks. Again, when you are ready to
work again, just issue a `vagrant up`. The benefit of this is that _no cruft_
is left on your machine. The disk space and RAM consumed by the guest machine
is reclaimed and your host machine is left clean. The downside is that
`vagrant up` to get working again will take some extra time since it
has to reimport the machine and re-provision it.


# Rebuild

When you are ready to come back to your project, whether it is tomorrow,
a week from now, or a year from now, getting it up and running is easy:

```shell-session
$ vagrant up
```

That's it! Since the Vagrant environment is already all configured via
the Vagrantfile, you or any of your coworkers simply have to run
`vagrant up` at any time and Vagrant will recreate your work environment.


# Providers

In this getting started guide, your project was always backed with
[VirtualBox](https://www.virtualbox.org). But Vagrant can work with
a wide variety of backend providers, such as [VMware](/docs/vmware/),
[Hyper-V](/docs/hyperv), and more. Read the page
of each provider for more information on how to set them up.

Once you have a provider installed, you do not need to make any modifications
to your Vagrantfile, just `vagrant up` with the proper provider and
Vagrant will do the rest:

```shell-session
$ vagrant up --provider=vmware_fusion
```

Once you run `vagrant up` with another provider, every other Vagrant
command does not need to be told what provider to use. Vagrant can automatically
figure it out. So when you are ready to SSH or destroy or anything else,
just run the commands like normal, such as `vagrant destroy`. No extra
flags necessary.

For more information on providers, read the full documentation on
[providers](/docs/providers/).