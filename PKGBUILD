# Maintainer: Your Name <your.email@example.com>

pkgname=focaltech-fingerprint-lts
pkgver=1.0
pkgrel=1
pkgdesc="Focaltech fingerprint SPI driver and custom libfprint for linux-lts kernel"
arch=('x86_64')
url="https://github.com/ftfpteams/ubuntu_spi"
license=('GPL' 'LGPL')
depends=('linux-lts')
makedepends=('linux-lts-headers')
conflicts=('libfprint')
provides=('libfprint' 'focaltech-fingerprint-lts')
source=(
  "https://raw.githubusercontent.com/ftfpteams/ubuntu_spi/main/focal_spi.c"
  "Makefile"
  "libfprint.deb::https://github.com/ftfpteams/ubuntu_spi/raw/main/libfprint-2-2_1.94.4+tod1-0ubuntu1~22.04.2_spi_amd64_20240620.deb"
)
md5sums=('SKIP' 'SKIP' 'SKIP')

prepare() {
  cd "$srcdir"

  # Extract libfprint files from the deb package
  bsdtar -xf libfprint.deb
  bsdtar -xf data.tar.xz

  # Modify the focal_spi.c if necessary (uncomment if needed)
  # sed -i 's/spi->mode = SPI_MODE_0;/spi->mode = SPI_MODE_0|SPI_CS_HIGH;/' focal_spi.c
}

build() {
  cd "$srcdir"

  # Build the kernel module
  KERNELDIR="/usr/lib/modules/$(uname -r)/build"
  make KERNELDIR=$KERNELDIR
}

package() {
  cd "$srcdir"

  # Install the kernel module
  install -D -m644 focal_spi.ko "${pkgdir}/usr/lib/modules/$(uname -r)/kernel/drivers/input/touchscreen/focal_spi.ko"

  # Install the custom libfprint library
  install -Dm755 "./usr/lib/x86_64-linux-gnu/libfprint.so.2.0.0" "${pkgdir}/usr/lib/libfprint.so.2.0.0"
  ln -sf libfprint.so.2.0.0 "${pkgdir}/usr/lib/libfprint.so.2"

  # Install udev rules if any
  if [ -d "./lib/udev/rules.d" ]; then
    install -d "${pkgdir}/usr/lib/udev/rules.d"
    cp -a "./lib/udev/rules.d/." "${pkgdir}/usr/lib/udev/rules.d/"
  fi

  # Clean up extracted files
  rm -rf usr lib etc
}

post_install() {
  depmod -a "$(uname -r)"

  # Ensure the module loads on boot
  echo "focal_spi" | sudo tee /etc/modules-load.d/focal_spi.conf >/dev/null
}

post_upgrade() {
  depmod -a "$(uname -r)"
}

post_remove() {
  depmod -a "$(uname -r)"
  sudo rm -f /etc/modules-load.d/focal_spi.conf
}
