# Deploy a Flask or Django app in VPS
How to deploy a Flask app in VPS like AWS or Google Cloud console, we'll learn from here

# 1. Connecting to the VPS using FTP for file uploads
At first, we need to upload all our project files inside a folder in the VPS. You can use any FTP software such as WinSCP in Windows or Filezilla for Linux or Mac or anything else as you prefer.

# 2. Connecting to the VPS SSH terminal
Now, we have to connect to the VPS terminal using Putty or any other SSH software<br>
Once connected, let's run these commands<br>
First, we update the package list
<pre>sudo apt update</pre>
Now we install nginx to use it as a server software
<pre>sudo apt install nginx</pre>
Now we install MySQL server, we'll configure it later
<pre>sudo apt install mysql-server</pre>
Now we will start the MySQL service in the background
<pre>sudo systemctl start mysql.service</pre>
Time for installing Python
<pre>sudo apt install python3</pre>
and pip
<pre>sudo apt install python3-pip</pre>
As it will be in a production server, we'll use gunicorn for this
<pre>sudo apt install gunicorn</pre>

# 3. Installing the library dependency
We need to install the libraries in the server using pip command. If you have a requirements.txt containing the libraries list, you can run this command to install all of them together
<pre>sudo pip install -r requirements.txt</pre>
If you're using an AWS EC2 instance, it may not allow you to install using the pip command, it will require you to install the libraries using the command like this
<pre>sudo apt install python3-library-name</pre>
For example, if you want to install Flask in AWS EC2, you have to install it like this
<pre>sudo apt install python3-flask</pre>
Although some libraries may not work this way, you can use this command in that case (python-pptx is an example of such a library)
<pre>sudo python3 -m pip install python-pptx --break-system-packages</pre>

# 4. Setting up MySQL server configuration
Let's configure the MySQL server now
<pre>sudo mysql</pre>
Create a strong password and use this command and change the value 'password' with your desired password below (only copy the content after mysql>)
<pre>mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
mysql> exit
</pre>
<pre>mysql -u root -p</pre>
<pre>mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH auth_socket;
exit</pre>
<pre>sudo mysql_secure_installation</pre>
then follow the instructions and finish. Copy your previously generated strong password. Use this command and change the value 'password' with your desired password below
<pre>sudo mysql</pre>
<pre>mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';</pre>
Now you can create your database or import tables/databases as needed from this console. Type exit to exit from the MySQL console.<br>
Note down the login and user information or alter it according to how you want it to work and update your Flask/Django config with the MySQL info.

# 5. Configuring the server proxy to listen to the Flask port
First, get the server IP address. We'll create a configuration file for the proxy server first. Let's run this command
<pre>sudo nano /etc/nginx/sites-enabled/"server IP here"</pre>
This will open an editor. Paste this code inside (replace <YOUR INSTANCE IP> with your server IP)
<pre>
server {
    listen 80;
    listen [::]:80;
    server_name <YOUR INSTANCE IP>;
        
    location / {
        proxy_pass http://127.0.0.1:5000;
        include proxy_params;
    }
}
</pre>
press ctrl+o to write the changes and press ctrl+x to exit nano editor
Now we will restart the nginx server
<pre>sudo systemctl restart nginx</pre>

# 6. Setting up gunicorn and deploying our server
For Django tutorial, check <a href="https://docs.gunicorn.org/en/latest/deploy.html">Official Gunicorn Tutorial</a><br>
For Flask, go to the main folder where your app.py or the main python file is. Create a file named <code>wsgi.py</code> and place this
<pre>
from app import app

if __name__ == "__main__":
    app.run()
</pre>

Now, back to the terminal again. Run this command (make sure to stay on the same path where you have wsgi.py)
<pre>gunicorn --bind 0.0.0.0:5000 wsgi:app</pre>
If everything is good, it will show no error and should serve the website (confirm it by accessing the server IP in a browser)<br>
If it's working, come back to the terminal and press ctrl+c to close the process as we're not done yet.<br>
Run this command (change the path correctly)
<pre>gunicorn --workers 3 --bind unix:/home/path/to/app/app.sock -m 777 wsgi:app</pre>
press ctrl+c to terminate the process again after no output is printing for a moment.
<pre>sudo nano /etc/nginx/sites-available/app</pre>
Now we use the same path for app.sock from the previous command and paste it inside the editor
<pre>
  server {
   listen 80;
 
   location / {
       include proxy_params;
       proxy_pass http://unix:/home/path/to/app/app.sock;
   }
}
</pre>
Press ctrl+o to save the changes and press ctrl+x to exit<br>
run the command
<pre>sudo ln -s /etc/nginx/sites-available/app /etc/nginx/sites-enabled/</pre>
<pre>sudo nginx -t</pre>
<pre>sudo systemctl restart nginx</pre>
<pre>sudo ufw allow 'Nginx Full'</pre>
<pre>gunicorn --bind 0.0.0.0:5000 wsgi:app --daemon</pre>
And we're done! Your website is running perfectly.<br>
To stop and restart your flask app after any changes, run this command after coming to the folder where you have your <code>wsgi.py</code> file
<pre>sudo pkill -f gunicorn</pre>
<pre>gunicorn --bind 0.0.0.0:5000 wsgi:app --daemon</pre>

Thanks for reading. If this helps, please star this repository and share it with your friends.<br>
If you need a developer to deploy your web app, reach out to me @ <a href="https://www.fiverr.com/thechoyon">Fiverr</a>
