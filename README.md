# OpenSprinkler Push

This is a project for adding push notifications to an OpenSprinkler installation. The project assumes the use of Parse to keep iOS/Android abstraction clean and easy.

## Development Environment Setup

Assuming using Homebrew for package management

	$ brew install qemu

Grab the qemu kernel

	$ curl http://xecdesign.com/downloads/linux-qemu/kernel-qemu > kernel-qemu

Grab a [Raspbian image](https://www.raspberrypi.org/downloads/).

Boot up the image replacing 2015-02-16-raspbian-wheezy.img with the image you downloaded.

	$ qemu-system-arm -cpu arm1176 -m 256 -M versatilepb -no-reboot -serial stdio -append "root=/dev/sda2 panic=1 rootfstype=ext4 rw init=/bin/bash" -kernel kernel-qemu -hda 2015-02-16-raspbian-wheezy.img

In the qemu console, add the following file and lines to deal with the disk:

	$ vi /etc/udev/rules.d/90-qemu.rules

	    KERNEL=="sda", SYMLINK+="mmcblk0"
	    KERNEL=="sda?", SYMLINK+="mmcblk0p%n"
	    KERNEL=="sda2", SYMLINK+="root"

Resize the image now on your host:

	$ qemu-img resize 2015-02-16-raspbian-wheezy.img +8G

Almost ready to run, perform any normal options you may need to configure, change password, set boot into desktop, etc)

	$ qemu-system-arm -cpu arm1176 -m 256 -M versatilepb -no-reboot -serial stdio -append "root=/dev/sda2 panic=1 rootfstype=ext4 rw" -kernel kernel-qemu -hda 2015-02-16-raspbian-wheezy.img

Time to setup OpenSprinkler on the guest:

	$ cd /usr/local
	$ git clone https://github.com/OpenSprinkler/OpenSprinklerGen2 OpenSprinkler
	$ cd OpenSprinkler
	$ ./build.sh ospi (may need sudo)
	$ chmod +x /etc/init.d/OpenSprinkler.sh
	$ update-rc.d OpenSprinkler defaults

Go ahead and shutdown the instance, we'll relaunch and forward port 8080 back to our host

	$ qemu-system-arm -cpu arm1176 -m 256 -M versatilepb -no-reboot -serial stdio -append "root=/dev/sda2 panic=1 rootfstype=ext4 rw" -kernel kernel-qemu -hda 2015-02-16-raspbian-wheezy.img -redir tcp:8080::8080

Ready to develop for OpenSprinkler!