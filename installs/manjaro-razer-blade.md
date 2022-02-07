# Installation Process
Partially cribbed from [https://github.com/pkerichang/linux_files/blob/master/README_arch_razer_15_2018.md].

## Get yay package manager
```
mkdir -p Code
cd Code
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
cd ~
```

## Install tmux and vim
```
sudo yay -S tmux
sudo yay -S vim
```

## Update system
Vim wouldn't work because it depended on libraries newer
than are installed on the system. I googled it and it
looks like the whole thing is out of date by default,
which isn't that unusual.

https://forum.manjaro.org/t/cant-open-vim-glibc-2-29-not-found/89583
```
sudo pacman-mirrors -f3
sudo pacman -Syyu
```

When running the above commands, pacman asked me if I wanted
to replace a bunch of packages with different packages. I had
no idea what it was talking about so I just said `y` to
everything. Seems legit.

## Setup touchpad
With the new and improved system update, the touchpad settings
UI had more stuff in it. I configured the scrolling, pointer
speed, etc the way I like (previously not all those options
were available -- they were greyed out and the UI layout was
different).

## Installing Optimus Manager
*I don't recommend this, it didn't work. Keeping this section for posterity.*

From https://github.com/pkerichang/linux_files/blob/master/README_arch_razer_15_2018.md
```
yay -S optimus-manager
sudo systemctl enable optimus-manager.service
sudo systemctl start optimus-manager.service
optimus-manager --set-startup intel
```

To switch video card, run `optimus-manager --switch <intel/nvidia>`.

## Keyboard Backlight
*I don't recommend this, it didn't work. Keeping this section for posterity.*
*See instead: Keyboard Backlight: Round 2*

```
yay -S openrazer-meta
```
It asked which provider I wanted for a few packages, I chose
the ones that did _not_ have a `-git` suffix.

*Reboot*

## Disable Bumblebee
*I don't recommend this, it didn't work. Keeping this section for posterity.*

Bumblebee interferes with Optimus apparently.

```
sudo systemctl disable bumblebeed.service
```

This requires another restart, but first I went ahead and opened
the Hardware Configuration GUI and selected the Nvidia drivers.

I clicked the "Auto Install Proprietary Driver" button.
It said "Starting" for a long time without doing anything, so 
closed it and decided to reboot first.

Machine hung trying to reboot, so I hard shut it off O.o. It
restarted just fine after that though, and running

```
optimus-manager --print-mode
```

Displayed `intel` as expected.

However, when I ran `optimus-manager --switch nvidia`, after the
GUI rebooted it still displayed `intel`, which isn't right.

So I opened the Hardware Configuration again. I tried to right
click the nvidia driver (not the 390xx one, which the internet
says is for legacy compatibility), but it said it conflicted with
the bumblebee one. So I right clicked and removed the bumblebee
one, then tried to install the `video-nvidia` one again. And it
said it worked.

Reboot!

Still no dice with the display manager. Switching gears and
trying out the chroma stuff.

Had to run `sudo gpasswd -a gmalmquist plugdev` and then a
`sudo reboot`, then `yay -sS polychromatic` to get it to
install properly.

Unfortunately, when I loaded up the Polychromatic Controller
it said it couldn't recognize my devices. That's sad, because
the Razer Blade is specifically listed on their website.

Okay I give up. I'm switching back to bumblebee. It seems fine.

[https://wiki.archlinux.org/index.php/bumblebee#Bumblebee:_Optimus_for_Linux]
```
optirun glxgears -info
optirun -b none nvidia-settings -c :8
```

## Keyboard Backlight: Round 2
After much fiddling and googling, this worked:

Uninstall package:
```
sudo pacman -R openrazer-meta
```

Install linux headers:
```
yay -S linux
yay -S linux419-headers
```
Note that you can see which kernel you're running by
opening the Manjaro `Kernel` GUI utility.

Alternatively, the kernels are also listed in `/lib/modules/`,
but I don't know how to tell which one is active if there are
multiple there.

In my case, I'm on `4.19.50_rt22-1`, and the first two parts
of that semver get smooshed to make `419`.

Install `pacaur`, which handles a weird edge-case in the
`openrazer-meta` package better, as recommended by the
[openrazer download page](https://openrazer.github.io/#download).
```
yay -S pacaur
```

Install openrazer-meta with pacaur:
```
pacaur -S openrazer-meta
```

*Note*: I had already run these two commands earlier, otherwise you
have to run them now:
```
sudo gpasswd -a gmalmquist plugdev
yay -S polychromatic
```

*Reboot* for the drivers to take effect. Then run the
`Polychromatic Tray Applet` and you're good.

## Setup git
```
git config --global user.email "gdmalmquist@gmail.com"
git config --global user.name "Gwen Malmquist"
git config --global core.editor vim
```

## Install Xclip
```
yay -S xclip
```

## Create SSH Keys
```
ssh-keygen
```
And [upload to github](https://github.com/settings/keys) the contents of `~/.ssh/id_rsa.pub`.
Also uploaded to gitlab, but gitlab's ssh remote doesn't seem to work well, so I'm using https
anyway :/.

## Install Java
Downloaded latest java jdk8 tar.gz from oracle.
```
cd /usr/lib
sudo mkdir jvm
cd jvm
tar -xzvf ~/Downloads/jdk-8u211-linux-i586.tar.gz
```

Then added this to the end of `/etc/profile`:
```
JAVA_HOME='/usr/lib/jvm/jdk1.8.0_211'
PATH="$PATH:$JAVA_HOME/bin"
export JAVA_HOME
export PATH
```

Have to restart the terminal to take effect, or
just:
```
source /etc/profile
source ~/.profile
source ~/.bashrc
```

## Add git bash completion
Append to `~/.bashrc`:
```
source /usr/share/git/completion/git-completion.bash
```

## Install IntelliJ
Download [https://download-cf.jetbrains.com/idea/ideaIC-2019.1.3.tar.gz].
```
cd /usr/lib
sudo tar -xzvf ~/Downloads/ideaIC-2019.1.3.tar.gz
```

Create `/usr/local/bin/idea` with:
```
#!/bin/bash
/usr/lib/idea-IC-191.7479.19/bin/idea.sh "$@"
```

```
sudo chmod +x /usr/local/bin/idea
```

## Installing Software
```
yay -S inkscape
yay -S gimp
```

## Disabled tmux mouse mode
I decided I didn't like it.

Commented line out of tmux conf, then ran:
```
setw -g mouse off
```

## Japanese Keyboard Input
```
sudo pacman -S ibus
yay -S ibus-mozc
```
Widget shows up in taskbar, but doesn't do anything currently.

Added this:
```
# /etc/profile
export GTK_IM_MODULE=ibus
export XMODIFIERS=@im=ibus
export QT_IM_MODULE=ibus
```

## MultiMC
```
yay -S multimc5
```

Selected the option to use jre8-openjdk and jdk8-openjdk.

Then hit super and ran MultiMC. On initial setup it asked
which installed JDK to use, and I picked the oracle jdk8
I had already installed, rather than using the openjdk
one that the package manager installed.

However, upon creating a Minecraft vanilla instance and
launching it, it failed to launch due to invalid heap size.
Maybe I accidentally installed a 32 bit jvm? Not sure, but
I switched to openjdk8 and it was able to launch it.

## Window Management Stuff
Install `xdotool` to allow automated X window management.
```
yay -S xdotool
```

Bind window management keys mirroring my moom setup, by
opening the `System Settings` menu, and adding new
custom shortcuts at `Shortcuts > Custom Shortcuts`. I put
the shortcuts in a new group called "Goom".

### Bind CTRL 1, 2, 3 to vertical thirds
`ctrl+1`
```
xdotool getactivewindow windowmove 0 0 windowsize 33% 100%
```

`ctrl+2`
```
xdotool getactivewindow windowmove 33% 0 windowsize 33% 100%
```

`ctrl+3`
```
xdotool getactivewindow windowmove 67% 0 windowsize 33% 100%
```

### Bind CTRL 4, 5 to first and last vertical sixths
`ctrl+4`
```
xdotool getactivewindow windowmove 0 0 windowsize 16.5% 100%
```

`ctrl+5`
```
xdotool getactivewindow windowmove 83.3% 0 windowsize 16.5% 100%
```

### Bind CTRL+SHIFT 1, 2, 3, 4 to vertical fourths
`ctrl+shift+1` (`ctrl+!`)
```
xdotool getactivewindow windowmove 0 0 windowsize 25% 100%
```

`ctrl+shift+2` (`ctrl+@`)
```
xdotool getactivewindow windowmove 25% 0 windowsize 25% 100%
```

`ctrl+shift+3` (`ctrl+#`)
```
xdotool getactivewindow windowmove 50% 0 windowsize 25% 100%
```

`ctrl+shift+4` (`ctrl+$`)
```
xdotool getactivewindow windowmove 75% 0 windowsize 25% 100%
```

## Install tk
So I can do python tkinter things.
```
sudo pacman -S tk
```

This initially failed, but succeeded when I tried again after doing a system upgrade which I'd been putting off. I noticed that the wiki for pacman said to do upgrades before installing new stuff, so I guess that's why. Interesting.

Wrote up a python script called `goom` to manage X windows programatically, and
added it to the path:
`/etc/profile`
```
export PATH=${PATH}:/home/gmalmquist/Code/unix-config/bin
```

Changed all the shortcuts in `window management` to `goom` ones, and also added:

`Super+Shift+M`
```
goom key-chord
```

`Alt+Shift+M`
```
goom key-chord
```

Sometimes the windows can't be resized vertically, because kwin has them in a
state where they're snapped to be the full vertical height. Manually resizing
them first fixes this, but I just turned off window snapping to avoid this
problem:

`System Settings > Window Management > Window Behavior`
If you set the snap zone size of everthing to `0`, it disables snapping.

## Visual Studio Code
```
pacaur -S aur/visual-studio-code-bin
```

## Bump fs watches
Follow instructions here

```
https://github.com/webpack/docs/wiki/troubleshooting#not-enough-watchers
```

## Auto Mount Blade HDD
Added this to my `/etc/fstab`:

```
UUID=5C8C8D8D8C8D61FE /run/media/bladehdd ntfs    defaults,noatime,discard,umask=0002,gid=1000,uid=1000 0 2
```

I found the UUID by running `lsblk -f`.

## Setting up Unity3D + Visual Studio Code
Install Unity from the unity website. It'll drop a `.AppImage` file in
`~/Applications`. You won't be able to launch it normally, but running
`primusrun ~/Applications/UnityHub*.AppImage` works.

Configure where you want it to put Unity installs. I did set it to
`/opt/Unity3D`, which I first created with:

```
sudo mkdir /opt/Unity3D
sudo chown gmalmquist:gmalmquist /opt/Unity3D
```

Installed the latest stable version, then created a project. It likes
creating the projects in the home directory, so I closed the UnityHub,
manually moved the project to my subdirectory of choice, relaunched
the hub, re-added the project (Add rather than New, then select the
folder), and launched it.

This worked for unity, but more work to do to get Visual Studio Code
working with it.

Loosely followed: https://medium.com/@sami1592/set-up-visual-studio-code-for-unity-in-linux-69b7f4352e0b

I say loosely because I had to improvise / google a lot of stuff
because everything assumes Debian flavors.

To get dotnet, I followed the instructions here:
https://snapcraft.io/install/dotnet-sdk/manjaro
```
sudo pacman -S snapd
sudo systemctl enable --now snapd.socket
sudo ln -s /var/lib/snapd/snap /snap
```

Restart the computer here or else the next step will claim it
completes successfully, but actually will not.
```
sudo snap install dotnet-sdk --classic
```

Install mono:
```
sudo pacman -S mono
```

Restart again. Open unity, change editor preference to vs code,
as per the medium article linked above.

> Step 3 : Change your code editor preference in Unity.
> Unity->preferences->External tools: /usr/bin/code

## Android Studio
Followed this: https://linuxconfig.org/how-to-install-android-studio-on-manjaro-18-linux

```
git clone https://aur.archlinux.org/android-studio
cd android-studio/
makepkg -s
ls *pkg.tar.xz
sudo pacman -U --noconfirm android-studio-3.2.1.0-1-x86_64.pkg.tar.xz
android-studio
```

## Rust
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
