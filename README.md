# SSH Troubleshooting
[NMRhub](https://nmrhub.org) now requires an ssh keypair to connect use [ssh / sftp / rsync](https://openssh.com) and tools like [FileZilla](https://filezilla-project.org) to connect to NMRhub virtual machines (VMs).

## Where is my private key?
After following the instructions at https://nmrbox.nmrhub.org/user-dashboard/ssh-key, your
private key should be at $HOME/.ssh/id_ed25519 on Mac/Linux or "%HOMEDRIVE%%HOMEPATH%\.ssh\id_ed25519" on Windows.

## I have the private key in place but I can't connect.
Try on more than one VM to make sure it's not just that VM. (Please let us know if you can SSH to one VM but not another via support@nmrbox.org).

Please try *ssh -vvv* and verify ssh is trying to user your private key. 

### It's trying to use the key but it still doesn't work
If you can't connect after uploading your SSH key, please verify the following commands give the same result. Windows users should run them from PowerShell.

ssh-keygen -lf $HOME/.ssh/id_ed25519

ssh-keygen -lf $HOME/.ssh/id_ed25519.pub

Please double check the key listed at https://nmrbox.nmrhub.org/user-dashboard/ssh-key is the same as your $HOME/.ssh/id_ed25519.pub .
![Date key set](sample.jpeg)

## Home directory permissions ##
If you home directory is writable by anyone other than you SSH will not work. Connect via [VNC](https://nmrbox.nmrhub.org/pages/getting-started) and *ls -l \~* .
If **w** appears after the first group of permissions, drwx, use *chmod go-w \~* to correct.

## How do I use a private key with *rsync*?
Add _-e "ssh -i /path/to/your/private_key"_ to your rsync command.

## How do I use a private key with *sftp*?
Add _-i /path/to/your/private_key_  to your sftp command.

## How do I use private key with FileZilla?

Go to FileZillas Site Manager and make the following settings in the General tab. You can leave the port blank, it defaults to the correct value (22).

**Protocol:**      SFTP - SSH File Transfer Protocol

**Host:**          *element*.nmrbox.org             **Port:** ______

**Logon Type:**    Key file

**User:**          *username* 

**Key file:**      *local path*/.ssh/id_ed25519            

## How do I use private key with [WINScp](https://winscp.net)?
See [The Authentication Page](https://winscp.net/eng/docs/ui_login_authentication).

## How do I scp from a shared account?
(Note: This is not an endorsement of shared accounts but that is a site specific decision.)

If multiple users are using a shared account (e.g. from a spectrometer console), the following steps will allow copying local data to each user's account.
For the example we assume your username is *rernst.*

Create another ssh keypair **with a passphrase:**

```
ssh-keygen -t ed25519 -C "spectrometer scp"
Generating public/private ed25519 key pair.
Enter file in which to save the key (/Users/rernst/.ssh/id_ed25519): id_ed25519rernst
Enter passphrase (empty for no passphrase):  <type a password>
Enter same passphrase again: <retype the password> 
Your identification has been saved in id_ed25519rernst
Your public key has been saved in id_ed25519rernst.pub
```

*Add* the contents of id_ed25519rernst.pub to https://nmrbox.nmrhub.org/user-dashboard/ssh-key .

Go to the home directory of the shared account. Create an .ssh directory if necessary and ensure it has permssions *drwx------* 
```cd $HOME
mkdir -p .ssh
chmod 0o700 .ssh
```

Create or edit **~/.ssh./config**. Add the following lines.

```
Match User rernst Host *.nmrbox.org
        IdentityFile ~/.ssh/id_ed25519rernst
```

You should now be able to do the following to scp data from the shared account to nmrbox.org.

> scp fid rernst@*element*.nmrbox.org:

This will prompt:

> Enter passphrase for key '*home directory*/.ssh/id_ed25519rernst'

at which point your enter the password you used when creating a keypair. 



