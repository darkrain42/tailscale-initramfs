# tailscale-initramfs

Run the [tailscale](https://tailscale.com) client in a Debian or Ubuntu
initramfs, to provide access to the Linux system prior to unlocking an encrypted
root filesystem. For instance, when combined with
[dropbear-initramfs](https://packages.debian.org/stable/dropbear-initramfs),
allows remote unlocking of an encrypted root filesystem from other systems in
the tailnet.

Intended to be used with a [tailscale ephemeral auth
key](https://tailscale.com/kb/1085/auth-keys/) to log into your tailnet.  Assign
an [ACL
tag](https://tailscale.com/kb/1068/acl-tags/#generate-an-auth-key-with-an-acl-tag
) to that auth key to lock down what access the pre-boot environment can have to
the rest of the tailnet.

## Install

0. Requires tailscale already be [installed](https://pkgs.tailscale.com/stable/)

1. Install tailscale-initramfs package

```bash
# Add the repository
sudo mkdir -p --mode=0755 /usr/share/local/keyrings
curl -fsSL https://darkrain42.github.io/tailscale-initramfs/keyring.asc | sudo tee /usr/local/share/keyrings/tailscale-initramfs-keyring.asc >/dev/null
echo 'deb [signed-by=/usr/local/share/keyrings/tailscale-initramfs-keyring.asc] https://darkrain42.github.io/tailscale-initramfs/repo stable main' | sudo tee /etc/apt/sources.list.d/tailscale-initramfs.list >/dev/null

# Install tailscale-initramfs
sudo apt-get update && sudo apt-get install tailscale-initramfs
```

2. Add authkey in /etc/tailscale/initramfs/config

3. Rebuild the initramfs

```bash
update-initramfs -c -k all
```

## Alternatives

* [initramfs-tools-tailscale](https://github.com/mabels/initramfs-tools-tailscale/)

  uses the tailscale state/data from the normal Linux install.  This means that
  the initramfs will show up as the existing device on the tailnet, but means
  the private key material is stored in the initramfs (which is commonly
  unencrypted).
