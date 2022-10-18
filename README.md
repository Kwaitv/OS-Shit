# OS-Shit
Random OS Bullshit

# Linux Bullshit

## Audio Rescan
removes audio PCI-E device and then re-scans for it
```bash
echo 1 > /sys/bus/pci/devices/0000\:00\:1f.3/remove
echo 1 > /sys/bus/pci/rescan
```

## Script run on resume from Suspend of Hibernate
[Run on Resume](https://askubuntu.com/questions/1313479/correct-way-to-execute-a-script-on-resume-from-suspend)
to have a script run on resume from suspend/hibernate copy it to `/lib/systemd/system-sleep/`

## More recent OpengL version on Spice Server display server
[Virtual OpenGL Driver](https://thomas.inf3.ch/2019-06-12-opengl-kvm-mesa3d/index.html)

## GPU Drivers
Requires Secure Boot to be disabled (Nvidia)
### Disabling nvidia GPU
huh `envycontrol` just kinda works for switching gpus
### NVIDIA driver preventing linux from loading (specifically Xserver)
solution was in `/etc/default/grub` adding `nomodeset` to `LINUX_CMDLINE_DEFAULT=`
prefereably just use an `linux-nvidia` type kernel image

### Video Hardware Acceleration playback
[https://reddit.com/r/linux/comments/xcikym/tutorial_how_to_enable_hardware_video/](https://reddit.com/r/linux/comments/xcikym/tutorial_how_to_enable_hardware_video/ "https://reddit.com/r/linux/comments/xcikym/tutorial_how_to_enable_hardware_video/")

## Network
### Netplan instead if NetworkManager isn't enabled
so if ur not running `NetworkManager` and `/etc/netplan/` is empty all network devices will show up as off in `networkctl`

### Utilizing a Linux box as a gateway rather than a switch/router
to use a linux machine as a gateway where `Internet ---- Linux machine ---- ur machine` u need somewhere in `nmcli connection show <wired connection>` to have the field `ipv4.method` be set to `shared` and on ur machine u need to set ur ip to one under the subnet of the `Linux machine` with the `ipv4.routes` and `ipv4.gateway` set to the ip of the Linux machine
u can do it without shared u just need to set routes for all child machines off ur Linux machine (edited)

`nmcli connection add con-name direct ipv4.gateway 10.42.0.1 ipv4.addresses 10.42.0.2/24 ipv4.method manual ipv4.routes 10.42.0.1 type ethernet` example command for `ur machine`

for u master machine a command like `nmcli connection add con-name children ipv4.addresses 10.42.0.1/24 ipv4.method shared master <the interface device like eno1> type ethernet` should work

its a bit different if u do it through `netplan` since u have to manually set the routes not do the `ipv4.method shared` arg

#### Reference
[No swtich Network Topologies](https://www.cnblogs.com/zszmhd/p/3365161.html)
[RedHat setting statuc routes](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/networking_guide/sec-configuring_static_routes_using_nmcli)
## Sudo Stupid
`sudo -S` lets u pipe in the password from stdin

## Qt application segfaults with error saying incompatable Qt version
force update/rebuold `qt5-styleplugins`

## ffmpeg
### `.ogg` to `.mp3` metadata retainment
if u want to retain metadata when converting from `.ogg` to `.mp3` with `ffmpeg` u'll need to add these args `-map 0 -map_metadata 0:s:0 -id3v2_version 3 -write_id3v1` :))
## Discord
if discord wants you to update, just `sudoedit /opt/discord/resources/build_info.json` and increment the `version` field
## Qt complying with GTK theme
for qt apps to follow ur gtk icon theme rather than forcing it to follow ur gtk theme with export `QT_QPA_PLATFORMTHEME="gtk2"` use `export QT_QPA_PLATFORMTHEME="qt5ct"` and use the `qt5ct` package to set it to follow `gtk2` which lets u set the icon theme separately

## CPU Control
`echo 0 > /sys/devices/system/cpu/cpu#/online` will disable a cpu core 
*note: u cant disable cpu0*

### Bullshit battery management
the most annoying this for reason in udev rules some form environment vars gets messed up so setting up a udev rule to shut down all cores but 2 goes like:
writning the udev rule `60-powerCPUmgmt.rules` to `/etc/udev/rules.d/`:
```
#Rule for switching to BAT0
SUBSYSTEM=="power_supply",ENV{POWER_SUPPLY_ONLINE}=="0", RUN+="/bin/batCPU"

#Rule for switching to AC
SUBSYSTEM=="power_supply",ENV{POWER_SUPPLY_ONLINE}=="1", RUN+="/bin/batCPU"
```
with `batCPU` being:
```
#!/usr/bin/bash
[[ $(cat /sys/class/power_supply/AC/online) -eq 0 ]] && \
	(for i in $(seq 2 11); do echo 0 > /sys/devices/system/cpu/cpu$i/online; done) || \
	(for i in $(seq 1 11); do echo 1 > /sys/devices/system/cpu/cpu$i/online; done)
```
where 11 is whatever the max cpu index on your device is
I would use `nproc` to determin the max cpu dynamically but udev shits itself and will only stop cores 2-6

Systemd Service for on Startup
```
[Unit] 
Description=Enables X number of Cpus dependant on power state 

[Service] 
Type=simple 
RemainAfterExit=yes 
Execstart=/bin/batCPU 

[Install] 
WantedBy=default.target
```

## Pacman
### PGP sigs
if you come across a `invalid or corrupted package (PGP signature)` or something along those lines run a `pacman -S archlinux-keyring` and then `pacman -Syu`

# Head-ass Windows
## Chocolaty Powershell install
`Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))` 

## SSH pubkey setup
1. install openssh server with `Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0` 
2. start and enable the sshd service with `Start-Service sshd` and `Set-Service -Name sshd -StartupType 'Automatic'` 
3. allow it through the firewall `New-NetFirewallRule -Name 'OpenSSH-Server-In-TCP' -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22` 
now openssh is installed 
on your client machine 
1. copy ur ssh key with `scp id_rsa.pub <user>@<hostname/ip>:%programdata%/ssh` 
2. now ssh into the windows machine from ur client machine and `cd %programdata%/ssh` 
3. run `type id_rsa.pub >> administrators_authorized_keys` now u'll need a cli editor like vim or nano refer to [[Linux bullshit#Chocolaty Powershell install]] on chocolaty install then run `chocolaty install <editor>` 
4. once u have a cli editor run `<editor> sshd_config` and uncomment the line with `#PubkeyAuthentication yes` 
5. restart the sshd service with `Restart-Service sshd` within powershell
