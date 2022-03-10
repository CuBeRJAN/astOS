# astOS (Arch Snapshot Tree OS)
### An immutable Arch based distribution utilizing btrfs snapshots  

---
## What is astOS?  

astOS is a distribution based on Arch Linux.  
Unlike Arch it uses an immutable (read-only) root filesystem.  
Software is installed and configured into individual snapshot trees, which can then be deployed and booted into.  
It doesn't use it's own package format or package manager, instead using pacman from Arch

astOS supports containers such as docker or podman for modern software development workflows.  
It is also suitable for embedded systems, as setting up a minimal install is fairly easy and it's features can be really useful in this context


**This has several advantages:**

* Security
  * Even if running an application with eleveted permissions, it cannot do stuff like replace system libraries with malicious version
* Stability 
  * Due to the system being mounted as read only, it's not possible to accidentally overwrite system files
  * If the system runs into issues, you can easily rollback the last working snapshot within minutes
  * Atomic updates - Updating your system all at once is safer
  * Thanks to the snapshot feature, astOS can ship cutting edge software without becoming unstable
* Configurability
  * With the overlays organised into a tree, you can easily have multiple different configurations of your software available, with varying packages, without any interference
  * For example: you can have a single Gnome desktop installed and then have 2 overlays on top - one with your video games, with the newest kernel and drivers, and the other for work, with the LTS kernel and more stable software, you can then easily switch between these depending on what you're trying to do
  * You can also easily try out software without having to worry about breaking your system or polluting it with unnecessary files, for example you can try out a new desktop environment in a snapshot and then delete the snapshot after, without modifying your main system at all
  * This can also be used for multi-user systems, where each user has a completely separate system with different software, and yet they can share certain packages such as kernels and drivers
  * astOS allows you to install software by chrooting into overlays, therefore you can use software such as the AUR to install additional packages
  * astOS is, just like Arch, very customizable, you can choose exactly which software you want to use

---
## astOS compared to other similar distributions
* **NixOS** - compared to nixOS, astOS is a more traditional system with how it's setup and maintained. While nixOS is entirely configured using the Nix programming language, astOS uses Arch's pacman package manager. astOS uses btrfs snapshots while NixOS builds squashfs system images.
  * astOS allows declarative configuration using Ansible, for similar functionality to NixOS
* **Fedora Silverblue** - astOS is more customizable, but does require more manual setup.
* **OpenSUSE MicroOS** - astOS is a more customizable system, but once again requires a bit more manual setup. MicroOS works similarly to astOS in the way it utilizes btrfs snapshots.

---
## Installation
* astOS is installed from the official Arch Linux live iso available on [https://archlinux.org/](https://archlinux.org)

Install git first - this will allow us to download the install script

```
pacman -Sy git
```
Clone repository

```
git clone "https://github.com/CuBeRJAN/astOS"  
cd astOS  
```
Partition and format drive

```
lsblk  # Find your drive name
cfdisk /dev/*** # Format drive, make sure to add EFI partition, if using BIOS leave 2M before partition  
```
Run installer

```
python3 main.py /dev/<partition> /dev/<drive> /dev/<efi part> # You can skip the EFI partition if installing in BIOS mode
```

## Post installation setup
* astOS doesn't do much setup for the user, therefore some post-installation setup is going to be necessary
* A lot of information for how to handle post-install setup is available on the [ArchWiki page](https://wiki.archlinux.org/title/general_recommendations) 
* Here is a small example setup procedure:
  * Start by creating a new snapshot from the base image using ```ast clone 0```
  * Chroot inside this new snapshot (```ast chroot <snapshot>```) and begin setup
    * Start by adding a new user account: ```useradd username```
    * Set the user password ```passwd username```
    * Now set a new password for root user ```passwd root```
    * Now you can install additional packages (desktop environments, container technologies, flatpak) using pacman
    * Once done, exit the chroot with ```exit```
    * Then you can deploy it with ```ast deploy <snapshot>```

## Additional documentation
* It is advised to refer to the Arch wiki for documentation not part of this project

#### Base image
* The snapshot ```0``` is reserved for the base system image, it cannot be changed and can only be updated using ```ast base-update```

#### Show filesystem tree

```
ast tree
```

* The output can look for example like this:

```
root - root
├── 0 - base image
└── 1 - multiuser system
    └── 4 - applications
        ├── 6 - MATE full desktop
        └── 2*- Plasma full desktop
```
* The asterisk shows which snapshot is currently selected as default

* You can also get only the number of the currently booted snapshot with

```
ast current
```
#### Add descritption to snapshot
* Snapshots allow you to add a description to them for easier identification

```
ast desc <overlay> <description>
```
#### Delete a tree
* This removes the tree and all it's branches

```
ast del <tree>
```
#### Software installation
* Run ```ast deploy``` and reboot after installing new software for changes to apply
* Software can also be installed using pacman in a chroot
* AUR can be used under the chroot
* Flatpak can be used for persistent package installation
* Using containers for additional software installation is also an option, the advantage is no need for a reboot. A recommended way of doing this is with [distrobox](https://github.com/89luca89/distrobox)

```
ast install <overlay> <package>
```
* After installing you can sync the newly installed packages to all the branches of the tree with

```
ast sync <tree>
```
#### Updating
* It is advised to clone a snapshot before updating it, so you can roll back in case of failure

* To update a single snapshot

```
ast upgrade <snapshot>
```
* To recursively update an entire tree

```
ast tree-update <overlay>
```

* ast also supports automatic updates, these will automatically upgrade and deploy current snapshot, but will not deploy if the update fails

```
ast auto-upgrade
```
* To check the status and date of the last automatic upgrade run

```
ast check
``` 

* This can be configured in a script (ie. a crontab script) for easy and safe automatic updates

#### Custom boot configuration
* If you wish to use a custom grub configuration, chroot into an overlay and edit /etc/default/grub, then deploy the snapshot and reboot

#### chroot into snapshot 
* Once inside the chroot the OS behaves like regular Arch, so you can install and remove packages using pacman or similar
* Do not run ast from inside a chroot, it could cause damage to the system, there is a failsafe in place, which can be bypassed with ```--chroot``` if you really need to (not recommended)  

```
ast chroot <overlay>
```
#### Clone snapshot
* This clones the snapshot as a tree

```
ast clone <snapshot>
```
#### Create new tree branch

```
ast branch <snapshot to branch from>
```
#### Clone snapshot under same parent

```
ast cbranch <snapshot>
```
#### Clone snapshot under specified parent

```
ast ubranch <parent> <snapshot>
```
#### Create new base tree

```
ast new-tree
```
#### Deploy snapshot  

```
ast deploy <snapshot>  
```
#### Update base which new snapshots are built from

```
ast base-update
```

## Known bugs

* When running ast without arguments - IndexError: list index out of range
* Running ast without root permissions shows a lot of errors
* GDM and LightDM may not work  
* Docker has issues with permissions, to fix run

```
sudo chmod 666 /var/run/docker.sock
```


# TODO: more docs
