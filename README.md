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
+ Create a Droplet running Arch Linux using the 'doctl' command-line tool
+ Use 'doctl' and cloud-init for every stage of setting up an Arch Linux droplet

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
+ You will also be prompted to enter a passphrase, add one if you'd like.


*We are using `ed25519` for the digital signature generation and verification since it is generally more secure and efficient than `rsa`.*

This will generate a private key: "do-key" and a public key: "do-key.pub" which we will be using for the server.

You will want to copy the the public key so you can paste it to your DigitalOcean account. This is a better alternative than having to copy it from the plain text file that it would be found in.

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
    passwd: NewUserPassword  # local access (not used for SSH)

package_update: true
packages:
  - htop
  - vim
  - git

ssh_pwauth: false
disable_root: true
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

To connect to your server use the command:\
```ssh -i .ssh/do-key arch@droplet_ip_address ```
+ Replace `droplet_ip_address` with the ip address of droplet which should be found right beside its name on DigitalOcean.
+ In the `cloud-init` configuration, `name: newuser` would be the name you use in place of `arch`

### Creating a Droplet running Arch Linux using the 'doctl' command-line tool
To create and manage a DigitalOcean Droplet from your terminal, you will need to use the `doctl` command-line tool.
1. Install doctl on your local machine:\
``` sudo pacman -S doctl```
*Run this command in your VM for Arch Linux*
2. Go to DigitalOcean, and click **API** located on the left side.
3. Under **Tokens**, click **Generate a New Token**.
  + You can give it any name, but it may be easier set as **doctl**.
  + For Scope, you can give it **Full Access** this will give it a read and write scope.
![Creating a new personal access Token](<Screenshot 2024-09-29 151928.png>)\
*You can either use Full Access (if you want to simplify things) or Custom Scopes.*
4. Authenticate doctl by running the following command:\
``` doctl auth init```
5. Enter your access token which is copied from DigitalOcean
6. You will want to list all the available images on DigitalOcean by running the following:\
```doctl compute image list --public```
 *Look for the Arch Linux image in the output and copy the **ID** of it.*
 7. You will also need the ssh-key id, run the following command and copy the `do-key` id:\
``` doctl compute ssh-key list```
 8. You will want to create a new Droplet using the following `doctl` command:
 ```
 doctl compute droplet create arch-droplet \
  --region SFO3 \
  --size s-1vcpu-1gb \
  --image arch_image_id \
  --ssh-keys ssh_key_id \ 
  --wait
```
+ Replace `arch-droplet` with the name you want for the droplet.
+ Keep the region the same.
+ Replace `arch_image_id` with your arch image ID from step 6
+ Replace `ssh_key_id` with your ssh key ID from step 7
+ The `--wait` flag will ensure the command waits until the Droplet is fully created.

### Using 'doctl' and cloud-init for every stage of setting up an Arch Linux droplet
You can automate the Droplet setup using a cloud-init configuration file. This will allow you to automate tasks.
1. Create a `cloud-init.yml` configuration file
You can create the `cloud-init.yml` file using any text editor.\

In Windows:
  1. Open **Notepad** or **VS Code**.
  2. Create a new file and name it `cloud-init.yml`.
  3. Save it to a directory of your choice (e.g.,`C:\Users\your-user-name\cloud-init.yml`).

If you are on MacOS or Linux:
  1. Open a terminal and use `nano` or `vim` to create the file:\
```nano ~/cloud-init.yml```

2. Now we will need to define the contents of the cloud-init.yml file so copy the following into the file:
```
#cloud-config
users:
  - name: newuser
    ssh_authorized_keys:
      - ssh-ed25519 YourPubKey
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: sudo
    shell: /bin/bash
    passwd: NewUserPassword

package_update: true
packages:
  - htop
  - curl
  - vim

ssh_pwauth: false
disable_root: true
```
+ The `users` section creates a new user called `newuser`, with SSH key-based authentication.
    + The public SSH key (replace `yourPubKey` with your actual SSH key) is added to the `~/.ssh/authorized_keys` file.
    + The new user is added to the `sudo` group and granted root privileges with `NOPASSWD:ALL`, allowing the user to run `sudo` without entering a password.
    + A password (`NewUserPasswordHere`) is set, but since SSH key authentication is used, the password is primarily for local access if needed.\

+ The `package_update: true` directive ensures the package lists are updated before installing software.
+ The `packages` installs essential packages like `htop`, `vim`, and `git` automatically.
Feel free to add any other packages you need for your specific setup but those are the ones included in this tutorial.\

+ The `ssh_pwauth: false` directive disables SSH password authentication, enforcing SSH key-based authentication.
+ The `disable_root: true` option disables root login via SSH for security reasons, ensuring no direct root login is allowed over SSH.\

3. Save the cloud-init file after you've written the content and save the file:
  + On Windows: save the file to your desired folder (e.g., `C:\Users\your-user-name\cloud-init.yml`).
  + On MacOS or Linux: Press **CTRL + O** to save in `nano` and then **CTRL + X** to exit the editor

4. Make sure doctl is installed on your local machine if not run the following command:\
For Windows, install scoop which will be used to install doctl:
```
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
iwr -useb get.scoop.sh | iex
```
+ Install doctl using scoop:\
  ```scoop install doctl```

For MacOS if you don't have Homebrew then make sure to install then run:\
``` brew install doctl```
  + Verify the installation:\
  ```doctl version```

For Linux you can install doctl using the snap package manager:\
``` sudo snap install doctl```

5. Once doctl is install, authenicate it with your DigitalOcean account by running:\
``` doctl auth init```
*Make sure to generate a new token as it will ask for an access token*

6. Once the cloud-init.yml file is ready, you can use it during the creation of a Droplet with the following command:\
For Windows:
```
doctl compute droplet create my-arch-droplet `
  --region SFO3 `
  --size s-1vcpu-1gb `
  --image arch_image_id `
  --ssh-keys ssh_key_id `
  --user-data-file C:\Users\your-user-name\cloud-init.yml
  ```
For MacOS or Linux:
```
doctl compute droplet create my-arch-droplet \
  --region SFO3 \
  --image arch_image_id \
  --size s-1vcpu-1gb \
  --ssh-keys your-ssh-key-id \
  --user-data-file ~/cloud-init.yml
```

+ Replace `arch-droplet` with the name you want for the droplet.
+ Keep the region the same.
+ Replace `arch_image_id` with your arch image ID obtained from an above step.
+ Replace `ssh_key_id` with your ssh key ID obtained from an above step.
+ Replace `C:\Users\your-user-name\` with the path to your `cloud-init.yml`

If you followed these steps, the cloud-init.yml file will be used to fully automate the Droplet's setup.