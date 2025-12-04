# SSH Service

## Service Name
- `ssh` or `sshd` (works on most distributions)

## Check Service Status
```bash
systemctl status ssh
systemctl status sshd  # Also works
```

## Configuration Location

Main directory: `/etc/ssh/`

Key files:
- `/etc/ssh/sshd_config` - Server configuration (most important)
- `/etc/ssh/ssh_config` - Client configuration
- `/etc/ssh/ssh_host_*_key` - Server private keys (multiple algorithms)
- `/etc/ssh/ssh_host_*_key.pub` - Server public keys

## Important sshd_config Options

```bash
Port 22                    # Default SSH port
ListenAddress 0.0.0.0     # Listen on all IPs (or specify one)
PermitRootLogin prohibit-password  # Or "yes" or "no"
```

### Port
Default is 22. Can change to non-standard port for security.

### ListenAddress
- `0.0.0.0` = listen on all IP addresses
- Or specify a single IP to restrict access

### PermitRootLogin
- `no` - root cannot SSH in at all
- `yes` - root can SSH in with password
- `prohibit-password` - root must use key authentication

## Connecting to SSH Server

Basic syntax:
```bash
ssh username@ip_address
ssh username@hostname.com
```

Example:
```bash
ssh sandbox@192.168.1.100
```

First connection prompts to accept server's fingerprint (say yes).

## Host Keys (Server-Side)

SSH server has multiple key pairs in `/etc/ssh/`:
- RSA keys: `ssh_host_rsa_key` and `ssh_host_rsa_key.pub`
- ECDSA keys: `ssh_host_ecdsa_key` and `ssh_host_ecdsa_key.pub`  
- ED25519 keys: `ssh_host_ed25519_key` and `ssh_host_ed25519_key.pub`

These are **asymmetric key pairs**:
- Private key stays on server (read-only to root)
- Public key shared with clients
- Data encrypted with one key only decrypts with the other

## Regenerating Host Keys

If keys are compromised (or cloned VMs have identical keys):

```bash
sudo ssh-keygen -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key
```

Options:
- `-t ecdsa` - key type (also: rsa, ed25519)
- `-f /path/to/key` - where to save
- Will prompt to overwrite existing key
- Can add passphrase or leave blank

## Client-Side Known Hosts

Location: `~/.ssh/known_hosts`

Contains public keys of servers you've connected to before.

If server key changes, you'll get a warning. To fix:
```bash
# Remove old entry for that IP
ssh-keygen -R 192.168.1.100

# Or delete the entire file and re-accept connections
rm ~/.ssh/known_hosts
```

## Passwordless Authentication

Allows login without password using key pairs.

**Setup process:**

1. Generate key pair on client (or server acting as admin):
```bash
ssh-keygen -t ecdsa -f ~/id_bob_key
```

2. Create `.ssh` directory for user:
```bash
sudo mkdir /home/bob/.ssh
sudo chmod 700 /home/bob/.ssh
sudo chown bob:bob /home/bob/.ssh
```

3. Copy public key to authorized_keys:
```bash
sudo cp id_bob_key.pub /home/bob/.ssh/authorized_keys
sudo chmod 644 /home/bob/.ssh/authorized_keys
sudo chown bob:bob /home/bob/.ssh/authorized_keys
```

4. Transfer private key to client using SCP:
```bash
scp sandbox@192.168.1.100:/path/to/id_bob_key .
```

5. Connect using the key:
```bash
ssh -i id_bob_key bob@192.168.1.100
```

**Critical permissions:**
- `.ssh/` directory: `700` (drwx------)
- `authorized_keys` file: `644` (-rw-r--r--)
- Private keys: `600` (-rw-------)
- Public keys: `644` (-rw-r--r--)

## SCP (Secure Copy)

Copy files over SSH:

```bash
# Copy from remote to local
scp user@remote:/path/to/file .

# Copy from local to remote  
scp localfile user@remote:/path/

# Use sudo on remote side
sudo scp user@remote:/root/file .
```

## Exit SSH Session

```bash
exit
```

## Restart After Config Changes

```bash
sudo systemctl restart ssh
```
