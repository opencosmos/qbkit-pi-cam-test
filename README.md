# qbkit Raspberry Pi camera demo

_Estimated total duration: 20-60 minutes._

This demonstrates controlling power for and communicating with a Raspberry Pi via UART.
It powers on the Pi, waits for it to boot up, commands it to take a photograph using the Pi Camera, then downloads the photograph to the qbkit.
All data is transferred between qbkit and payload via 230400-baud UART.

Software and data is transferred between the qbkit and the operator (yourself) via Github in this case.
Note that by default, anything pushed to Github is publically visible to the world.

The qbkit is not restricted to Github however, if you use some other Git provider (or host your own private Git servers) then qbkit will just as readily be able to use those if you grant appropriate access with the qbkit's SSH public key.

In future tarball and file-level access will also be available, however this is intended for quick tweaks only and not for bulk transfer since it will be much slower than using Git.

## Pre set-up

In qbapp, generate/get the qbkit's SSH public key.
This will be used by external Git providers (e.g. Github) to identify and to authenticate the qbkit.

### Github account

_Estimated duration: 2 minutes._

#### Using your personal account for the qbkit

Create a [Github](https://github.com) account (or use your existing one).

Add the qbkit's SSH public key (from qbapp) to the keys associated with your account.

#### Using an organisation, with a distinct qbkit account

Another option is to create a new account just for the qbkit.

 1. Add the qbkit's SSH public key as the only key for that account.

 2. Add the qbkit account to your organisation.

 3. Grant the qbkit account read access to repositories which contain programs/libraries/data that you want to install on the qbkit.

 3. Grant the qbkit account read+write access to repositories which the qbkit should push data out to.

### Program repository

_Estimated duration: 2 minutes._

Open [the repository containing the qbkit pi-camera-demo software](https://github.com/opencosmos/qbkit-pi-cam-test) and fork it to your own profile.

### Images repository

_Estimated duration: 2 minutes._

Create a new repository called `qbkit-pi-cam-images`.

## Set-up (qbkit, via qbapp)

### A. Creating volumes

_Estimated duration: 3 minutes._

Create two **Git** volumes on the qbkit.
Use the SSH URLs (`git@...`) not the HTTPS URLs (`https://...`).

1. Program volume (`bin` type) called `pi-cam`.
   Remote is the SSH URL of your fork of the `qbkit-pi-cam-test` repository.

2. Data volume (`var` type) called `images`.
   Remote is the SSH URL of your new `qbkit-pi-cam-images` repository.

### B. Installing software

_Estimated duration: 1 minute._

Pull the program repository to the qbkit.

### C. Creating service

_Estimated duration: 1 minute._

Create a service:

 * name: `capture-image`.

 * program volume: `pi-cam`.

 * description: *(optional)* `Capture an image via the PI camera and download it (pi-cam)`.

 * command-line: `/bin/sh -c './program client /dev/ttyS4 image-$$(date -uIsec).jpg /qbkit/var/images'`

## Set-up (Raspberry PI, via SSH terminal)

These instructions assume that you use a recent version of Debian ARM / Raspbian / Arch Linux ARM.
If your distribution does not support systemd, then use initscripts or upstart to configure your Pi to start the server automatically.

It is assumed that you will run these commands as root.
Prefix with `sudo` as appropriate if not running as root.

### Pre-requisites

_Estimated duration: 1-15 minutes._

In order to compile the server program, will need GNU Make v4.0+ and GCC v6+ to be installed on your Pi.
On Ubuntu, this is acheived with `apt update && apt install build-essential`.
On Arch, this is acheived with `pacman -Sy --needed base base-devel`

The server uses the `raspistill` program, found in `libraspberrypi-bin` package on Raspbian.

### Disable serial terminal, free up the UART

_Estimated duration: 1 minute._

Disable the serial terminal on the Pi.

Removing the `console=ttyAMA0,...` kernel command-line argument from /boot/config.txt should  be sufficient, however it is recommended to use the `raspi-config` tool to disable the serial terminal, and to configure the UART for general purpose use.

### Installation to Pi

_Estimated duration: 5 minutes._

*Advanced users may wish to use cross-compilation, which I won't describe in this document.*

The source code for the demo software (both qbkit and payload sides) can be found in [the Open Cosmos Github repository qbkit-pi-cam-source](https://github.com/opencosmos/qbkit-pi-cam-source.git).

Then run the following commands on the Pi:

	# Clone the source repository onto the Pi
	git clone https://github.com/opencosmos/qbkit-pi-cam-source.git /root/pi-cam-demo

	# Compile program by running
	make -BC /root/pi-cam-demo

	# Copy systemd service file to the correct place
	cp /root/pi-cam-demo/systemd/server.service /etc/systemd/system/pi-cam.service

	# If you are using a USB camera (v4l compatible) instead of the Pi camera
	# then uncomment the "Environment=USE_V4L" line in the unit file.
	#
	# sed -e 's/^# //' /root/pi-cam-demo/systemd/server.service > /etc/systemd/system/pi-cam.service

	# Enable the service so it starts automatically on boot-up
	systemctl enable pi-cam.service

	# Start the service (no reboot required)
	systemctl start pi-cam.service

	# Wait a couple of seconds then verify that the service is running
	systemctl status pi-cam.service

## Connecting

_Estimated duration: 5-10 minutes + 2 more to double-check every connection._

 1. Disconnect all cables from the Raspberry Pi.
    If it is connected to any power source other than the qbkit while also connected to the qbkit power output, it will probably break permanently.

 2. Connect the camera to the Pi.

 3. Connect the following pins on the Pi GPIO to the qbkit.

| Pi #pin | purpose on Pi | connects to #pin on qbkit |
|---------|---------------|---------------------------|
|     2   | 5V0           | Power #4 [5V0]            |
|     4   | -             | -                         |
|     6   | Ground        | Power #3 [GND]            |
|     8   | UART TX       | Data #10 [UART RX]        |
|     10  | UART RX       | Data #9 [UART TX]         |

 4. Ensure that both the qbkit and the Pi have their SD cards properly inserted.

 5. Double-check your wiring.
    The only connections to the Pi should be from the qbkit and from the Pi camera.
    Ensure TX/RX are the correct way (RX of one device to TX of the other).
    Ensure that the power connections are correct.
    Get this wrong and you'll have a baked Pi.

 6. Connect the power supply to the qbkit.

 7. Ensure your computer has an internet connection, and is configured to share the connection with the qbkit.
    This can be fiddly (especially on Windows), however we are working to create dedicated drivers for qbkit on Windows and OSX in order to avoid this.
    Linux users can achieve this with a couple of iptables rules (masquerading + forwarding), enabling ip forwarding via sysctl/procfs, then starting a dhcp server on the qbkit network interface, when it appears on your system.

 8. Connect the qbkit USB to the computer.
    The qbkit will power on and blue LEDs should flash.
    A running-leds ("Knight Rider") pattern should appear once the qbkit has booted up.

## Running (via qbapp)

_Estimated duration: 3 minutes._

Start the service `capture-image`.

This will start the `program` from the program volume `pi-cam`, which then does the following:

 * Powers on the Pi by turning on the 5V0 power rail via the `gpio` command.

 * Waits until the Pi starts responding to pings over UART.

 * Waits 10s.

 * Captures an image, with the name `image-{iso8601-date/time}.jpg`.

 * Transfers the image to the qbkit (over UART), saving into data volume `/qbkit/var/images/`.

 * Commands the Pi to shut down, then waits until the Pi stops responding to pings over UART.

 * Waits 10s.

 * Powers off the Pi by turning off the 5V0 power rail via the `gpio` command.

## Acquisition (via qbapp)

_Estimated duration: 2 minutes._

Push the images volume (type: `var`, name: `images`) to Git.

The images will then be accessible via your Github repository, which you can view in the web browser.
