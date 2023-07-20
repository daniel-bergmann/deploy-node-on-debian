# Node.js app deployment for Debian Linux based distros

If you have a domain, configure on your service provider - SSL is taken care of later in the process and is not needed to deploy the system but you need a domain to configure SSL. 

## Node.js

---

1. Install nvm, Node.js, yarn, pnpm..
    - `curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash`
    - `source ~/.bashrc`
    - `command -v nvm`
    - `npm i yarn -g`
2. **Node source** - Configure Node.js so it works better with the Debian APT manager.
    - Find the LTS one for your OS here: https://github.com/nodesource/distributions and run it.

## Nginx & the server firewall

---

1.  Install nginx by running `sudo apt-get install nginx`
2. Configuring the firewall
    - Allow SSH so the server won’t lock you out, run `sudo ufw allow 'OpenSSH’`
    - Allow Nginx, run `sudo ufw allow 'Nginx HTTP'`
    - run `sudo ufw enable` and type `y` for yes - Expected output: `Firewall is active and enabled on system startup`
    - run `sudo ufw status` and output should resemble this:
    - Action should have ALLOW in the whole column.
    
    ```jsx
    To                         Action      From
    --                         ------      ----
    OpenSSH                    ALLOW       Anywhere                  
    Nginx HTTP                 ALLOW       Anywhere                  
    OpenSSH (v6)               ALLOW       Anywhere (v6)             
    Nginx HTTP (v6)            ALLOW       Anywhere (v6)
    ```
    
    1. Copy your ip address to access the Nginx default site in the browser. If it fails, make sure you’re using `HTTP` but not `HTTPS`
    
    ## /var/www/~
    
    1. make a dir for the site by running `sudo mkdir /var/www/the-domain.com`
    2. make a conf file for Nginx by running `sudo vim /etc/nginx/sites-available/the-domain.com`
    3. Copy the below code and change the `server_name` and `proxy_pass` according to your app. Check the port in use.
    
    ```jsx
    server {
        server_name the-domain.com www.the-domain.com;
    
        location / {
            proxy_pass http://localhost:3000;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
        }
    }
    ```
    
    1. To activate your new site instead of the Nginx default one, run `sudo ln -s /etc/nginx/sites-available/the-domain.com` `/etc/nginx/sites-enabled/the-domain.com`
    2. Check if the Nginx syntax in the config file is correct by running `nginx -t`
        - Output should be so:
        
        `nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
        nginx: configuration file /etc/nginx/nginx.conf test is successful`
        
    3. reload Nginx, run `nginx -s reload`
    4. Now you should see a `502 **Bad Gateway**` error if you put the IP address in the browser because it’s looking for the website on localhost. There is no Node.js app running. So we need to start running it.
    5.  run `cd /var/www/the-domain.com/` to go to the dir where your app should live and run.
    6. clone your project to this dir so the `package.json` and all supposed equally nested files are together.
    7. Move all your files in the cloned directory into the [`the-domain.com](http://the-domain.com)` dir by running `mv name-of-cloned-dir/* .`
    8. Run `yarn, pnpm install` or `npm install` to install dependencies.
    9. Start your server with `node name-of-root-file.js`
    10. If you can see your app in the browser using the IP address, close the app and continue to next step.
    
    ## PM2 - keep app running after you exit your session
    
    1. Install PM2 by running: `sudo npm install pm2@latest -g` **note: you cannot use yarn or pnpm** to install PM2**.**
    2. Start PM2 by running `pm2 startup systemd`
    3. Start Node server with `pm2 start name-of-root-file.js`
    4. Run `pm2 save` to save the instance in root
    5. Run `sudo systemctl start pm2-root`

       Now go fuck shit up.
