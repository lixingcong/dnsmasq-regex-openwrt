# Compile it as ipk

Simple steps to compile:

	# Download openwrt sdk
	wget https://downloads.openwrt.org/releases/22.03.5/targets/ipq40xx/generic/openwrt-sdk-22.03.5-ipq40xx-generic_gcc-11.2.0_musl_eabi.Linux-x86_64.tar.xz
	tar xf openwrt-sdk*.tar.xz
	cd openwrt-sdk*

	# Update and install feeds
	./scripts/feeds update -a
	./scripts/feeds install libpcre

	# Disable some options related to kernel modules, They are located in 'Global Build Settings'
	make menuconfig
	Press SPACE to disable 'Select all target specific packages by default'
	Press SPACE to disable 'Select all kernel module packages by default'
	Press SPACE to disable 'Select all userspace packages by default'
	Press ESC twice to exit and save.

	# Clone the source of dnsmasq-regex
	pushd package
	git clone https://github.com/lixingcong/dnsmasq-regex-openwrt -b openwrt-22.03
	popd
	
	# Config modules. It was located in 'Base System'
	make menuconfig
	Press M to compile dnsmasq as a module

	# (Optional) Speed up the compile progress
	Press SPACE to disable useless 'libatomic', 'libgomp', 'librt', 'libstdcpp'
	
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

Consider adding ```option confdir /etc/dnsmasq.d``` to ```/etc/config/dhcp``` to get file access permission.

Or you can add ```procd_add_jail_mount /etc/dnsmasq.d``` to ```/etc/init.d/dnsmasq```, a few lines before ```procd_close_instance```

Check for [this issue](https://github.com/openwrt/openwrt/issues/9726#issuecomment-1198828327) for more details.

## Where does the patch come from?

The patch is from [dnsmasq-regex](https://github.com/lixingcong/dnsmasq-regex/tree/master/patches), and ```900-regex-server-ipset.patch``` is produced by [quilt](https://openwrt.org/docs/guide-developer/toolchain/use-patches-with-buildsystem) command.

<details>
  <summary>How to use quilt to deploy patches</summary>

```
# Install upstream dnsmasq
./scripts/feeds install dnsmasq

# Move the upstream dnsmasq folder to custom one
rm package/feeds/base/dnsmasq
mv feeds/base/package/network/services/dnsmasq package/dnsmasq-regex-openwrt

# Change dnsmasq/Makefile to add some build flags

# Apply patches from dnsmasq-regex repo
make package/dnsmasq-regex-openwrt/{clean,prepare} V=s QUILT=1
cd build_dir/target*/dnsmasq-nodhcpv6/dnsmasq-*

# Apply all patches maintained by Openwrt developers
quilt push -a

# If you want to apply several patches(not all), run 'quilt push xxx' many times
# quilt series
# quilt push 001-xxx.patch
# quilt push 002-xxx.patch

# Create a regex patch
quilt new 900-regex-server-ipset.patch

quilt add Makefile
quilt add src/config.h
quilt add src/dnsmasq.h
quilt add src/domain-match.c
quilt add src/forward.c
quilt add src/network.c
quilt add src/option.c

patch -p1 < /path/to/001-regex-server.patch
patch -p1 < /path/to/002-regex-ipset.patch
quilt refresh
make package/dnsmasq-regex-openwrt/update V=s

# Rebuild the package
make package/dnsmasq-regex-openwrt/{clean,compile} V=s
```

</details>
