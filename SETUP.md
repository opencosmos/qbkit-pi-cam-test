# qbkit Raspberry Pi camera demo

This demonstrates controlling power for and communicating with a Raspberry Pi via UART.
It powers on the Pi, waits for it to boot up, commands it to take a photograph using the Pi Camera, then downloads the photograph to the qbkit.
All data is transferred between qbkit and payload via 230400-baud UART.

Software and data is transferred between the qbkit and the operator (yourself) via Github in this case.
Note that by default, anything pushed to Github is publically visible to the world.

The qbkit is not restricted to Github however, if you use some other Git provider (or host your own private Git servers) then qbkit will just as readily be able to use those if you grant appropriate access with the qbkit's SSH public key.

In future tarball and file-level access will also be available, however this is intended for quick tweaks only and not for bulk transfer since it will be much slower than using Git.

## Pre set-up

### Github account

Create a [Github](https://github.com) account (or use your existing one).

### Program repository

Open [the repository containing the qbkit pi-camera-demo software](https://github.com/opencosmos/qbkit-pi-cam-test) and fork it to your own profile.

On your fork, go to `Settings` → `Deploy keys`, add the qbkit's SSH public key the list with title `qbkit` and without granting write access.

### Images repository

Create a new repository called `qbkit-pi-cam-images`.

On your new repository, go to `Settings` → `Deploy keys`, add the qbkit's SSH public key the list with title `qbkit` **and grant write access**.

## Set-up (qbkit, via qbapp)

### A. Creating volumes

Create two **Git** volumes on the qbkit.
Use the SSH URLs (`git@...`) not the HTTPS URLs (`https://...`).

1. Program volume (`bin` type) called `pi-cam`.
   Remote is the SSH URL of your fork of the `qbkit-pi-cam-test` repository.

2. Data volume (`var` type) called `images`.
   Remote is the SSH URL of your new `qbkit-pi-cam-images` repository.

### B. Installing software

Pull the program repository to the qbkit.

### C. Creating service

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

In order to compile the server program, will need GNU Make v4.0+ and GCC v6+ to be installed on your Pi.
On Ubuntu, this is acheived with `apt update && apt install build-essential`.
On Arch, this is acheived with `pacman -Sy --needed base base-devel`

The server uses the `raspistill` program, found in `libraspberrypi-bin` package on Raspbian.

### Disable serial terminal, free up the UART

Disable the serial terminal on the Pi.
Removing the `console=ttyAMA0,...` kernel command-line argument from /boot/config.txt should  be sufficient, however it is recommended to use the `raspi-config` tool to disable the serial terminal, and to configure the UART for general purpose use.

### Installation

Copy the program source folder to your Pi to the path `/root/pi-cam-demo` (creating if necessary).
Then run the following commands on the Pi:

	# Compile program by running
	make -BC /root/pi-cam-demo

	# Copy systemd service file to the correct place
	cp /root/pi-cam-demo/systemd/server.service /etc/systemd/system/pi-cam.service

	# Enable the service so it starts automatically on boot-up
	systemctl enable pi-cam.service

	# Start the service (no reboot required)
	systemctl start pi-cam.service

	# Wait a couple of seconds then verify that the service is running
	systemctl status pi-cam.service

## Connecting

 1. Disconnect all cables from the Raspberry Pi.
    If it is connected to any power source other than the qbkit while also connected to the qbkit power output, it will probably break permanently.

 2. Connect the camera to the Pi.

 3. Connect the following pins on the Pi GPIO to the qbkit.

	| 2  | 5V0 out (qbkit power #4 [5V0])
	| 4  | (nothing)
	| 6  | Ground (qbkit power #3 [GND])
	| 8  | UART TX (qbkit data #10 [UART RX])
	| 10 | UART RX (qbkit data #9 [UART TX])

 4. Ensure that both the qbkit and the Pi have their SD cards properly inserted.

 5. Connect the power supply to the qbkit.

 6. Ensure your computer has an internet connection, and is configured to share the connection with the qbkit.
    This can be fiddly (especially on Windows), however we are working to create dedicated drivers for qbkit on Windows and OSX in order to avoid this.
    Linux users can achieve this with a couple of iptables rules (masquerading + forwarding), enabling ip forwarding via sysctl/procfs, then starting a dhcp server on the qbkit network interface, when it appears on your system.

 7. Connect the qbkit USB to the computer.
    The qbkit will power on and blue LEDs should flash.
    A running-leds ("Knight Rider") pattern should appear once the qbkit has booted up.

## Running (via qbapp)

Start the service `capture-image`.

This will start the `program` from the program volume `pi-cam`, which then does the following:

 * Powers on the Pi by turning on the 5V0 power rail via the `gpio` command.

 * Waits until the Pi starts responding to pings over UART.

 * Waits 10s.

 * Captures an image, with the name `image-{iso8601-date/time}.jpg`.

 * Downloads the image to the qbkit (over UART), saving into data volume `/qbkit/var/images/`.

 * Commands the Pi to shut down, then waits until the Pi stops responding to pings over UART.

 * Waits 10s.

 * Powers off the Pi by turning off the 5V0 power rail via the `gpio` command.

## Acquisition (via qbapp)

Push the images volume (type: `var`, name: `images`) to Git.

The images will then be accessible via your Github repository, which you can view in the web browser.
