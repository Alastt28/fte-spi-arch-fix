# How to install Focaltech fingerprint SPI driver on Arch Linux

Simply install dependencies, clone the repo and build:

```bash
sudo pacman -S git base-devel
git clone https://github.com/Alastt28/fte-spi-arch-fix.git
cd fte-spi-arch-fix
makepkg -si
```

Then finally reboot your computer and enjoy!

Credit:
ftfpteams (https://github.com/ftfpteams/ubuntu_spi)
