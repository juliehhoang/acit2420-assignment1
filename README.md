# ACIT 2420 Assignment 1

By completing this tutorial, you will:
+ Generate SSH keys
+ Add a custom Arch Linux image using the web console
+ Create a new Droplet running Arch Linux using DigitalOcean web console
+ Automate the initial setup tasks using a cloud-init file
    + Create a new regular user
    + Install some initial packages
    + Add a public ssh key to the `authorized_keys` file in your new users home directory
    + Disable root access via SSH

### What is a SSH key?
Secure Shell (SSH) keys are an authentication method that allows for a secure way of sending commands to a computer over an unsecured network. 

Using SSH keys to authenticate a user's indentity allows for greater protection of their data. Username/password authentication is less favourable since it can often lead to security compromises.

### Generating a new SSH key
You can generate a new SSH key on your local machine using `ssh-keygen` which should already be installed on Windows and MacOS.

Check to see if you have an .ssh directory in your home directory, if not try running:\
```ssh-keygen -t ed25519 -f ~/.ssh/do-key -C "your email address"```

+ `-t` is the type of encryption for your key
+ `-f` is for specifying the filename and the location
+ `-C` is for the comment

In PowerShell, MacOS, or Linux you can generate an SSH key pair by running:\
```ssh-keygen -t ed25519 -f C:\Users\your-user-name\.ssh\do-key -C "youremail@email.com"```

+ Make sure to replace `youremail@gmail.com` with your actual email, and update the path for your specific username.
+ You will also be prompted to enter a passphrase, it isn't required but you can add one if you want.


*We are using `ed25519` for the digital signature generation and verification since it is generally more secure and efficient than `rsa`.*

This will generate a private key: "do-key" and a public key: "do-key.pub" which we will be using for the server.

You will want to copy the the public key so you can paste it to your DigitalOcean account. This is an easier alternative to having to copy it from the plain text file that it would be in.

PowerShell:\
```Get-Content C:\Users\your-user-name\.ssh\do-key.pub | Set-Clipboard```

MacOS:\
``` cat ~/.ssh/do-key.pub | pbcopy```

Linux:\
``` cat ~/.ssh/do-key.pub | wl-copy```\
*You can always replace `wl-copy` with which ever command works for system but this is commonly used*

### Adding a Custom Arch Linux Image Using the Web Console
1. Log in to your DigitalOcean, then click **Backups & Snapshots** located on the left side.
2. Under **Custom Images**, and click **Upload Image**.
    + Distribution: Arch Linux
    + Datacenter: San Francisco 3 (SF03)
![What the Upload an Image should look like](<Screenshot 2024-09-29 002114.png>)
3. Once uploaded, make sure the image is available for creating Droplets.
![Showing where uploaded images would show up](<Screenshot 2024-09-28 235927-1.png>)

### Creating a new Droplet Running Arch Linux Using DigitalOcean Web Console
1. Click **Droplets** located on the left side and click **Create Droplet**.
2. Select the custom Arch Linux image you uploaded earlier.
    + Datacenter: San Francisco 3 (SF03)
    + Choose size: whichever you prefer
    + CPU: 7$/month or whichever you prefer
    + Choose Authentication Method: the SSH Key you created
    + Hostname: name it something you don't mind seeing since it will appear in your prompt when connect to the server
**Don't** click **Create Droplet** if you plan on automating the inital setup tasks using a cloud-init file

### Automate the initial setup tasks using a cloud-init file
1. Under **Choose Authentication Method**, click **Advanced Options**.
2. Click **Add Intialization scripts**.
3. Paste the following cloud-init configuration file:
```
#cloud-config
users:
  - name: newuser
    ssh_authorized_keys:
      - ssh-ed25519 yourPubKey  
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: sudo
    shell: /bin/bash
    passwd: NewUserPasswordHere  # local access (not used for SSH)

package_update: true
packages:
  - htop
  - vim
  - git

ssh_pwauth: false
disable_root: true

# Final message after setup so we know it worked
final_message: "Cloud-init has completed the setup tasks."

```
+ This `cloud-init` configuration automates the setup process when creating a new Droplet on DigitalOcean or other cloud platforms.\

+ The `users` section creates a new user called `newuser`, with SSH key-based authentication.
    + The public SSH key (replace `yourPubKey` with your actual SSH key) is added to the `~/.ssh/authorized_keys` file.
    + The new user is added to the `sudo` group and granted root privileges with `NOPASSWD:ALL`, allowing the user to run `sudo` without entering a password.
    + A password (`NewUserPasswordHere`) is set, but since SSH key authentication is used, the password is primarily for local access if needed.\

+ The `package_update: true` directive ensures the package lists are updated before installing software.
+ The `packages` installs essential packages like `htop`, `vim`, and `git` automatically.
Feel free to add any other packages you need for your specific setup but those are the ones included in this tutorial.\

+ The `ssh_pwauth: false` directive disables SSH password authentication, enforcing SSH key-based authentication.
+ The `disable_root: true` option disables root login via SSH for security reasons, ensuring no direct root login is allowed over SSH.\
4. Click **Create Droplet**









