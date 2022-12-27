# Arch on Older Thinkpads: The Master Reference

Linux has really, really bad support for certain device drivers, like Nvidia drivers. The problem is amplified when you're dealing with *Legacy* Nvidia drivers. Thankfully, the thinkpad community has developed a bunch of tricks and programs to overcome various problems; but they're not packaged by default. I've made this article as a reference for myself on any future arch reinstalls, but I figure I may as well expand and publicize it. + I devote a section to Nvidia Legacy drivers on Optimus laptops, like Thinkpads. I know ThinkWiki exists; for me, however, with a Thinkpads W530, it's woefully incompatible, and it's extremely old generally. 

## Pre-Installation

These are some tips for the pre-installation. They vary depending on your bios (propietary, coreboot, libreboot) and device make, and aren't obligatory, but are helpful regardless. Some of the tweaks in this article may not work if you don't follow these tips, so you'll have to work around that:

- Turn off SecureBoot 
- Turn on UEFI; disable Legacy (for GRUB, which makes life easier & simplre)
- If you have an Nvidia card, enable Nvidia Optimus and force OS-specific optimizations

Next comes booting onto an Arch live-installation. First, download the ArchISO. Make sure to verify its checksum. You have a couple of options next:

- Booting through a USB
- Booting through a CD-ROM
- Booting through the internet

For booting through a USB or a CD-ROM with a Windows start, the recommended choice is generally Rufus; note that, in my experience, Arch does not work using Rufus **UNLESS YOU USE DD mode**. If you're on MacOS, or already on Linux, just use the dd command. 

`dd bs=4M if=path/to/archlinux-version-x86_64.iso of=/dev/sdx conv=fsync oflag=direct status=progress`  

I plan to add a section on PXE booting.

Once you've got your live USB/disk ready, reboot your device and spam F12, or the ThinkVantage button. Select your device in the quick boot menu, and you should boot to the arch installation screen.

## Installation

See the ArchWiki ;)

## Post-Installation

On your first reboot, you should be booted to the tty, unless you installed a WM/DE during the install. Before installing a GUI, you should download some daemons:

`
sudo pacman -S networkmanager alsa alsa-utils
sudo systemctl enable NetworkManager.service
` 

Next, you should configure your `pacman.conf` and run reflector. Open `/etc/pacman.conf` with sudo privileges using your preferred text editor. Scroll down to 'multilib' and uncomment the two lines. Next, in 'Misc options', uncomment ParallelDownloads and increase it to 10 (this is quite intensive). To run reflector, install the package with `sudo pacman -S reflector`, and run the command `sudo reflector --latest 20 --protocol https --sort age --save /etc/pacman.d/mirrorlist`.

Now, we will install yay, the AUR helper. You can install your choice of AUR helper, but AUR will be needed for the rest of the guide. 

`
cd /opt/
sudo git clone https://aur.archlinux.org/yay-git
sudo chown -R tecmint:tecmint ./yay-git
cd yay-git
makepkg -si
sudo yay -Syu
`

Onto a more important step, let's perform some system-specific optimization: `yay -S tlp tlp-rdw acpi acpi_call tlpui` to install the necessary packages. Next, run the following commands:

`
sudo fstrim.timer enable
sudo systemctl enable tlp
sudo systemctl mask systemd-rfkill.service
sudo systemctl mask systemd-rfkill.socket
`

Once done, disable intel-pstate by adding `intel_pstate=disable` to `/etc/default/grub` in the line `GRUB_CMDLINE_LINUX_DEFAULT=`. Run `sudo tlpui` and configure as per your preference; make sure to manually choose your CPU scaling governors as p-state will not handle them for you now. You may want to limit your battery threshold to 80-90 if you use your device primarily on AC.
