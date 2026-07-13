# Forge SSH Access Runbook

## Status

Secure SSH access from the MacBook is complete for story #12.

Related stories:

- [#11 Establish Forge's permanent network identity](https://github.com/stfolder/home-platform/issues/11)
- [#12 Establish secure SSH access from the MacBook](https://github.com/stfolder/home-platform/issues/12)
- [#13 Host firewall and baseline hardening](https://github.com/stfolder/home-platform/issues/13)

## Access Model

Forge is administered over SSH from approved home-LAN clients only. The canonical MacBook client path is:

```bash
ssh forge
```

The `forge` client alias resolves to `forge.home.arpa` and uses a dedicated per-device Ed25519 key. Password-based SSH authentication and root SSH login are disabled on Forge.

Do not commit private keys, passphrases, full `authorized_keys` contents, complete unrestricted SSH configuration exports, or credential material.

## MacBook Client Configuration

The MacBook has a dedicated Forge key:

| Item | Value |
|---|---|
| Key type | Ed25519 |
| Private key path | `~/.ssh/id_ed25519_forge` |
| Public key path | `~/.ssh/id_ed25519_forge.pub` |
| Public key fingerprint | Retained privately |
| Public key comment | `serge-macbook-to-forge-2026-07` |

Expected local permissions:

| Path | Expected mode |
|---|---|
| `~/.ssh` | `700` |
| `~/.ssh/id_ed25519_forge` | `600` |
| `~/.ssh/id_ed25519_forge.pub` | `644` |

Expected `~/.ssh/config` alias:

```sshconfig
Host forge
    HostName forge.home.arpa
    User serge
    IdentityFile ~/.ssh/id_ed25519_forge
    IdentitiesOnly yes
```

## Server Configuration

Forge manages SSH hardening through a drop-in under `/etc/ssh/sshd_config.d/`.

Current site-specific drop-in:

```text
/etc/ssh/sshd_config.d/90-home-platform-hardening.conf
```

Required effective server settings:

```text
permitrootlogin no
pubkeyauthentication yes
passwordauthentication no
kbdinteractiveauthentication no
```

Reference drop-in content:

```sshconfig
# Managed for home-platform story #12.
# Public-key SSH only; no root SSH login.
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
```

Before reloading SSH after any future change:

```bash
sudo sshd -t
```

Keep an existing working SSH session open until a new fresh session succeeds.

Reload SSH:

```bash
sudo systemctl reload sshd
```

## Validation

Run from the MacBook:

```bash
ssh forge hostname
```

Expected result:

```text
forge
```

Confirm password-only authentication is rejected:

```bash
ssh -o PubkeyAuthentication=no \
    -o PreferredAuthentications=password \
    -o NumberOfPasswordPrompts=0 \
    serge@forge.home.arpa hostname
```

Expected result:

```text
Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
```

Confirm root SSH login is rejected:

```bash
ssh -o BatchMode=yes \
    -o PreferredAuthentications=publickey \
    root@forge.home.arpa hostname
```

Expected result:

```text
Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
```

Run on Forge:

```bash
sudo sshd -T | grep -E '^(permitrootlogin|pubkeyauthentication|passwordauthentication|kbdinteractiveauthentication|usepam|authenticationmethods|authorizedkeysfile) '
sudo sshd -t
sudo systemctl status sshd --no-pager
```

Expected effective configuration:

```text
usepam yes
permitrootlogin no
pubkeyauthentication yes
passwordauthentication no
kbdinteractiveauthentication no
authorizedkeysfile .ssh/authorized_keys
authenticationmethods any
```

Story #12 validation evidence:

| Check | Result | Evidence / notes |
|---|---|---|
| Dedicated MacBook key exists | PASS | `~/.ssh/id_ed25519_forge`; exact fingerprint retained privately. |
| Private key permissions are restricted | PASS | `~/.ssh/id_ed25519_forge` mode is `600`. |
| Public key installed for `serge` on Forge | PASS | `authorized_keys` contains the dedicated public key fingerprint. |
| SSH alias uses dedicated identity | PASS | `ssh -G forge` resolves `identityfile ~/.ssh/id_ed25519_forge`. |
| SSH alias uses canonical hostname | PASS | `ssh -G forge` resolves `hostname forge.home.arpa`. |
| `ssh forge` succeeds | PASS | Fresh key-authenticated session returns `forge`. |
| SSH syntax is valid | PASS | `sudo sshd -t` completed successfully before reload. |
| Effective server config is hardened | PASS | `sshd -T` reports root login and password auth disabled. |
| Password-only authentication fails | PASS | Password-only attempt returns permission denied with public-key methods only. |
| Root SSH login fails | PASS | Root login attempt returns permission denied. |
| Access survives SSH reload | PASS | Fresh `ssh forge` succeeded after reload. |
| Access survives Forge reboot | PASS | Fresh `ssh forge` succeeded after reboot; `sshd` remained active and enabled. |

## Adding a New Device

Create a dedicated key on the new device:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_forge -C "device-name-to-forge-YYYY-MM"
```

Copy only the public key from the new device:

```bash
cat ~/.ssh/id_ed25519_forge.pub
```

From an already-authorized admin device, append the public key as one line to `/home/serge/.ssh/authorized_keys` on Forge:

```bash
ssh forge
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

Do not temporarily re-enable password-based SSH only to enroll a new device. Use an already-authorized SSH session, Cockpit if authenticated, or the local console.

## Key Revocation

To revoke one device without deleting unrelated keys:

1. Identify the public-key fingerprint to revoke.

```bash
ssh-keygen -lf ~/.ssh/authorized_keys
```

2. Back up the current authorized keys file on Forge.

```bash
cp ~/.ssh/authorized_keys ~/.ssh/authorized_keys.bak.$(date +%Y%m%d%H%M%S)
```

3. Edit `~/.ssh/authorized_keys` and remove only the single line matching the revoked device key.

```bash
nano ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

4. Keep one current session open and confirm a fresh session from a still-approved device works.

## Troubleshooting

If `ssh forge` cannot resolve the host:

```bash
dig @10.42.42.1 forge.home.arpa
dscacheutil -q host -a name forge.home.arpa
```

The MacBook should use the home router as the LAN DNS server for `home.arpa`.

If key authentication fails:

```bash
ssh -vvv forge
ssh-keygen -lf ~/.ssh/id_ed25519_forge.pub
```

On Forge, compare the expected public-key fingerprint with:

```bash
ssh-keygen -lf ~/.ssh/authorized_keys
```

If SSH configuration changes fail validation, do not reload `sshd`. Correct the configuration and rerun:

```bash
sudo sshd -t
```

## Local Console Recovery

If SSH access is accidentally lost:

1. Log in through the local Forge console or authenticated Cockpit web console.
2. Inspect the hardening drop-in:

```bash
sudo ls -la /etc/ssh/sshd_config.d
sudoedit /etc/ssh/sshd_config.d/90-home-platform-hardening.conf
```

3. Validate syntax:

```bash
sudo sshd -t
```

4. Reload or restart SSH after syntax passes:

```bash
sudo systemctl reload sshd
```

5. From the MacBook, validate:

```bash
ssh forge hostname
```

If needed, temporarily move the site-specific drop-in aside from the local console, validate syntax, reload SSH, and restore a compliant configuration before considering the story complete.

## Network Boundary

Story #12 does not introduce public SSH exposure. No router port forward, dynamic DNS, Internet-facing SSH, SSH port change, VPN, fail2ban, or broad firewall hardening is included here. Broader host firewall and hardening work belongs to story #13.
