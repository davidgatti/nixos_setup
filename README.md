# NixOS Setup Guide

## 1. Initial Setup on the Target PC

1. **Insert the NixOS USB into the PC**:
   - Make sure the USB is inserted properly before booting.

1. **Boot from USB**:
   - Go to the BIOS and ensure the PC is set to boot from the USB drive.

1. **Boot into NixOS**:
   - Once prompted, select the option to boot into NixOS from the USB.

1. **Set a temporary password**:
   - Once NixOS is running, set a password to enable remote access via SSH. Run the following command in the terminal:
     ```bash
     passwd
     ```
   - Follow the prompts to set a password. This password will be used later for SSH access.

1. **Find the PC’s IP address**:
   - Run the following command to find the PC’s IP address:
     ```bash
     ip addr show
     ```
   - Note down the IP address, which you’ll use to connect via SSH.

1. **Connect remotely via SSH**:
   - From another computer on the same network, connect to the PC using the following command (replace `IP` with the actual IP address of the PC):
     ```bash
     ssh nixos@IP
     ```
   - Use the password you just set when prompted.

## 2. OS Installation

Copy and paste the following in the terminal to start the installation process.

```bash
sudo bash -c '
(
echo g                        # Create a new GPT partition table
echo n                        # New partition (EFI)
echo 1                        # Partition number 1
echo                          # Default - start at beginning of disk
echo +512M                    # 512 MB EFI partition
echo t                        # Change type
echo 1                        # Set type to EFI System
echo n                        # New partition (Root)
echo 2                        # Partition number 2
echo                          # Default - start immediately after previous partition
echo                          # Use remaining space
echo w                        # Write changes
) | fdisk /dev/sda            # Run fdisk on /dev/sda

# Format the partitions
mkfs.fat -F 32 /dev/sda1        # Force format EFI partition as FAT32
mkfs.ext4 -F /dev/sda2          # Force format root partition as ext4

# Mount the partitions
mount /dev/sda2 /mnt            # Mount root partition
mkdir -p /mnt/boot              # Create boot directory
mount /dev/sda1 /mnt/boot       # Mount EFI partition

# Set secure permissions on /boot
chmod 700 /mnt/boot             # Restrict access to /boot

# Generate NixOS configuration
nixos-generate-config --root /mnt

# Insert SSH and firewall settings into configuration.nix
sed -i "/^}$/i \
  services.openssh = {\n\
    enable = true;\n\
    settings.PermitRootLogin = \"no\";\n\
    settings.PasswordAuthentication = true;\n\
  };\n\
  networking.firewall.allowedTCPPorts = [ 22 ];\n\
  users.users.nixos = {\n\
    isNormalUser = true;\n\
    extraGroups = [ \"wheel\" ];\n\
    password = \"password\"; # Set a default password (replace as needed)\n\
  };" /mnt/etc/nixos/configuration.nix

# Install NixOS
nixos-install

# Set the root password non-interactively
echo "root:root" | chroot /mnt chpasswd

# Unmount and reboot
umount -R /mnt
reboot
'
```

## 3. Reconnect After Installation

1. **Remove `known_hosts` (if necessary)**:
1. **Reconnect via SSH**:
   - Connect to the newly installed NixOS system:
     ```bash
     ssh nixos@IP
     ```
   - Use the password you set during the installation process.

## 4. NixOS Configuration

Once connected via SSH, proceed with the NixOS configuration. Copy and paste the following command to start the configuration process:

### One-liner Command:
```bash
sudo bash -c ': > /etc/nixos/configuration.nix && \
curl -L https://raw.githubusercontent.com/davidgatti/nixos_setup/main/configuration.nix -o /etc/nixos/configuration.nix && \
nixos-rebuild switch && \
cat ~/.config/code-server/config.yaml'
```

### Explanation:

1. **Clear the NixOS configuration file**:
   - Empties the current `/etc/nixos/configuration.nix` file to start with a fresh configuration.
   
2. **Download the new configuration**:
   - Fetches the latest `configuration.nix` file from GitHub and saves it to `/etc/nixos/configuration.nix`.

3. **Apply the NixOS configuration**:
   - Rebuilds and applies the NixOS configuration immediately.

4. **View the Code Server configuration**:
   - Displays the contents of the VS Code Server configuration file for verification.


sudo nixos-rebuild switch -I nixos-config=./configuration.nix
