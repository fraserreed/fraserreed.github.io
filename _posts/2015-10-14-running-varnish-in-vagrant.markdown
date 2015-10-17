---
layout: post
title:  "Setting up and running Varnish in Vagrant"
summary: 
date:   2015-10-14 18:54:40
categories: varnish
comments: true
---

If your production environment is fronted by a [Varnish][varnish] caching layer, there's a good chance
your development environment isn't.  This can introduce a number of unanticipated consequences when deploying new code to your
production environment that interacts with Varnish, so it's best to have the cache layer also available in your dev environment.

This post will cover the steps required to install and configure Varnish 4 in a local [Vagrant][vagrant] environment, 
running a vanilla Ubuntu 14.04 install.  If you don't already have a Vagrant box set up, follow the instructions in 
their [getting started guide][vagrant-getting-started].

<!--more-->

# Setting Varnish up manually

The simplest way to set up the Varnish service is on the Vagrant box directly, so I'll cover that first.

{% highlight sh %}
Frasers-MBP:~ $ vagrant ssh
Welcome to Ubuntu 14.04.2 LTS (GNU/Linux 3.13.0-32-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

19 packages can be updated.
0 updates are security updates.

Last login: Tue Oct 6 22:19:39 2015 from 10.0.2.2
vagrant@fraser-vagrant:~ $
{% endhighlight %}

First you need to install the Varnish GPG key.

{% highlight sh %}
vagrant@fraser-vagrant:~ $ sudo apt-get install apt-transport-https
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages will be upgraded:
  apt-transport-https
1 upgraded, 0 newly installed, 0 to remove and 157 not upgraded.
Need to get 25.1 kB of archives.
After this operation, 1,024 B of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu/ trusty-updates/main apt-transport-https amd64 1.0.1ubuntu2.10 [25.1 kB]
Fetched 25.1 kB in 0s (68.6 kB/s)
(Reading database ... 91946 files and directories currently installed.)
Preparing to unpack .../apt-transport-https_1.0.1ubuntu2.10_amd64.deb ...
Unpacking apt-transport-https (1.0.1ubuntu2.10) over (1.0.1ubuntu2.8) ...
Setting up apt-transport-https (1.0.1ubuntu2.10) ...
vagrant@fraser-vagrant:~ $ sudo curl https://repo.varnish-cache.org/GPG-key.txt | sudo apt-key add -
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2442  100  2442    0     0   3938      0 --:--:-- --:--:-- --:--:--  3938
OK
vagrant@fraser-vagrant:~ $
{% endhighlight %}

Next add the deb package from Varnish.

{% highlight sh %}
vagrant@fraser-vagrant:~ $ sudo echo "deb https://repo.varnish-cache.org/ubuntu/ trusty varnish-4.1" | sudo tee -a /etc/apt/sources.list.d/varnish-cache.list

deb https://repo.varnish-cache.org/ubuntu/ trusty varnish-4.1
vagrant@fraser-vagrant:~ $
{% endhighlight %}

Finally install the varnish package.

{% highlight sh %}
vagrant@fraser-vagrant:~ $ sudo apt-get update
....
Reading package lists... Done
vagrant@fraser-vagrant:~ $ sudo apt-get install varnish
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following extra packages will be installed:
  libvarnishapi1
Suggested packages:
  varnish-doc
The following NEW packages will be installed:
  libvarnishapi1 varnish
0 upgraded, 2 newly installed, 0 to remove and 157 not upgraded.
Need to get 580 kB of archives.
After this operation, 1,812 kB of additional disk space will be used.
Do you want to continue? [Y/n] Y
Get:1 https://repo.varnish-cache.org/ubuntu/ trusty/varnish-4.1 libvarnishapi1 amd64 4.1.0-1~trusty [58.8 kB]
Get:2 https://repo.varnish-cache.org/ubuntu/ trusty/varnish-4.1 varnish amd64 4.1.0-1~trusty [521 kB]
Fetched 580 kB in 5s (97.5 kB/s)
Selecting previously unselected package libvarnishapi1.
(Reading database ... 91946 files and directories currently installed.)
Preparing to unpack .../libvarnishapi1_4.1.0-1~trusty_amd64.deb ...
Unpacking libvarnishapi1 (4.1.0-1~trusty) ...
Selecting previously unselected package varnish.
Preparing to unpack .../varnish_4.1.0-1~trusty_amd64.deb ...
Unpacking varnish (4.1.0-1~trusty) ...
Processing triggers for ureadahead (0.100.0-16) ...
ureadahead will be reprofiled on next reboot
Processing triggers for man-db (2.6.7.1-1ubuntu1) ...
Setting up libvarnishapi1 (4.1.0-1~trusty) ...
Setting up varnish (4.1.0-1~trusty) ...
 * Starting HTTP accelerator varnishd                                                                                                           [ OK ]
Processing triggers for libc-bin (2.19-0ubuntu6.6) ...
Processing triggers for ureadahead (0.100.0-16) ...
vagrant@fraser-vagrant:~ $ 
{% endhighlight %}

You will see from the `* Starting HTTP accelerator varnishd` line that the Varnish service has been started.

If you open a web browser and hit your Vagrant box root url on port `6081` you should get a `503 Backend fetch failed` 
error (assuming your default web app responds on port 80 and not port 8080 - Varnish is set up to hit port 8080 by 
default).  If you open `/etc/varnish/default.vcl` you will see the default settings:

{% highlight sh %}
vagrant@fraser-vagrant:~ $ sudo cat /etc/varnish/default.vcl
#
# This is an example VCL file for Varnish.
vcl 4.0;

# Default backend definition. Set this to point to your content server.
backend default {
    .host = "127.0.0.1";
    .port = "8080";
}
{% endhighlight %}

Update this file to use the correct port and restart Varnish.

{% highlight sh %}
vagrant@fraser-vagrant:~ $ sudo service varnish restart
 * Stopping HTTP accelerator varnishd                                                                                                           [ OK ]
 * Starting HTTP accelerator varnishd
vagrant@fraser-vagrant:~ $ 
{% endhighlight %}

Reload your web browser, and you should see the root page as expected.

There are two quick ways to validate that the requests to your web server are going through Varnish.  

The first is by simply inspecting the response headers returned from the request.  They should contain the headers 
`Via:1.1 varnish-v4` and `X-Varnish:<id>` or `X-Varnish: <id> <id-r>`.  For a cache miss, only the ID of the current 
request is returned (`<id>`), and for a cache hit, both the ID of the current request and the ID of the request that 
populated the cache are returned (`<id>` and `<id-r>`).
 
The second is by enabling `varnishlog` inside your Vagrant box, which is quite verbose but will tell you everything
there is to know about the request.

{% highlight sh %}
vagrant@fraser-vagrant:~ $ varnishlog
*   << BeReq    >> 65555
-   Begin          bereq 65554 pass
-   Timestamp      Start: 1444962247.992826 0.000000 0.000000
-   BereqMethod    GET
-   BereqURL       /
-   BereqProtocol  HTTP/1.1
-   BereqHeader    Host: local.fraser-vagrant:6081
-   BereqHeader    Cache-Control: max-age=0
-   BereqHeader    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
-   BereqHeader    Upgrade-Insecure-Requests: 1
-   BereqHeader    User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/46.0.2490.71 Safari/537.36
-   BereqHeader    Accept-Encoding: gzip, deflate, sdch
-   BereqHeader    Accept-Language: en-US,en;q=0.8
-   BereqHeader    X-Forwarded-For: 10.0.0.1
-   BereqHeader    X-Varnish: 65555
-   VCL_call       BACKEND_FETCH
-   VCL_return     fetch
-   BackendOpen    23 boot.default 127.0.0.1 80 127.0.0.1 43227
...
{% endhighlight %}

# Setting Varnish up via Chef

To provide this in a rebuildable Vagrant environment, you can add the steps outlined above to your Vagrantfile.

{% highlight ruby %}
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.hostname = "fraser-vagrant"

  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = "opscode-ubuntu-14.04"

  # The url from where the 'config.vm.box' box will be fetched 
  config.vm.box_url = "https://cloud-images.ubuntu.com/vagrant/trusty/current/trusty-server-cloudimg-i386-vagrant-disk1.box"

  config.vm.network :private_network, ip: "10.0.0.10"

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  config.vm.network :forwarded_port, guest: 80, host: 8080
  

  config.vm.define :varnish do |varnish|
    $script_varnish = <<SCRIPT
sudo apt-get update
sudo apt-get install apt-transport-https -y
sudo apt-get install curl -y
sudo curl https://repo.varnish-cache.org/GPG-key.txt | sudo apt-key add -
sudo echo "deb https://repo.varnish-cache.org/ubuntu/ trusty varnish-4.1" | sudo tee -a /etc/apt/sources.list.d/varnish-cache.list
sudo apt-get update -y
sudo apt-get install varnish -y
SCRIPT

    varnish.vm.provision :shell, :inline => $script_varnish
  end

end
{% endhighlight %}

[varnish]:     https://www.varnish-cache.org/
[vagrant]:     https://www.vagrantup.com/
[vagrant-getting-started]: https://docs.vagrantup.com/v2/getting-started/index.html

