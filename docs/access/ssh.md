# Shell access through SSH
SSH (Secure Shell) is a widely used protocol for securely gaining access to a shell (or terminal) on remote Linux machines and is the primary way to access the BioCloud servers. This page provides instructions on how to access the BioCloud through SSH using a few different SSH clients and platforms. There are many other SSH clients available, it is entirely up to you which you prefer, but regardless of the client everything will run over the same SSH protocol (port 22) in the exact same way. You authenticate using your AAU email and password (possibly also a second factor) and you must be on the AAU network unless you are connected to the AAU network from the outside using either VPN or by using the AAU SSH gateway, both are described later under [external access](#external-access).

If you need to run GUI apps like CLC, Arb, RStudio, VS Code, etc, you can instead use the [interactive web portal](webportal.md) described on the next page. For everything else, you need to learn how to submit SLURM jobs through one of the login-nodes `bio-ospikachu01.srv.aau.dk`, `bio-ospikachu02.srv.aau.dk`, or `bio-ospikachu03.srv.aau.dk`. After successfully logging in please consult the [SLURM guide](../slurm/intro.md). You can then start developing code, write scripts, install software and browse around etc, but you are **NOT ALLOWED** to run anything on a SLURM login node, everything must be submitted through SLURM jobs.

## Code editors (IDEs)
It's rarely enough with just a terminal because you more often than not need to edit some scripts in order to do anything, which is not very convenient to do in a terminal, so below are some instructions on how to connect using a few popular code editors, or IDEs (integrated development environments), with built-in SSH support, but also [just a terminal](#just-a-terminal).

### Visual Studio Code
[Visual Studio Code](https://code.visualstudio.com/) (VS Code) is a popular free cross-platform code editor with a myriad of [extensions](https://marketplace.visualstudio.com/VSCode) available for anything and everything including syntax highlighting for any programming language, integrated git support, [GitHub copilot](https://github.com/features/copilot) for AI autocompletion of code, and the list goes on. If you want one editor for everything, there isn't currently anything better out there. If you need an interactive VS Code session and actually run things from there, please follow [this guide](../guides/sshdslurm.md) to connect VS Code directly to a SLURM job instead. Alternatively, you can also start a Code Server (an open source alternative to VS Code) in a SLURM job from the [interactive web portal](../access/webportal.md) described on the next page.

#### Installation (Windows, macOS, or Linux)
Download and install using the instructions on the official [website](https://code.visualstudio.com/).

#### Connecting to a login-node
1. Open VS Code and install the "Remote - SSH" extension from the Extensions sidebar menu.
2. Click on the "Remote Explorer" icon after the extension has been installed.
3. Add a host by either:
    - clicking on the "+" icon and enter your AAU email followed by `@` and then the server's hostname, for example: `abc@bio.aau.dk@bio-ospikachu01.srv.aau.dk`, or
    - add all servers at once using the [SSH config template](#ssh-config-file) provided below by clicking the gear icon "Open SSH Config File" and paste its contents (optional).
![Edit SSH config](../img/sshconfigvscode.png)
4. Connect and log in with your SSH password.
5. Once connected open a project or workspace folder (or create one while doing so) by clicking File -> Open Folder (CTRL+k CTRL+o, CMD instead of CTRL if on macOS) to start your work

### MobaXterm
A more old-school and Windows-only client is MobaXterm. It's not as full-featured as VS Code, but is more lightweight.

#### Installation (Windows only)
Download and install MobaXterm Home Edition from the official [website](https://mobaxterm.mobatek.net/download.html).

#### Connecting to a server
1. Open MobaXterm and click on the "Start local terminal" button.
2. In the terminal, use the `ssh` command to connect to a server by entering AAU email followed by `@` and then the server's hostname, for example: `ssh abc@bio.aau.dk@bio-ospikachu01.srv.aau.dk`.

If you add the servers to your [SSH config file](#ssh-config-file) the server hostnames should auto-complete.

### Just a terminal
For a simple terminal you can on Windows use for example [PuTTY](https://www.putty.org/) or [msys2](https://www.msys2.org/), and the integrated terminal and the `ssh` command itself directly if on Linux or macOS (also Windows). Type `ssh email@hostname` like above to connect immediately or just the hostname if you've added them to the [SSH config file](#ssh-config-file).

## Transferring files
Both VS Code and MobaXterm support file transfers, but you can also use other GUI apps like [FileZilla](https://filezilla-project.org/download.php) or [WinSCP](https://winscp.net/eng/index.php). When just using a terminal there are several tools like `scp`, `rsync`, `rclone`, or `sftp`, all of which connect through the SSH protocol. You can also browse and transfer files through the [interactive web portal](webportal.md#transferring-files) described on the next page.

## External access
To access the servers while not directly connected to any AAU or [eduroam](https://eduroam.dk/) network there are two options. Either you connect through VPN, which will route all your traffic through AAU, or you can use the SSH gateway through `sshgw.aau.dk` for SSH connections only. If you need virtual desktop access only VPN will work (for now).

### VPN
The simplest way is to connect to the AAU VPN using the guide provided by ITS [here](https://www.its.aau.dk/vejledninger/vpn). After you connect everything should work as though you were at AAU.

### Using an SSH jump host
The SSH gateway is simply a server hosted at AAU whose only purpose is to bridge SSH connections from the outside (open port 22 ingress) to something else on the internal AAU network. Setting up [2-factor authentication](https://www.its.aau.dk/vejledninger/mfa) is required in order to connect. To enable the "proxy jumping" you need to adjust the [SSH config file](#ssh-config-file) by uncommenting the `ProxyJump sshgw.aau.dk` line under the `*.srv.aau.dk` host, see the [SSH config template file](#ssh-config-file-template). Keep in mind that as long as it's enabled it will always connect through `sshgw.aau.dk` regardless of which network you are connected to, including the AAU network.

## Additional configuration (optional)

### SSH config file
To avoid typing hostnames and user names constantly here's a template SSH config file that includes all current servers. The file must be saved at certain locations depending on your OS. On Windows it's [here](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_server_configuration#openssh-configuration-files), and on macOS and Linux it's usually under `~/.ssh/config`. Hostnames in the file will then be auto-completed and if you've set up [SSH public key authentication](#ssh-public-key-authentication) you won't need to type anything else than for example `ssh bio-oscloud02.srv.aau.dk` and you're in.

#### SSH config file template
```plaintext
# nice-to-have global options that apply to all servers,
# prevents disconnects on network glitches and wifi reconnects,
# allows forwarding GUI apps to client
Host *
  ServerAliveInterval 60
  ServerAliveCountMax 2
  ExitOnForwardFailure yes
  ForwardX11 yes
  ForwardX11Trusted yes

# this is only used for external access
Host sshgw.aau.dk
  HostName sshgw.aau.dk
  User abc@bio.aau.dk
  Port 22
  ForwardAgent yes

# use the same user name (and optional SSH key) for all bioservers
# uncomment the ProxyJump line for external access without VPN
Host *.srv.aau.dk bio-os*
  User abc@bio.aau.dk
  Port 22
  IdentityFile ~/.ssh/keys/bioservers
#  ProxyJump sshgw.aau.dk

Host bio-ospikachu01.srv.aau.dk
Host bio-ospikachu02.srv.aau.dk
Host bio-ospikachu03.srv.aau.dk
```

### SSH Public Key Authentication
[SSH public key authentication](https://www.ssh.com/academy/ssh/public-key-authentication) offers a more secure way to connect to a server, and is also more convenient, since you don't have to type in your password every single time you log in or transfer a file. An SSH private key is essentially just a very long password that is used to authenticate with a server holding the cryptographically linked public key for your user (think of it as the lock for the private key). Any SSH client that you choose to use will connect through the SSH program on your computer under the hood, so this will also apply to all of them.

#### Generating SSH Key Pairs
This must be done locally for security reasons, so that the private key never leaves your computer. If you use a password manager (please do) like 1Password or bitwarden you can usually both generate and safely store and use SSH keys directly from the vault without it lying around in a file. It's important that the key is not generated using the default (usually) RSA type algorithm, because it's outdated and can be brute-forced easily with modern hardware, so please use the for example the `ed25519` algorithm instead.

##### On Linux or macOS:
1. Open your terminal.
2. Generate an SSH key pair with the following command: `ssh-keygen -t ed25519`.
3. Follow the prompts, and save the two keyfiles somewhere (usually in the hidden folder `~/.ssh/biocloud`).

##### On Windows:
Use Git Bash, which includes SSH keygen, or install a tool like PuTTYgen.

#### Adding Public Keys to the Server
Copy your public key to the server using `ssh-copy-id -i ~/.ssh/biocloud.pub username@hostname`, or manually add the contents of the public key file into the `~/.ssh/authorized_keys` file on any of the servers (the home folder is shared across them all). SSH requires that your user is the only one able to read the contents before you are allowed to login, so ensure the correct permissions are set using `chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys`.

You should now be able to login securely without typing a password - have fun!