---
layout: post
title: "Patching OpenWRT on Netgear WNDR3700 for BT FTTC"
date: 2013-04-14 18:46
comments: true
sharing: true
categories: [OpenWRT]
---
Updated 6 June 2013 with some corrections and more detail.

With the arrival of BT's Fibre-to-the-Cabinet (FTTC) offering, the telco moved away from PPPoA as the default connection protocol to PPPoE, dropping PPPoA support. When using the provided BT Homehub 3, a MTU of 1500 is supported as per [RFC 4638](https://tools.ietf.org/html/rfc4638)
, rather than the more usual 1492 bytes. Being able to use a MTU of 1500 bytes removes to need to use [MSS clamping](http://lartc.org/howto/lartc.cookbook.mtu-mss.html) to work around problems with Path MTU discovery but it can create other problems. Having a larger MTU improves efficiency as more data can be transmitted in a single packet. So using a larger MTU is a no-brainer - or it would be if not for a couple of problems.

* A MTU of 1500 in the PPP session required the parent interface supports an MTU of 1508 ('baby jumbo' packets).
* RFC 4638 is not widely supported. I've yet to come across an off-the-shelf router than does.

I've been running [OpenWRT](https://openwrt.org/) on a [Netgear WNDR3700](http://www.netgear.co.uk/home/products/wirelessrouters/high-performance/WNDR3700.aspx) for a while with MSS clamping and it's be fine  - certainly the through put is good and it's allowed me to use [Hurricane Electric's IPv6 tunnel](http://ipv6.he.net/), update dynamic DNS with easy and provide a raft of monitoring. But due to the above issues, supporting a 1500 MTU over PPPoE isn't straight forward. However, it is possible by using a couple of patches and compiling OpenWRT from source. As I found out, it's not as scary as it sounds! This technique should work for the WNDR3700v1, v2 and the WNDR3800. I've based the work of the current stable version or OpenWRT 12.09 aka Attitude Adjustment. The [Buildroot - Installation](http://wiki.openwrt.org/doc/howto/buildroot.exigence), [WNDR3700](http://wiki.openwrt.org/toh/netgear/wndr3700) and [Building OpenWrt for Netgear WNDR3700](http://wiki.openwrt.org/doc/howtobuild/build.wndr3700) pages are well worth reading.

Preparing the build environment
-------------------------------

Firstly, you'll need a machine to build OpenWRT. I've used Ubuntu 12.04 (precise) under [Vagrant](http://vagrantup.com), with just a basic Vagrantfile.

Once the VM us up and running, the follow commands will install the neccasery software and checkout the OpenWRT source code. 

``` sh
# Make sure we have an up to date package index and install the build requirements
$ sudo apt-get update
$ sudo apt-get install build-essential subversion git-core libncurses5-dev zlib1g-dev gawk flex quilt libssl-dev xsltproc libxml-parser-perl
# Check out the source code and make sure all pre-requisites are satisfied.
$ svn co svn://svn.openwrt.org/openwrt/branches/attitude_adjustment/
$ cd attitude_adjustment
$ make prereq
```

If it's ready to go the configuration menu will appear. Exit for now - time to install the patch files.

Patching for RFC 4638 support
-----------------------------

For RFC 4638 - 'baby jumbo' packet size - support, two patches are required. 

1. A patch to the ag71xx ethernet driver to allow an MTU of great 1508.
2. A patch to the ppp client to enable RFC 4638 support.

### Ethernet driver

For the ag71xx, a [patch](https://lists.openwrt.org/pipermail/openwrt-devel/2012-June/015782.html) was submitted the openwrt-devel mailing list. It hasn't been accepted into trunk yet, although a [ticket](https://dev.openwrt.org/ticket/11347) does exist for it. As is it on the list, file paths needed editing for the patch to work with OpenWRT 12.09 . An [amended version](https://raw.github.com/mattwillsh/openwrt-wndr3x00-rfc4638/master/655-MIPS-ath79-baby-jumbo-support.patch) can be found in the [github repo](https://github.com/mattwillsh/openwrt-wndr3x00-rfc4638) that goes with this post.  Put this file in ```target/linux/ar71xx/patches-3.3```

### ppp client software

For support in ppp and its pppoe plugin, this [commit](http://git.ozlabs.org/?p=ppp.git;a=commit;h=fd1dcdf758418f040da3ed801ab001b5e46854e7) adds the necessary features. [A version](https://raw.github.com/mattwillsh/openwrt-wndr3x00-rfc4638/master/600-ppp-rfc4638.patch) is available in the github repo that goes with this article. The file needs to go in ```package/ppp/patches```.

If you clone the git repo into the same folder as VM's Vagrantfile, you can run the following:

``` sh
$ cp /vagrant/openwrt-wndr3x00-rfc4638/655-MIPS-ath79-baby-jumbo-support.patch ~/attitude_adjustment/target/linux/ar71xx/patches-3.3/
$ cp /vagrant/openwrt-wndr3x00-rfc4638/600-ppp-rfc4638.patch ~/attitude_adjustment/package/ppp/patches/
```

The firmware can now be build using the normal ```make menuconfig```, ```make``` process. (See the [building](http://wiki.openwrt.org/doc/howtobuild/build.wndr3700) page  as linked above).  I'll cover the options I've used to add dynamic DNS updates, OSPF routing and IPv6 in my next post.

