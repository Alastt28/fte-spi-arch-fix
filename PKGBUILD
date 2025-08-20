# Maintainer: alfaonyt

pkgname=libfprint
pkgver=1.94.4+tod1
pkgrel=1
pkgdesc="Custom libfprint library supporting Focaltech fingerprint SPI driver"
arch=('x86_64')
url="https://github.com/alfaonyt/fte-spi-arch"
license=('LGPL')
depends=()
conflicts=('libfprint') # Conflict with existing libfprint package
provides=('libfprint=1.94.4' 'libfprint-2.so=2-64')
source=(
  "https://raw.githubusercontent.com/ftfpteams/ubuntu_spi/main/focal_spi.c"
  "Makefile"
  "libfprint.deb::https://github.com/ftfpteams/ubuntu_spi/raw/main/libfprint-2-2_1.94.4+tod1-0ubuntu1~22.04.2_spi_amd64_20240620.deb"
  "fprintd.service"  # Add your custom fprintd.service here
)
md5sums=('SKIP' 'SKIP' 'SKIP' 'SKIP')

prepare() {
  cd "$srcdir"

  # Extract the libfprint deb package directly
  ar x libfprint.deb

  # Extract the files from the .zst archives
  if [ -f "data.tar.zst" ]; then
    bsdtar -xf data.tar.zst
  else
    echo "Error: Could not find data.tar.zst in the deb package."
    return 1
  fi
}

build() {
  cd "$srcdir"

  # Build the kernel module
  make KERNELDIR="/usr/lib/modules/$(uname -r)/build"
}

package() {
  cd "$srcdir"

  # Install the kernel module
  install -D -m644 focal_spi.ko "${pkgdir}/usr/lib/modules/$(uname -r)/kernel/drivers/input/touchscreen/focal_spi.ko"

  # Install the custom libfprint library
  install -Dm755 "./usr/lib/x86_64-linux-gnu/libfprint-2.so.2.0.0" "${pkgdir}/usr/lib/libfprint-2.so.2.0.0"
  ln -sf libfprint-2.so.2.0.0 "${pkgdir}/usr/lib/libfprint-2.so.2"

  # Install udev rules if any
  if [ -d "./lib/udev/rules.d" ]; then
    install -d "${pkgdir}/usr/lib/udev/rules.d"
    cp -a "./lib/udev/rules.d/." "${pkgdir}/usr/lib/udev/rules.d/"
  fi

  # Install the custom fprintd.service
  install -Dm644 "$srcdir/fprintd.service" "${pkgdir}/usr/lib/systemd/system/fprintd.service"
}

post_install() {
  # Unregister fprintd.service from the fprintd package
  sudo pacman -Qql fprintd | grep 'fprintd.service' && sudo pacman -Rdd fprintd

  # Reload systemd and enable the new fprintd.service
  sudo systemctl daemon-reload
  sudo systemctl enable fprintd.service

  # Add libfprint and fprintd to IgnorePkg
  sudo bash -c 'grep -qxF "IgnorePkg = libfprint fprintd" /etc/pacman.conf || echo "IgnorePkg = libfprint fprintd" >> /etc/pacman.conf'
}

post_upgrade() {
  # Ensure fprintd.service remains unregistered from fprintd package
  sudo systemctl daemon-reload
  sudo systemctl enable fprintd.service

  # Ensure packages are ignored after upgrades
  sudo bash -c 'grep -qxF "IgnorePkg = libfprint fprintd" /etc/pacman.conf || echo "IgnorePkg = libfprint fprintd" >> /etc/pacman.conf'
}
