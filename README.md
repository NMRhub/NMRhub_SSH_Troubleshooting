
# SSH Troubleshooting

[NMRhub](https://nmrhub.org) now requires an SSH key pair to connect. Use [SSH / SFTP / rsync](https://openssh.com) and tools like [FileZilla](https://filezilla-project.org) to connect to NMRhub virtual machines (VMs).

## Where is my private key?

After following the instructions at [NMRbox User Dashboard SSH Key Setup](https://nmrbox.nmrhub.org/user-dashboard/ssh-key), your private key should be at:
- On Mac/Linux: `$HOME/.ssh/id_ed25519`
- On Windows: `%HOMEDRIVE%%HOMEPATH%\.ssh\id_ed25519`

## I have the private key in place, but I can't connect.

1. Try connecting to more than one VM to make sure it's not just an issue with that specific VM. (Please let us know if you can SSH to one VM but not another via [support@nmrbox.org](mailto:support@nmrbox.org)).
2. Use `ssh -vvv` to verify if SSH is attempting to use your private key.

### It's trying to use the key, but it still doesn't work.

If you can't connect after uploading your SSH key, verify the following commands give the same result. Windows users should run these commands from PowerShell:

```bash
ssh-keygen -lf $HOME/.ssh/id_ed25519
ssh-keygen -lf $HOME/.ssh/id_ed25519.pub
```

Ensure the key listed at [NMRbox User Dashboard SSH Key](https://nmrbox.nmrhub.org/user-dashboard/ssh-key) matches your `$HOME/.ssh/id_ed25519.pub`.

![Date key set](sample.jpeg)

## Home Directory Permissions

If your home directory is writable by anyone other than you, SSH will not work. Connect via [VNC](https://nmrbox.nmrhub.org/pages/getting-started) and run:

```bash
ls -l ~
```

If **`w`** appears after the first group of permissions (e.g., `drwx`), use the following command to correct it:

```bash
chmod go-w ~
```

## How do I use a private key with `rsync`?

Add the following option to your `rsync` command:

```bash
-e "ssh -i /path/to/your/private_key"
```

## How do I use a private key with `sftp`?

Add the following option to your `sftp` command:

```bash
-i /path/to/your/private_key
```

## How do I use a private key with FileZilla?

Go to FileZilla's **Site Manager** and make the following settings in the **General** tab. You can leave the port blank, as it defaults to the correct value (22).

- **Protocol:** SFTP - SSH File Transfer Protocol
- **Host:** *element*.nmrbox.org
- **Port:** [Leave blank]
- **Logon Type:** Key file
- **User:** *username*
- **Key file:** *local path*/.ssh/id_ed25519

## How do I use a private key with [WinSCP](https://winscp.net)?

See [The Authentication Page](https://winscp.net/eng/docs/ui_login_authentication).

## How do I `scp` from a shared account?

(Note: This is not an endorsement of shared accounts, but that is a site-specific decision.)

If multiple users are using a shared account (e.g., from a spectrometer console), the following steps will allow copying local data to each user's account. For this example, we assume your username is `rernst`.

1. **Create another SSH key pair with a passphrase:**

   ```bash
   ssh-keygen -t ed25519 -C "spectrometer scp"
   Generating public/private ed25519 key pair.
   Enter file in which to save the key (/Users/rernst/.ssh/id_ed25519): id_ed25519rernst
   Enter passphrase (empty for no passphrase):  <type a password>
   Enter same passphrase again: <retype the password>
   Your identification has been saved in id_ed25519rernst
   Your public key has been saved in id_ed25519rernst.pub
   ```

2. **Add** the contents of `id_ed25519rernst.pub` to [NMRbox User Dashboard SSH Key](https://nmrbox.nmrhub.org/user-dashboard/ssh-key).

3. Go to the home directory of the shared account. Create an `.ssh` directory if necessary and ensure it has permissions `drwx------`:

   ```bash
   cd $HOME
   mkdir -p .ssh
   chmod 700 .ssh
   ```

4. Create or edit `~/.ssh/config` and add the following lines:

   ```bash
   Match User rernst Host *.nmrbox.org
       IdentityFile ~/.ssh/id_ed25519rernst
   ```

5. You should now be able to copy data from the shared account to `nmrbox.org`:

   ```bash
   scp fid rernst@*element*.nmrbox.org:
   ```

   When prompted:

   ```text
   Enter passphrase for key 'home_directory/.ssh/id_ed25519rernst':
   ```

   Enter the password you used when creating the key pair.
