# tailscale-initramfs

Run the [tailscale](https://tailscale.com) client in a Debian or Ubuntu
initramfs, to provide access to the Linux system prior to unlocking an
encrypted root filesystem. When combined with [tailscale
ssh](https://tailscale.com/kb/1193/tailscale-ssh) or
[dropbear-initramfs](https://packages.debian.org/stable/dropbear-initramfs),
allows remote unlocking of an encrypted root filesystem from other systems in
the tailnet.

Intended to be used with a [tailscale ephemeral auth
key](https://tailscale.com/kb/1085/auth-keys/) to log into your tailnet.  Assign
an [ACL
tag](https://tailscale.com/kb/1068/acl-tags/#generate-an-auth-key-with-an-acl-tag
) to that auth key to lock down what access the pre-boot environment can have to
the rest of the tailnet, i.e. to disallow all outbound access from the
initramfs, and only permit inbound connections.

**Caution**: The initramfs tailscale instance cannot log in your tailnet after
the auth key expires, so the pre-boot environment won't be accessible from
your tailnet.  Tailscale auth keys are only valid for up to 90 days,
configured when generating the auth key.  To avoid losing access to the
pre-boot environment, make sure you create a new auth key and update the
initramfs before the old key expires.

## Install

0. Requires tailscale already be [installed](https://pkgs.tailscale.com/stable/)

1. Install tailscale-initramfs package

```bash
# Add the repository
sudo mkdir -p --mode=0755 /usr/local/share/keyrings
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

## Security

If you're using full disk encryption (i.e. LUKS), the initramfs is typically
unencrypted alongside the boot loader.  The auth key you configure for this
package is included in that initramfs in plaintext.  Someone who can access
the initramfs file can get a copy of the auth key and register new devices
into the tailnet as long as the key is valid.

You can restrict what access such devices have by configuring [Tailscale
ACLs](https://login.tailscale.com/admin/acls/) that prevent these devices from
making any connections, and only accepting incoming connections.

### Example ACLs
```json
{
	"tagOwners": {
		"tag:initramfs": [],
	},
	"acls": [
		{"action": "accept", "src": ["autogroup:member"], "dst": ["*:*"]},
	],
	"tests": [
		// initramfs tag cannot make any outbound connections
		{
			"src":  "tag:initramfs",
			"deny": ["100.101.102.103:22", "192.0.2.1:22", "[2001:db8::feed]:22"],
		},
		// but users can connect to the initramfs
		{
			"src":   "user@example.com",
			"allow": ["tag:initramfs:22"],
		},
	],
}
```

## Alternatives

* [initramfs-tools-tailscale](https://github.com/mabels/initramfs-tools-tailscale/)

  uses the tailscale state/data from the normal Linux install.  This means that
  the initramfs will show up as the existing device on the tailnet, but means
  the private key material is stored in the initramfs (which is commonly
  unencrypted).

* [tailscale-initramfs by Lugoues](https://github.com/Lugoues/tailscale-initramfs)

  Similar to this package, but registers the initrd as a tailscale device when
  you configure the package.  The initrd device will be present in the tailnet
  all the time.
