---
layout: post
title: "Patching OpenWRT on Netgear WNDR3700 for BT FTTC"
date: 2013-04-14 18:46
comments: true
sharing: true
categories: openwrt
---
With the arrival of BT's Fibre-to-the-Cabinet (FTTC) offering, the telco moved away from PPPoA as the default connection protocol to PPPoE, dropping PPPoA support. When using the provided BT Homehub 3, a MTU of 1500 is supported as per [RFC 4638](https://tools.ietf.org/html/rfc4638)
, rather than the more usual 1492 bytes. Being able to use a MTU of 1500 bytes removes to need to use [MSS clamping](http://lartc.org/howto/lartc.cookbook.mtu-mss.html) to work around problems with Path MTU discovery but it can create other problems. Having a larger MTU improves efficiency  as more data can be transmitted in a single packet and there is less change of packet. So using a larger MTU is a no-brainer - or it would be if not for a couple of problems.
<!-- more -->

* A MTU of 1500 in the PPP session required the parent interface supports an MTU of 1508 ('baby jumbo' packets).
* RFC 4638 is not widely supported. I've yet to come across an off-the-shelf router than does.

I've been running [OpenWRT](https://openwrt.org/) on a [Netgear WNDR3700](http://www.netgear.co.uk/home/products/wirelessrouters/high-performance/WNDR3700.aspx) for a while with MSS clamping and it's be fine as far as I can tell - certainly the through put is good and it's allowed me to use [Hurricane Electrics IPv6 tunnel](http://ipv6.he.net/), update dynamic DNS with easy and provide a raft of monitoring. But due to the above issues, supporting a 1500 MTU over PPPoE isn't straight forward. However, it is possible by using a couple of patches and compiling OpenWRT from source. It's not as scary as it sounds! This technique should work for the WNDR3700v1, v2 and the WNDR3800. I've based the work of the current stable version or OpenWRT 12.09 aka Attitude Adjustment. The [Buildroot - Installation](http://wiki.openwrt.org/doc/howto/buildroot.exigence), [WNDR3700](http://wiki.openwrt.org/toh/netgear/wndr3700) and [Building OpenWrt for Netgear WNDR3700](http://wiki.openwrt.org/doc/howtobuild/build.wndr3700) pages are well worth reading.

Firstly, you'll need a machine to build OpenWRT. I've used Debian wheezy. ```apt-get install build-essential subversion``` then, somewhere convenient, ```svn co svn://svn.openwrt.org/openwrt/branches/attitude_adjustment/```. In the attitude_adjustment folder run ```make prereq``` and check what packages are needed and install them.

For baby jumbo support, a [patch](https://lists.openwrt.org/pipermail/openwrt-devel/2012-June/015782.html) was submitted the openwrt-devel mailing list. It hasn't been accepted into trunk yet, although a [ticket](https://dev.openwrt.org/ticket/11347) does exist for it. The file paths needed editing. An [amended version](https://raw.github.com/mattwillsh/openwrt-wndr3x00-rfc4638/master/655-MIPS-ath79-baby-jumbo-support.patch) can be found in the [github repo](https://github.com/mattwillsh/openwrt-wndr3x00-rfc4638) that goes with this post.  Put this file in target/linux/ar71xx/patches-3.3

For support in ppp and its pppoe plugin, this [commit](http://git.ozlabs.org/?p=ppp.git;a=commit;h=fd1dcdf758418f040da3ed801ab001b5e46854e7) adds the necessary support.  The [raw patch](http://git.ozlabs.org/?p=ppp.git;a=patch;h=fd1dcdf758418f040da3ed801ab001b5e46854e7) can be downloaded and given a suitable name and put in packages/ppp/patches.

The firmware can now be build using the normal ```make menuconfig```, ```make``` process. (See the [building](http://wiki.openwrt.org/doc/howtobuild/build.wndr3700) page  as linked above)


