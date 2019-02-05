# Getting a React app running with json-server on Digital Ocean
This will help you setup subdomains to serve your React Frontend and its json-server backend.
This assumes you've followed Steve's steps to get your Droplet up and running.

<br>

## GitHub setup
Follow GitHub's instructions to setup an SSH key ***on your Droplet*** and add it to your Github account.
This allows you to clone from your repos on Github directly to your Droplet.

<br>

## Update Node JS
1. `sudo npm install -g n`
2. `sudo n stable`<br>
This will get you the latest version of NodeJS on your Droplet. I had issues installing packages before this.

<br>

## Install json-server and forever globally
1. `sudo npm install -g json-server forever`<br>
We'll use `forever` in the last step to keep json-server running, well, forever.

<br>

## Point subdomains to your Droplet IP Address
You will need a subdomain for the frontend, and a subdomain for json-server.<br>
Wherever you bought your domain, go to your DNS settings and create A records for the two subdomains.<br>
**Point them to the IP address of your Droplet.**

ex: I created `barkeeper.sebastiancivarolo.com` and `barkeeper-json.sebastiancivarolo.com`

<br>

## Cloning your React Project to your Droplet
1. In your home directory in your droplet, clone down your react project:<br>
`git clone [repo-address] [project-folder]`
2. `npm install`<br>
**BEFORE YOU BUILD** Update your API manager to point to the new subdomain your created instead of `http://localhost:5002`.
3. `sudo npm run build` This will create a production version of your react project in a `/build` folder.

<br>

## Create a directory for your react subdomain
This format will help you stay organized and know where each subdomain server lives.
<br>In `~` aka `home/{username}/`: 
- `mkdir ~/www`
- `cd ~/www`
- `mkdir {your-subdomain.folder.com}` (name it the same as the subdomain url. just do it.)
- `mkdir {your-json-subdomain.folder.com}` You can stick your database.json here, or just know where you're keeping it.
- `cd` back to the folder where you cloned your git repo (`cd ~/{repo-folder}`)
- `cd build` Should take you into the build folder

<br>

## Copy the contents of the build folder to your subdomain folder
- `cp -r * ~/www/your-subdomain.folder.com`
- You can `cd ~/www/your-subdomain.folder.com` and `ls` to check the files copied.
- Also copy your `database.json` to your json subdomain directory or wherever you want to store it.

<br>

## The NGINX stuff
Now we need to tell NGINX where our subdomains need to point.

1. `sudo touch /etc/nginx/sites-available/yourdomain.com`
2. `sudo vi /etc/nginx/sites-available/yourdomain.com`

<br>

### Create the server blocks for your two subdomains
Both go in the file we are editing now. 
**Semicolons are crucial, you'll probably forget one and get an error**
<br>
For your react project:
```
server {
        listen 80;
        
        root /home/{username}/www/yoursubdomain.folder.com;
        index index.html;
        
        server_name yoursubdomain.url.com;
        
        location / {
                try_files $uri /index.html;
        }
}
```

For json-server:
```
server {
        listen 80;
        
        server_name your-json.subdomain.com;
        
        root /home/{username}/www/your-json.subdomain.com;
        
        location / {
                proxy_pass http://localhost:5002/;
        }
        
}
```

Your file should look like this: **Two separate server blocks**
```
server {
        ...
        ...
}
server {
        ...
        ...
}
```


To exit: `esc` then `:q` and return

- We need to symlink this file into sites-enabled:<br>
`sudo ln -s /etc/nginx/sites-available/yourdomain.com /etc/nginx/sites-enabled/yourdomain.com`

- Restart NGINX
`sudo service nginx restart`<br>
If you get an error, check your file.<br>
`sudo systemctl status nginx.service` should help tell you why it didn't work.

<br>

## Start JSON-server
- `cd` into the directory where you put your database.json
- `forever start -c "json-server -p 5002 -w db.json" ./`

Other useful commands:<br>
`forever stopall`
`forever list`
`forever restartall`

<br>

## You did it!
Go to `yoursubdomain.domain.com` and hopefully it works!

<br>

## Updates to your app
- If you make updates, `git pull` the latest version
- `sudo npm run build`
- `rm -rf ~/www/subdomain.yoursite.com/*`
- `cd ~/your-repo-folder/build`
- `cp * ~/www/subdomain.yoursite.com/`

<br>

#### Credit
Adapted from https://albertogrespan.com/blog/running-multiple-domains-or-subdomains-in-nginx-with-server-blocks/
