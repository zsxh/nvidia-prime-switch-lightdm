pkgname="nvidia-prime-switch-lightdm"
pkgver=1
pkgrel=7
pkgdesc="(! use only with lightdm !) Setup nvidia and intel for optimus based laptops without bumblebee (! use only with lightdm !)"
license=("MIT")
install="${pkgname}".install
depends=('xf86-video-intel' 'python' 'lightdm' 'xorg-xrandr')
optdepends=("nvidia: nvidia kernel module(s) for newer gpus"
'lib32-nvidia-utils: 32bit nvidia utils for newer gpus'
'nvidia-utils: nvidia utils for newer gpus'
"nvidia-390xx: nvidia kernel module(s) for older gpus (Fermi)"
'lib32-nvidia-390xx-utils: 32bit nvidia utils for older nvidia gpus (Fermi)'
'nvidia-390xx-utils: nvidia utils for older nvidia gpus (Fermi)')
makedepends=('python')
conflicts=('nvidia-prime-switch' 'nvidia-prime-switch-sddm')
source=('prime-switch.py' 'prime-switch-conf.json' 'display_setup.sh' 'intel.conf' 'nvidia.conf' 'intel-modesetting.conf' '999-nvidia-gpu-power.rules' 'nvidia-prime-displaymanager.hook' 'prime-switch-systray.py' 'prime-switch-systray.desktop')
sha256sums=(
'1a92788e893c462e77ae1340490831e062824ad768818f45d0fa3fa80387ced8'
'193bd60870e1dd484a205e6c2607ffa299ad165530e5904ef0348d7e7378a4bf'
'f280e87f3bd842d0c547d51c4675387e07415aa776c27df24a43f66bb0688cee'
'b7e686d0f689c9d7e2d99ffa6a3b3c110730e36a911b5672f711551b3e41d6a8'
'5ff9c2f17ac10eb42a258f861094ba478fabaa283d11212553e1256b3997dc91'
'edd5b3968e0cf46dcc13a8335f71291b19355c8fc75c8c3af597540fe548c929'
'88ae6059682db82a9f88a28ae1aefb988fbe7e1079f36ca1df02062f422e4c0d'
'547ca622632e234f04900a46e5652ea5c77b252595689b22c8e46f81a800173f'
'5b418bccb30fe3e7e50b71b1f5c2801498b8e1f0e48ac29efb26c1e33fa555c6'
'318d8a6707d3024f83c2ba0d47ff642bece91dabe834cd7719ef8673d4f7a0ad'
)

arch=('x86_64')
prefix="/usr/local"

# with (1) or without (0) udev rule
with_udev=1

# with or without gtk3-systray
with_systray_gtk3=1

if [ $with_udev == 1 ];
  then
    backup=('etc/X11/mhwd.d/intel.conf' 'etc/X11/mhwd.d/intel-modesetting.conf' 'etc/X11/mhwd.d/nvidia.conf' 'etc/lightdm/display_setup.sh' 'etc/udev/rules.d/999-nvidia-gpu-power.rules')
  else
    backup=('etc/X11/mhwd.d/intel.conf' 'etc/X11/mhwd.d/intel-modesetting.conf' 'etc/X11/mhwd.d/nvidia.conf' 'etc/lightdm/display_setup.sh')
fi

if [ $with_systray_gtk3 == 1 ];
  then
    depends+=('libappindicator-gtk3' 'polkit')
fi

prepare() {
# xconfigs
# ensure right buisd (maybe not needed)
pcis=$(python -c '
import os

SYSFS_PATH="/sys/bus/pci/devices"
devs = os.listdir(SYSFS_PATH)
gpus = [ i for i in devs if "0x030000" in open(SYSFS_PATH+"/"+i+"/class").read() or "0x030200" in open(SYSFS_PATH+"/"+i+"/class").read()]
gpuids = {}
# use dict instead of list, use "0" as key for intel and "1" for nvidia
for pciid in gpus:
    with open(SYSFS_PATH+"/"+pciid+"/vendor") as f:
        vendor = f.read().rstrip("\n")
    if vendor == "0x8086":
        gpuids[0] = pciid
    elif vendor == "0x10de":
        gpuids[1] = pciid

# use range instead of dict.values(), print intel pci-id first, also format output to x:x:x for xorg conf file
for i in range(0,2):
    print("%s" %(":".join([str(int(i)) for i in gpuids[i].replace(".",":").split(":")][1:])))
')

i=$(echo ${pcis} | cut -d \  -f 1) # intel config
n=$(echo ${pcis} | cut -d \  -f 2) # nvidia config
sed -i "s/BusID \"PCI:0:2:0\"/BusID \"PCI:${i}\"/" ${srcdir}/intel.conf
sed -i "s/BusID \"PCI:0:2:0\"/BusID \"PCI:${i}\"/" ${srcdir}/intel-modesetting.conf
sed -i "s/BusID \"PCI:1:0:0\"/BusID \"PCI:${n}\"/" ${srcdir}/nvidia.conf
}


package() {
cd "${srcdir}"

install -Dm755 prime-switch.py "${pkgdir}/${prefix}/bin/prime-switch"
install -Dm644 prime-switch-conf.json "${pkgdir}/${prefix}/share/nvidia-prime-switch/prime-switch-conf.json"
install -Dm755 display_setup.sh "${pkgdir}/etc/lightdm/display_setup.sh"
# xconfigs
for i in intel.conf intel-modesetting.conf nvidia.conf
  do
    install -Dm644 ${srcdir}/$i "${pkgdir}/etc/X11/mhwd.d/$i"
done

if [ $with_udev == 1 ]; then install -Dm644 999-nvidia-gpu-power.rules "${pkgdir}/etc/udev/rules.d/999-nvidia-gpu-power.rules"; fi;

# hooks
install -Dm644 "${srcdir}/nvidia-prime-displaymanager.hook" "${pkgdir}/usr/share/libalpm/hooks/nvidia-prime-displaymanager.hook"

# systray
if [ $with_systray_gtk3 == 1 ];
  then
    install -Dm755 "${srcdir}/prime-switch-systray.py" "${pkgdir}/usr/bin/prime-switch-systray"
    install -Dm644 "${srcdir}/prime-switch-systray.desktop" "${pkgdir}/etc/xdg/autostart/prime-switch-systray.desktop"
fi
}
