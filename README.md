# WSL2-Mac-Utilize-CUDA
Guide for setting up WSL2 on Windows and utilizing CUDA with a Mac. Installing WSL2 on Windows and Using Mac to Utilize CUDA

## Why Do This?
The author uses a Mac but wants to use an Nvidia GPU installed on a Windows machine as computational power for models. Additionally, the author prefers to operate in a Linux system due to its simple interface and abundant commands.

## Use Case
Within the same network, if you have a Mac but its computational power is insufficient, you can use the Nvidia GPU on your home Windows computer as computational power. If you find Windows command operations limited and not intuitive enough, you can use Linux as the operating interface. After reading this article, you will gain valuable insights.

![Usage Scenario](https://hackmd.io/_uploads/HJGZRCnY0.jpg)

## What You Will Gain
- Basic concepts of WSL
- Basic concepts of SSH
- A Linux subsystem on a Windows system

## Setting Up the Linux Subsystem on the Windows Server

### Ensure Virtualization is Enabled and the Windows Subsystem for Linux is Enabled
1. In Task Manager, ensure CPU virtualization is enabled.
2. Open PowerShell and run the following command:
    ```shell
    dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
    ```
3. In the search bar, type "Turn Windows features on or off", select "Windows Subsystem for Linux" and "Virtual Machine Platform", click OK, and restart after installation.

## Downloading WSL

### What is WSL?
According to Microsoft's official explanation, Windows Subsystem for Linux (WSL) is a feature of Windows that allows you to run a Linux environment on a Windows computer without a separate virtual machine or dual-boot setup. The recommended version is WSL2.

### Downloading Ubuntu from Microsoft Store
This guide uses Ubuntu 22.04.

![Downloading Ubuntu](https://hackmd.io/_uploads/BJvWX0nKR.png)

#### Common Issue
If, after opening and pressing a random key, there is no response, check the WSL version in the terminal to confirm it is version 2.

```shell
wsl -l -v
```
If it is not version 2, then:
1. Upgrade WSL:
    ```shell
    wsl --update
    ```
2. Set the default WSL version to WSL2:
    ```shell
    wsl --set-default-version 2
    ```
3. Use WSL2 to manage the specified subsystem and ensure your Ubuntu version:
    ```shell
    wsl --set-version Ubuntu-22.04 2
    ```

Reopen Ubuntu from the store. If successful, the Ubuntu installation location and folder can be directly found under "This PC -> Linux".

## SSH Configuration

### Connection Behavior
MAC --> Windows (Windows) --> Windows (Linux)

### SSH Configuration on Windows
1. Install OpenSSH server:
    ```shell
    Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
    ```
2. Enable and start the OpenSSH server:
    ```shell
    Start-Service sshd
    Set-Service -Name sshd -StartupType 'Automatic'
    ```
3. Allow SSH traffic through the firewall:
    ```shell
    New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
    ```
4. Check the SSH server status:
    ```shell
    Get-Service sshd
    ```

### SSH Configuration on WSL (Open Ubuntu Terminal)
1. Install OpenSSH:
    ```shell
    sudo apt update
    sudo apt install openssh-server
    ```
2. Start and configure the OpenSSH server:
    ```shell
    sudo service ssh start
    sudo systemctl enable ssh
    ```
3. Configure SSH settings by editing the /etc/ssh/sshd_config file:
    ```shell
    sudo nano /etc/ssh/sshd_config
    ```
    Ensure the following lines are uncommented and correctly set:
    ```
    Port 22
    ListenAddress 0.0.0.0
    PermitRootLogin yes
    PasswordAuthentication yes
    ```
4. Set WSL to open ports for Windows:
    Edit or create the WSL startup configuration file (e.g., ~/.bashrc) and add the following content:
    ```shell
    sudo /usr/sbin/sshd -D &
    ```
    Restart WSL or reload the configuration:
    ```shell
    source ~/.bashrc
    ```

### Setting Windows (Windows) to Windows (Linux)
Otherwise, the Mac SSH connection will default to the Windows directory.
```shell
netsh interface portproxy add v4tov4 listenaddress=[Windows_IP] listenport=[PORT] connectaddress=[WSL_IP] connectport=[PORT]
```
- Set the port to 22.
- Check Windows_IP using `ipconfig` in the Windows terminal.
- Check WSL_IP using `ip addr show eth0` in the WSL terminal.

## Mac SSH Connection
In the Mac terminal, enter:
```shell
ssh username@Windows_IP -p 22
```
Or set up an SSH connection in VS Code:
```shell
Host Labs               # Custom name
  HostName Windows_IP   # Host name or IP address
  User jenhaoyang       # Username for login
  Port 22               # If a specific port number is specified
```
If successful, you will directly enter the Linux subsystem on Windows.

![Successful Connection](https://hackmd.io/_uploads/HJIuTR3tC.png)

## Future Topics
- Setting up Conda environment
- Installing Nvidia drivers and CUDA on Linux
- Common issues with downloading PyTorch

### References
- [Microsoft Official Documentation](https://learn.microsoft.com/en-us/windows/wsl/about)
- [CSDN Article 1](https://blog.csdn.net/qq_40181592/article/details/131393536)
- [CSDN Article 2](https://blog.csdn.net/qq_42973562/article/details/132067549)
