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
