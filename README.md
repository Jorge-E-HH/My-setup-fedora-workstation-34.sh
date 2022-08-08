# My-setup-fedora-workstation-34.sh

#!/usr/bin/env bash
# References:
#
# * <https://github.com/smvarela/fedora-postinstall>.
# * <https://mutschler.eu/linux/install-guides/fedora-post-install/>.
# * <https://docs.fedoraproject.org/en-US/fedora/f34/system-administrators-guide/basic-system-configuration/System_Locale_and_Keyboard_Configuration/>.
# * <https://docs.fedoraproject.org/en-US/fedora/f34/system-administrators-guide/kernel-module-driver-configuration/Working_with_the_GRUB_2_Boot_Loader/>.
# * <https://docs.fedoraproject.org/en-US/fedora/f34/system-administrators-guide/kernel-module-driver-configuration/Working_with_Kernel_Modules/>.
# * <https://wiki.archlinux.org/index.php/silent_boot>.
# * <https://rpmfusion.org/Howto/NVIDIA>.
set -eufo pipefail

timedatectl set-timezone Europe/London
timedatectl set-ntp yes
# # Warning: The system is configured to read the RTC time in the local time zone.
# #          This mode cannot be fully supported. It will create various problems
# #          with time zone changes and daylight saving time adjustments. The RTC
# #          time is never updated, it relies on external facilities to maintain it.
# #          If at all possible, use RTC in UTC by calling
# #          'timedatectl set-local-rtc 0'.
# timedatectl set-local-rtc 0

function kernel_parameters_configuration() {
  echo '-------------------------------------------------------------------------------'
  # rd.driver.blacklist=nouveau modprobe.blacklist=nouveau nvidia-drm.modeset=1 rd.driver.blacklist=nouveau modprobe.blacklist=nouveau nvidia-drm.modeset=1
  sudo grubby --update-kernel=ALL --args="vconsole.keymap=gb"
  sudo grubby --update-kernel=ALL --args="loglevel=0 rd.udev.log_priority=0 rd.systemd.show_status=0"
  sudo grubby --update-kernel=ALL --args="libata.force=1.00:disable acpi_enforce_resources=lax amdgpu.ppfeaturemask=0xffffffff"
  sudo grubby --update-kernel=ALL --args="zswap.enabled=1 zswap.max_pool_percent=25 zswap.compressor=lz4hc"
  sudo grubby --update-kernel=ALL --remove-args="rhgb quiet" --args="rd.plymouth=0 plymouth.enable=0 logo.nologo nosplash verbose"
  echo '-------------------------------------------------------------------------------'
}
kernel_parameters_configuration

function configure_dnf() {
  echo '-------------------------------------------------------------------------------'
  echo 'max_parallel_downloads=20' | sudo tee -a /etc/dnf/dnf.conf
  echo 'fastestmirror=true' | sudo tee -a /etc/dnf/dnf.conf
  echo 'deltarpm=true' | sudo tee -a /etc/dnf/dnf.conf
  # https://dnf-plugins-extras.readthedocs.io/en/latest/
  # sudo dnf install dnf-plugins-core
  # sudo dnf install dnf-plugin-kickstart
  # sudo dnf install dnf-plugin-rpmconf
  # sudo dnf install dnf-plugin-showvars
  # sudo dnf install dnf-plugin-snapper
  # sudo dnf install dnf-plugin-system-upgrade
  # sudo dnf install dnf-plugin-torproxy
  # sudo dnf install dnf-plugin-tracer
  sudo dnf install \
    dnf-plugins-core \
    dnf-plugin-showvars \
    dnf-plugin-tracer
  echo '-------------------------------------------------------------------------------'

}
configure_dnf

sudo dnf install fedora-workstation-repositories
sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm

sudo dnf groupupdate core
sudo dnf groupupdate multimedia --setop="install_weak_deps=False" --exclude=PackageKit-gstreamer-plugin
sudo dnf groupupdate sound-and-video

sudo dnf install -y rpmfusion-free-release-tainted
sudo dnf install libdvdcss

sudo dnf install -y rpmfusion-nonfree-release-tainted

sudo dnf config-manager --set-enabled fedora-cisco-openh264
sudo dnf config-manager --set-enabled google-chrome
sudo dnf config-manager --set-enabled rpmfusion-nonfree-nvidia-driver
sudo dnf config-manager --set-enabled rpmfusion-nonfree-steam
sudo dnf config-manager --set-enabled rpmfusion-free
sudo dnf config-manager --set-enabled rpmfusion-free-updates
sudo dnf update -y

sudo dnf install -y \
  vim bash-completion \
  kernel-devel-$(uname -r) kernel-headers-$(uname -r) \
  git git-lfs wget curl util-linux-user \
  make \
  coreutils tree coreutils-common progress nmap arp-scan iproute net-tools iputils \
  powerline vim-powerline tmux-powerline powerline-fonts fontawesome-fonts \
  google-chrome-stable \
  steam \
  gnome-extensions-app gnome-extensions-app gnome-tweaks gnome-shell-extension-appindicator \
  gnome-shell-extension-pop-shell \
  mozilla-fira-fonts-common mozilla-fira-mono-fonts mozilla-fira-sans-fonts fira-code-fonts \
  fira-code-fonts 'mozilla-fira*' 'google-roboto*' \
  jq \
  timeshift \
  yubikey-manager \
  gpg gnupg2 \
  p7zip p7zip-plugins gzip xz bzip2 lzo lz4 lzma libknet1-compress-lz4-plugin \
  kernel-devel-$(uname -r) kernel-headers-$(uname -r) \
  coreutils util-linux tree jq parallel ShellCheck shfmt \
  corectrl \
  dnfdragora \
  mesa-demos vulkan-tools vkmark \
  mangohud goverlay \
  neofetch lm_sensors hw-probe \
  inkscape gimp

# MANGOHUD=0 vkmark --present-mode immediate

function install_and_use_fonts() {
  sudo dnf copr enable peterwu/iosevka
  sudo dnf install -y \
    '*iosevka*'
  echo '-------------------------------------------------------------------------------'
  # https://gist.github.com/alokyadav15/c3a2bbe6089ceff286215113bd092703
  TEMP_DIR="$(mktemp -d)"
  cd "${TEMP_DIR}"

  # git clone --depth=1 git@github.com:mozilla/Fira.git ./mozilla-Fira
  git clone --depth=1 https://github.com/mozilla/Fira.git ./mozilla-Fira
  sudo cp -r ./mozilla-Fira /usr/share/fonts/mozilla-Fira

  fc-cache --force --really-force --system-only --verbose /usr/share/fonts-mozilla-Fira || true
  fc-cache --force --really-force --system-only --verbose /usr/share/fonts/mozilla-Fira || true
  sudo fc-cache --force --really-force --system-only --verbose /usr/share/fonts-mozilla-Fira || true
  sudo fc-cache --force --really-force --system-only --verbose /usr/share/fonts/mozilla-Fira || true
  fc-cache --force --really-force --verbose || true
  fc-cache --force --really-force --system-only --verbose || true

  gsettings set org.gnome.desktop.interface document-font-name 'Fira Sans Regular 11'
  gsettings set org.gnome.desktop.interface font-name 'Fira Sans Regular 11'
  gsettings set org.gnome.desktop.interface monospace-font-name 'Fira Mono Regular 11'
  gsettings set org.gnome.desktop.wm.preferences titlebar-font 'Fira Sans Bold 11'
  gsettings set org.gnome.desktop.interface text-scaling-factor 1.0

  gsettings set org.gnome.desktop.interface font-antialiasing 'rgba'
  gsettings set org.gnome.desktop.interface font-rgba-order 'rgb'
  gsettings set org.gnome.desktop.interface font-hinting 'slight'

  cd ~
  rm -fr "${TEMP_DIR}"
}
install_and_use_fonts

sudo dnf upgrade --refresh
sudo dnf groupupdate core

function flatpak_configuration() {
  echo '-------------------------------------------------------------------------------'
  # https://developer.fedoraproject.org/deployment/flatpak/flatpak-usage.html
  # https://docs.fedoraproject.org/en-US/flatpak/
  flatpak remote-add --system -vv --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
  flatpak remote-add --system -vv --if-not-exists fedora oci+https://registry.fedoraproject.org
  flatpak update --system -vv
  flatpak install --system -vv flathub org.signal.Signal
  flatpak install --system -vv flathub com.skype.Client
  flatpak install --system -vv flathub us.zoom.Zoom
  flatpak install --system -vv flathub com.vscodium.codium
  flatpak install --system -vv flathub com.visualstudio.code
  flatpak install --system -vv flathub com.visualstudio.code-oss
  flatpak install --system -vv flathub org.mozilla.firefox
  echo '-------------------------------------------------------------------------------'
}
flatpak_configuration

function install_vscode() {
  echo '-------------------------------------------------------------------------------'
  cd /tmp
  sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
  sudo sh -c 'echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/vscode.repo'
  sudo dnf check-update
  sudo dnf install code
  xdg-mime default code.desktop text/plain
  echo 'fs.inotify.max_user_watches=524288' | sudo tee -a /etc/sysctl.conf
  echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
  sudo sysctl -p
  echo '-------------------------------------------------------------------------------'
}
install_vscode

sudo dnf groupupdate sound-and-video
sudo dnf install libdvdcss
sudo dnf install gstreamer1-plugins-{bad-\*,good-\*,ugly-\*,base} gstreamer1-libav --exclude=gstreamer1-plugins-bad-free-devel ffmpeg gstreamer-ffmpeg
sudo dnf install lame\* --exclude=lame-devel
sudo dnf group upgrade --with-optional Multimedia

sudo dnf config-manager --set-enabled fedora-cisco-openh264
sudo dnf install -y gstreamer1-plugin-openh264 mozilla-openh264

sudo dnf group install 'Development Tools'
sudo dnf install \
  make \
  cargo \
  rust rust-debugger-common rust-doc rust-gdb rust-lldb rust-src rust-std-static rustfmt \
  golang \
  nodejs npm yarnpkg nodejs-typescript

sudo dnf install vala libvala libgee-devel vte291-devel json-glib-devel

function dracut_configuration() {
  echo '-------------------------------------------------------------------------------'
  # https://gist.github.com/raymanfx/7b672c9fa59996a73c049e507f33fafb
  echo 'hostonly="yes"' | sudo tee -a /etc/dracut.conf.d/lz4.conf
  echo 'compress="lz4"' | sudo tee -a /etc/dracut.conf.d/lz4.conf
  echo 'add_drivers+="lz4hc lz4hc_compress"' | sudo tee -a /etc/dracut.conf.d/lz4.conf
  echo '-------------------------------------------------------------------------------'
}
dracut_configuration

function modprobe_configuration() {
  echo '-------------------------------------------------------------------------------'
  # https://www.reddit.com/r/RetroPie/comments/aakkop/xbox_one_s_controller_disable_ertm_persist_on/
  echo 'options bluetooth disable_ertm=1' | sudo tee -a /etc/modprobe.d/bluetooth-xbox-one-s.conf
  # https://wireless.wiki.kernel.org/en/users/drivers/iwlwifi
  echo 'options iwlmvm power_scheme=1' | sudo tee -a /etc/modprobe.d/iwlmvm.conf
  # https://wireless.wiki.kernel.org/en/users/drivers/iwlwifi
  echo 'options iwlwifi power_level=5' | sudo tee -a /etc/modprobe.d/iwlwifi.conf
  # https://nullr0ute.com/2021/03/setting-the-wireless-regulatory-domain/
  # https://wireless.wiki.kernel.org/en/developers/regulatory/crda
  # https://www.linuxquestions.org/questions/slackware-14/network-manager-wifi-regional-settings-4175559295/
  # http://cachestocaches.com/2016/1/disabling-ubuntus-broken-wi-fi-driver/
  echo 'options cfg80211 ieee80211_regdom=GB' | sudo tee -a /etc/modprobe.d/cfg80211.conf
  sudo iw reg set GB
  echo '-------------------------------------------------------------------------------'
}
modprobe_configuration

function ssh_configuration() {
  echo '-------------------------------------------------------------------------------'
  echo 'TODO: ssh_configuration'
  echo '-------------------------------------------------------------------------------'
}
ssh_configuration

echo '-------------------------------------------------------------------------------'
yes | sudo sensors-detect
echo '-------------------------------------------------------------------------------'

function regenerate_grub() {
  echo '-------------------------------------------------------------------------------'
  sudo dracut --force --verbose --regenerate-all
  sudo grub2-mkconfig -o /boot/grub2/grub.cfg
  sudo dracut --force --verbose --regenerate-all
  echo '-------------------------------------------------------------------------------'
}
regenerate_grub

function shell_configuration() {
  echo '-------------------------------------------------------------------------------'
  sudo dnf install -y git git-lfs vim curl wget zsh zsh-syntax-highlighting zsh-autosuggestions
  sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
  sudo usermod -s "$(which zsh)" "${USER}"
  echo '-------------------------------------------------------------------------------'
}
shell_configuration

echo '-------------------------------------------------------------------------------'
sudo dnf autoremove
echo '-------------------------------------------------------------------------------'

sudo systemctl reboot
