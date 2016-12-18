title: Elixir in Vagrant
date: 2015-11-10 22:56:13
tags:
  - elixir
  - elixir-lang
  - vagrant
---

I don't like polluting my environment with new languages. In this post I will describe all the steps I had to take to setup my Elixir environment using Vagrant.

For this post I will assume you already have Vagrant installed in you dev environment.

Let's start creating a new directoy which will have:

* Vagrant file
* `src/` directory synced with Vagrant
* Bootstrap file which will install Erlang and Elixir.

```
> mkdir jumpstart
> cd jumpstart
```

Then we create an empty Vagrant file:

```
> vagrant init
```

Just before modifying our Vagrant file let's create our `src/` and our `init/` folders.
```
> mkdir src
> mkdir init
```

Now, let's create our bootstrap file. This script file will run the first time we provision our box and will make sure that we have the required packages to install both Erlang and Elixir.

init/bootstrap.sh
```
#!/bin/bash

set -e

sudo locale-gen en_US.UTF-8
sudo update-locale LANG=en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8

#Getting Erlang pack
sudo wget https://packages.erlang-solutions.com/erlang-solutions_1.0_all.deb
sudo dpkg -i erlang-solutions_1.0_all.deb

# Updating and Upgrading dependencies
sudo apt-get update -y -qq > /dev/null
sudo apt-get upgrade -y -qq > /dev/null

# Install necessary libraries for guest additions and Vagrant NFS Share
sudo apt-get -y -q install linux-headers-$(uname -r) build-essential dkms nfs-common wget git xvfb vim gawk libreadline6-dev zlib1g-dev libssl-dev libyaml-dev libsqlite3-dev sqlite3 autoconf libgdbm-dev libncurses5-dev automake libtool bison pkg-config libffi-dev libxml2-dev libxslt-dev libxml2 erlang-dev erlang-common-test erlang-tools erlang-common-test esl-erlang elixir 
```

Finally we modify our Vagrant file, I removed all the extra comments:

```ruby
Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/trusty64"
  config.vm.hostname = "elixir-vm"
  config.vm.synced_folder "src/", "/home/vagrant/src"
  config.vm.network :private_network, ip: "10.0.15.10"
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
  end
  config.vm.provision :shell, path: "init/bootstrap.sh"
end
```

We go back to our console and type:
```
> vagrant up
```

After Vagrant is done provisioning our box we do:
```
> vagrant ssh
```

If everything is fine, you will be ssh'd to you new Vagrant box. Once here try doing:

```
> vagrant@elixir-vm:~$ iex
```

You will enter Elixir's interactive console:

```
iex(1)> "hello " <> "world"
```

You have a fully working elixir box, where you can have your full source code inside `src` and then compile that inside the box.

In you local box do:

```
> vim src/hello.exs
```

Inside hello.exs:

```
IO.puts "Hello world from Elixir"
```

Now in your elixir-vm you can do:

```
vagrant@elixir-vm:~$ cd src/
vagrant@elixir-vm:~/src$ elixir hello.iex
Hello world from Elixir
```

## Exiting
Once you're done, don't forget to stop your vagrant machine:

```
vagrant@elixir-vm:~/src$ exit
logout
Connection to 127.0.0.1 closed.
> vagrant halt
```
