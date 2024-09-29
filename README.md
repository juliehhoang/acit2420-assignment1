# ACIT 2420 Assignment 1

By completing this tutorial, you will:
+ Generate SSH keys for secure server access
+ Create a droplet runniong Arch Linux using DigitalOcean
+ Automate the initial setup tasks using a cloud-init file
    + Create a new user
    + Install packages
    + Add a public SSH key
    + Disable root login using SSH
    

### What is a SSH key?
Secure Shell (SSH) keys are an authentication method that allows for a secure way of sending commands to a computer over an unsecured network. 

Using SSH keys to authenticate a user's indentity allows for greater protection of their data. Username/password authentication is less favourable since it can often lead to security compromises.

### Generating a new SSH key
You can generate a new SSH key on your local machine using `ssh-keygen` which should already be installed on Windows and MacOS.

Check to see if you have an .ssh directory in your home directory, if not try running:
``ssh-keygen -t ed25519 -f ~/.ssh/do-key -C "your email address"``

`-t` is the type of encryption for your key
`-f` is for specifying the filename and the location
`-C` is for the comment

In PowerShell, MacOS, or Linux you can generate an SSH key pair by running:
``ssh-keygen -t ed25519 -f C:\Users\your-user-name\.ssh\do-key -C "youremail@email.com"``
+ Make sure to replace `youremail@gmail.com` with your actual email, and update the path for your specific username


*We are using `ed25519` for the digital signature generation and verification since it is generally more secure and efficient than `rsa`*
