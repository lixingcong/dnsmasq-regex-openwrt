# Compile it as ipk

Simple steps to compile:

	# Clone openwrt repo
	git clone https://github.com/openwrt/openwrt.git
	cd openwrt
	
	# Checkout the current release tag
	git checkout openwrt-xx.xx

	# Install feeds
	./scripts/feeds update -a
	./scripts/feeds install -a
	
	# Delete the orignal dnsmasq package from official feeds
	rm -rf package/network/services/dnsmasq
	
	# Config and build the 'dnsmasq' as module. It was located in 'Base System '
	make menuconfig
	
	# Compile the full openwrt, it may take about 20 minutes
	make -j8
	
	# Compile dnsmasq
	make package/dnsmasq-regex-openwrt/compile V=s
	
	# Copy the binary if you like
	find bin/packages | grep dnsmasq

# Notes

## Toolchain version mismatch

You may get segmentation fault if you built it with different toolchain(for example, install dnsmasq to openwrt-22.03, while package was built under 19.07)

You'd better build full openwrt again if new version was released.

## Get rid of procd-ujail

If you could not run dnsmasq, run ```logread``` and check if error occurred like this:

	daemon.crit dnsmasq[1]: cannot access directory /etc/dnsmasq.d/: No such file or directory

Consider add ```/etc/dnsmasq.d``` to ```/etc/config/dhcp``` to get file access permission. Check for [this issue](https://github.com/openwrt/openwrt/issues/9726#issuecomment-1198828327).


