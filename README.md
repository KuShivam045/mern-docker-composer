How to Deploy a React.js Website on AWS EC2 with Nginx
Deploying a React.js website on AWS EC2 using Nginx is a powerful and scalable solution for your web applications. Whether you're just starting out or scaling up for production, this guide will walk you through the process in a clear and straightforward way.

Here’s what you’ll learn:

Setting up an EC2 instance
Configuring your environment
Deploying a React app
Setting up Nginx as a reverse proxy to serve your app
Ready? Let’s get started!

Step 1: Setting Up an AWS EC2 Instance
First things first, you need an EC2 instance to host your React app. Head over to the AWS Management Console:

Launch an Instance:

Go to EC2 Dashboard and click "Launch Instance."
Select Ubuntu Server 20.04 LTS as your AMI.
Choose your instance type (for example, t2.micro works fine for small apps).
Configure the details as needed, then move on to Security Groups.
Security Group Setup:
Add inbound rules for SSH (port 22) and HTTP (port 80). These are essential for accessing your server and making your app available to the public.

Launch and Connect:
Launch the instance and download your SSH key pair. Connect to the instance using SSH:
 
ssh -i your_key.pem ubuntu@<EC2-Public-IP>
Now you’re in!

Step 2: Installing Node.js, NPM, and PM2
Once inside your EC2 instance, let’s install Node.js, npm, and PM2 to serve your React app.

Update Packages:
 
sudo apt update && sudo apt upgrade -y
Install Node.js and NPM:

Use NodeSource to get the latest version of Node.js:
 
curl -sL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt install nodejs -y
Confirm that Node.js and npm are installed:
 
node -v
npm -v
Install PM2:

PM2 helps manage your application processes:
 
sudo npm install pm2@latest -g
Step 3: Setting Up and Building Your React App
Transfer Your React App to EC2:

You can either clone your project from GitHub or transfer your files using scp or similar tools.

Example for cloning from GitHub:
 
git clone https://github.com/your-repo/react-app.git
cd react-app
Install Dependencies:

In your project directory, run:
 
npm install
Build the React App:

Create a production build of your app:
 
npm run build
Serve the App with PM2:

Use serve and PM2 to host your app on a specific port (e.g., 3000):
 
sudo npm install -g serve
pm2 serve build 3000 --name "react-app"
Enable PM2 on Reboot:

This ensures that PM2 starts your app automatically when the server restarts:
 
pm2 startup
pm2 save
Step 4: Installing and Configuring Nginx
Nginx will act as a reverse proxy to route requests to your React app.

Install Nginx:
 
sudo apt install nginx -y
Configure Nginx:

Open the default Nginx config file:
 
sudo nano /etc/nginx/sites-available/default
Replace the contents with the following configuration:

nginx
Copy code
server {
    listen 80;
    server_name your_domain_or_IP;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
This config sets Nginx to forward all traffic to the React app running on port 3000.

Test and Restart Nginx:

Test the configuration:
 
sudo nginx -t
If all is well, restart Nginx:
 
sudo systemctl restart nginx
sudo systemctl enable nginx
Step 5: Configuring Security Groups and Firewalls
Update EC2 Security Groups:

Make sure your security group allows traffic on port 80 (HTTP) and 22 (SSH). This can be done in the AWS console by editing the inbound rules of your instance's security group.

Optional: Configure UFW:

If you want extra security, you can enable UFW (Uncomplicated Firewall):
 
sudo ufw allow 'Nginx Full'
sudo ufw enable
Step 6: Access Your React App
You’re done! 🎉 Now head to your browser and type your EC2 instance’s public IP address. If everything went smoothly, you should see your React app live.

If you have a domain, you can point it to your EC2 instance by updating your DNS settings (e.g., create an A record pointing to the public IP). This will make your app accessible via the domain name.