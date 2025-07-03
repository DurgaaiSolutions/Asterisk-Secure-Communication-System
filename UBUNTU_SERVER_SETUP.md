# Ubuntu Server Initial Setup

This Script Installs The Essential Packages On A Fresh Ubuntu Server Installation.

## Installation Commands

```bash
# Update System
sudo apt update && sudo apt upgrade -y

# Enable universe, multiverse, and restricted repositories
sudo add-apt-repository universe
sudo add-apt-repository multiverse
sudo add-apt-repository restricted

# Update after enabling
sudo apt update

# Basic Networking And System Tools
sudo apt install -y net-tools curl wget htop iftop iotop lsof unzip zip tar git nano vim ufw tmux screen

# Build & Development Tools
sudo apt install -y build-essential software-properties-common make cmake autoconf automake libtool pkg-config

# Networking Utilities
sudo apt install -y iputils-ping traceroute dnsutils tcpdump nmap iproute2 openssh-server openssl

# Version Control And Compression
sudo apt install -y dkms preload build-essential linux-headers-$(uname -r) gdebi gnome-shell-extension-manager neofetch btop clang gcc cmake unrar p7zip-full ntfs-3g exfat-fuse fonts-firacode fonts-roboto fonts-noto default-jdk ubuntu-restricted-extras gufw synaptic bleachbit stacer

# Python Utilities
sudo apt install -y python3 python3-pip pipx sox alsa-utils ffmpeg

# System Monitoring And Logs
sudo apt install -y sysstat rsyslog logrotate

# Miscellaneous Helpful Tools
sudo apt install -y bash-completion debconf-utils lsb-release

# Enable SSH (If Not Running)
sudo systemctl enable ssh
sudo systemctl start ssh
