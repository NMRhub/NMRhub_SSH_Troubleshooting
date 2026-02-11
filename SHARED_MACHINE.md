# SSH Key Setup for Multi-User Shared Account

This guide covers setting up individual SSH keys for multiple users who share a single system account, with password-protected keys and convenient SSH config entries.

## Overview

When multiple people share one account on a machine, each person should have their own SSH key pair for security and accountability. This setup allows:
- Each user to have a password-protected private key
- Easy identification of who's connecting
- Simple login with SSH config shortcuts

## Part 1: Generate Individual SSH Keys

Each user should generate their own key pair on their local machine (not the shared server).

### Step 1: Create a key pair with a descriptive name

```bash
ssh-keygen -t ed25519 -C "your-email@example.com" -f ~/.ssh/id_ed25519_username_servername
```

**Options explained:**
- `-t ed25519` - Uses the modern Ed25519 algorithm (recommended)
- `-C "your-email@example.com"` - Adds a comment to identify the key
- `-f ~/.ssh/id_ed25519_username_servername` - Specifies the filename (replace `username` and `servername` with meaningful values)

**Alternative for older systems:**
If the server doesn't support Ed25519, use RSA instead:
```bash
ssh-keygen -t rsa -b 4096 -C "your-email@example.com" -f ~/.ssh/id_rsa_username_servername
```

### Step 2: Set a strong passphrase

When prompted, enter a strong passphrase to protect the private key:
```
Enter passphrase (empty for no passphrase): [type your passphrase]
Enter same passphrase again: [type your passphrase again]
```

**Important:** Never leave the passphrase empty! On a multi-user machine where others have access to this key, doing this would let other users log in as you.

### Step 3: Verify the keys were created

```bash
ls -la ~/.ssh/id_ed25519_username_servername*
```

You should see two files:
- `id_ed25519_username_servername` - Your private key (keep this secret!)
- `id_ed25519_username_servername.pub` - Your public key (safe to share)

## Part 2: Install Public Keys on the Server

### Step 1: Copy your public key to the server

View your key by running:
```bash
cat ~/.ssh/id_ed25519_username_servername.pub
```

Then add your individual user key to your NMRbox account at https://nmrbox.nmrhub.org/user-dashboard/ssh-key.

## Part 3: Configure SSH Client for Easy Access

On the shared user account, configure the local SSH client to know about the different keys available.

### Step 1: Edit your SSH config file

On your local machine:
```bash
nano ~/.ssh/config
```

### Step 2: Add a host entry

Add this configuration (adjust for your setup):
```
Host user1
    HostName fileaccess.nmrbox.org
    User user1
    IdentityFile ~/.ssh/id_ed25519_username_servername
    IdentitiesOnly yes
```

**Options explained:**
- `Host user1` - A short alias you'll use to connect
- `HostName` - The actual server address (domain or IP)
- `User` - The shared account username on the server
- `IdentityFile` - Path to your private key
- `IdentitiesOnly yes` - Only use the specified key (prevents trying other keys)

### Step 3: Set proper permissions

```bash
chmod 600 ~/.ssh/config
```

### Step 4: Test the connection

```bash
ssh user1
```

You'll be prompted for your key's passphrase (not the server password). After entering it once, the connection should establish.

Please note that in the SSH example directly above, you don't need to specify a hostname or key - just the alias for the connection (in this case user1) as defined in your `.ssh/config` file.

As you'll have one record for each user with a key on this system, each user can use their own alias to SSH into their own account using the proper key.

## Troubleshooting

### "Permission denied (publickey)"
- Check file permissions: `~/.ssh` should be 700, `authorized_keys` should be 600
- Ensure you're using the correct username and key file
- Try with verbose mode: `ssh -v myserver`

### "Bad permissions" errors
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/config
chmod 600 ~/.ssh/id_ed25519_*
chmod 644 ~/.ssh/*.pub
```

### Can't remember which key is which
```bash
# View public key fingerprint
ssh-keygen -lf ~/.ssh/id_ed25519_username_servername.pub

# List all keys in SSH agent
ssh-add -l
```
