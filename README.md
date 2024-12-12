# 🛠️ vServer Setup Guide

Welcome to the vServer setup guide! 🚀  
This document will guide you through setting up SSH keys, configuring an Nginx web server, and connecting your server to GitHub.

---

## 📋 Table of Contents

1. [Setup SSH Key Connection](#-1-setup-ssh-key-connection)
2. [Configure Nginx Web Server](#-2-configure-nginx-web-server)
3. [Connect to GitHub](#-3-connect-to-github)

---

## 🔑 1. Setup SSH Key Connection

### 1.1 Generate an SSH Key

To organize your SSH keys, create a separate folder for each project:

Bash

```bash
  mkdir ~/.ssh/project_folder
```

Powershell

```powershell
  mkdir C:\Users\user\.ssh\subfolder
```

### 1.2 Generate a new SSH key:

Bash

```bash
  ssh-keygen -t ed25519 -C "user@mail.com" -f ~/.ssh/project_folder/filename
```

Powershell

```powershell
  ssh-keygen -t ed25519 -C "user@mail.com" -f C:\Users\user\.ssh\subfolder\filename
```

- `-t` **t**ype of Key you can also choose another one
- `-C` **C**omment to identify your key
- `-f` specifies the **f**ile path and name for storing the key file

You do not need to enter a passphrase and can skip entering it by pressing Enter. It is recommended to increase security.

```bash
  Enter passphrase:
```

> Now you have 2 new files in your folder
>
> - Private Key "filename" | **ONLY** for **YOU**
> - Public Key "filename.pub" | to share

### 1.3 Save the created key on the server

To do this, we copy the desired key to the server.

Bash

```bash
  ssh-copy-id vserver-user@vserver_ipv4
```

Powershell

1. Copy the key to your server in the home directory of the server user

```powershell
  scp C:\Users\user\.ssh\subfolder\filename.pub user@vserver_ipv4:~
```

2. Login on your vServer with

```powershell
  ssh user@vserver_ipv4
```

3. Write your public-key in _authorized_keys_ flie

```bash
  cat <filename.pub> >> .ssh/authorized_keys
```

4. Delete with remove your public-key file

```bash
  rm ~/<filename.pub>
```

5. Print the contents of the file to check if everything worked.

```bash
  cat ~/ssh/authorized_keys
  ssh-ed25519 AAAAC3NzaC1lZFFAA13645B23AAAAIFE7TXS31fHp+/MbA4YlX4cG2OTQStMtX3R6+TDssBk0 user@mail.com
```

### 1.4 Check Connection

To test the login with the public key we connect to the server via sshkey.

```bash
  ssh -i ~/.ssh/subfolder/filename vserver-user@vserver_ipv4
```

```powershell
  ssh -i C:\Users\user\.ssh\subfolder\filename user@server_ipv4
```

- `-i` **i**dentity file by the path to your private key

### 1.5 Setup a short alias for connection

add to your local machine `./ssh/config` file

> ```
>  Host serverAlias
>    HostName hostname or IPv4
>    User user
>    IdentityFile ~/.ssh/subfolder/filename
> ```

```powershell
  ssh serverAlias
```

### 1.6 Update vServer connection rules

Login with key & passphrase

```bash
  ssh serverAlias
  Enter passphrase for key 'C:\Users\user\.ssh\subfolder\filename':
```

Open the SSH configuration file with the text editor nano. You can only change the file if you have root rights.
To confirm root rights, enter the user password.

```bash
  sudo nano /etc/ssh/sshd_config
```

find PasswordAuthentication

```bash
  #PasswordAuthentication yes
```

and change it to

```bash
  PasswordAuthentication no
```

Save file & Exit nano

> save file STRG + O
> quit file STRG + X

To make the changes effective, the service must be restarted.

```
sudo systemctl restart sshd
```

---

## 🖥️ 2. Configure Nginx Web Server

### 2.1 Install Nginx

Update your system and install Nginx:

```bash
  sudo apt-get Update && sudo apt-get upgrade -y
  sudo apt-get install nginx -y
```

Open your browser an navigate to "http://server_ipv4", you can see the nginx.defautl webpage.

![nginx_page](./img/nginx_page.png)

### 2.2 Configure Nginx

Remove the default folder & file:

```bash
  sudo rm -r /var/www/html/
```

Create a new project directory:

```bash
  sudo mkdir /var/www/my_page
```

Add an index.html file:

```bash
  sudo nano /var/www/my_page/index.html
```

Example content:

```html
<!DOCTYPE html>
<html lang="de">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>MY_PAGE</title>
    <style>
      body {
        background-color: darkblue;
      }
      h1 {
        color: white;
      }
    </style>
  </head>
  <body>
    <h1>Welcome to my-page</h1>
  </body>
</html>
```

Copy the default file to save the settings

```bash
  sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/default_copy
```

Remove file link

```bash
  sudo rm /etc/nginx/sites-enabled/default
```

Rename the default file to my_page

```bash
  sudo mv /etc/nginx/sites-available/default /etc/nginx/sites-available/my_page
```

Open the my_page file with nano

```bash
  sudo nano /etc/nginx/sites-available/my_page
```

Add your nginx webserver configuration

```
server {
        listen 8081;

        root /var/www/my_page;

        index index.html;

        server_name server_ipv4 hostname;

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ =404;
        }
}
```

Save file & Exit nano

> save file STRG + O
> quit file STRG + X

```bash
  sudo ln -s /etc/nginx/sites-available/my_page /etc/nginx/sites-enabled/
```

Test your nginx settings

```bash
  sudo nginx -t
  nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
  nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Adjust user rights for the project for nginx

```bash
  sudo chown -R www-data:www-data /var/www/my_page/
```

Restart Nginx service to apply the settings

```bash
  sudo systemctl restart nginx
```

Open your browser an navigate to "http://server_ipv4:8010", now you can see your webpage.

![my_page](./img/my_page.png)

---

## 🌐 3. Connect to GitHub

### 3.1 Install Git

Install Git on your server

```bash
  sudo apt-get install git -y
```

### 3.2 Set Up GitHub SSH Key

Generate a new SSH key for the server:

```bash
  ssh-keygen -t ed25519 -C "hostname"
```

Display the public key:

```bash
  cat ~/.ssh/id_ed25519.pub
```

Add the key to your GitHub account:

**GitHub > Settings > SSH and GPG keys > New SSH Key**

### 3.3 Configure Git

Set up your Git username and email:

```bash
  git config --global user.name "Your Name"
  git config --global user.email "your_email@example.com"
```

Now you have full access to your GitHub account and can clone repositories, work on them and upload them again.

```bash
  git clone git@github.com:YourName/vServer_setup.git
```

---

### 🎉 Done!

Your vServer is now configured with SSH, an Nginx web server, and GitHub integration.
