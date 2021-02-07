# How to create a dedicated Valheim server on Ubuntu 20.04 and expose it on the internet

## Prerequisites 
- Stable internet connection;
- Configurable router that permits you to expose ports;
- Dedicated machine or virtual machine running Ubuntu 20.04 either Desktop or Server(Recommended);

## Step 1 - Update your system
Before continuing, make sure you've updated you server by running the command:
```bash
sudo apt update && sudo apt upgrade -y
```

## Step 2 - SSH server(Optional)
This is just to make things easier so you can configure your server remotely via SSH which also works great with [Putty](https://www.putty.org/) and [WinSCP](https://winscp.net/eng/download.php) so you can manage it from your Windows machine. To install, run command:
```bash
sudo apt install ssh
```
## Step 3 - Firewall
This is not optional because I strongly suggest you configure this for security reasons. For this we can use `ufw` to make things simpler, and it should be already installed on Ubuntu 20.04(run one by one):
- for SSH, which is port 22 by default(if you decided to add SSH):
```bash
sudo ufw allow ssh
```
- for the Valheim server:
```bash
sudo ufw allow 2456
sudo ufw allow 2457
sudo ufw allow 2458
```
Activate the firewall:
```bash
sudo ufw enable
```
**IMPORTAT!** Remember to add a port forwarding rule for each port on your router if you want to expose this server on the internet.

## Step 4 - Software dependencies

I'm not even going to ask if you have a 64-bit system, because 32-bit is already old and non-existent with this version of Linux. In this case you need to install the following software via `apt` one by one:
```bash
sudo apt install software-properties-common
sudo add-apt-repository multiverse
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install lib32gcc1 steamcmd
```
## Step 5 - Create a steam user
It is recommended that you create a dedicated user for your server. In our case we will create user `steam` with the following command:
```bash
sudo useradd -m -s /bin/bash steam
```
We will use this user from the moment, so we can log in with him now:
```bash
sudo su - steam
```
## Step 6 - Install
As `steam` user we will start the installation process, and for this we need to link the steamcmd (in `steam` home directory):
```bash
ln -s /usr/games/steamcmd steamcmd
```
Running the command will take a while to download the update, but it's necessary:
```bash
steamcmd
```
After the download is complete, you will be prompted to the steamcmd command line:
```
Steam>
```
Valheim will allow you to download the dedicated server as an anonymous steam user, but some games will require you to login with your actual user. We will use the following command:
```
Steam> login anonymous
```
Output:
```
Connecting anonymously to Steam Public...Logged in OK
Waiting for user info...OK
```
We will want to install the Valheim server in a certain folder, you are free to use any folder as long as it has permissions for `steam` user, but for the purpose of this tutorial we will use `/home/steam/valheim` folder, so we run the command:
```
Steam> force_install_dir /home/steam/valheim
```
Now we want to download the server files and executables, so first of all we need the SteamAppId, in our case the ID is `896660` and you can check on https://steamdb.info/apps/ and search for "Valheim Dedicated Server".
The command to download is:
```
Steam> app_update 896660 validate
```
Output for downloading:
```
 Update state (0x3) reconfiguring, progress: 0.00 (0 / 0)
...
 Update state (0x61) downloading, progress: 87.82 (917786556 / 1045059920)
 Update state (0x0) unknown, progress: 0.00 (0 / 0)
Success! App '896660' fully installed.
```
Now you can check the folder `/home/steam/valheim` to see it's content:
```bash
ls -lha /home/steam/valheim
```
Output:
```bash
total 64M
drwxrwxr-x 5 steam steam 4.0K Feb  7 00:56  .
drwxr-xr-x 4 steam steam 4.0K Feb  7 00:40  ..
drwxrwxr-x 2 steam steam 4.0K Feb  7 00:56  linux64
-rwxrwxr-x 1 steam steam 4.7K Feb  7 00:56  LinuxPlayer_s.debug
-rwxrwxr-x 1 steam steam    2 Feb  7 00:56  server_exit.drp
-rwxrwxr-x 1 steam steam  716 Feb  7 00:56  start_server.sh
-rwxrwxr-x 1 steam steam   34 Feb  7 00:56  start_server_xterm.sh
-rwxrwxr-x 1 steam steam    7 Feb  7 00:56  steam_appid.txt
drwxrwxr-x 5 steam steam 4.0K Feb  7 00:56  steamapps
-rwxrwxr-x 1 steam steam  29M Feb  7 00:56  steamclient.so
-rwxrwxr-x 1 steam steam 6.8M Feb  7 00:56  UnityPlayer_s.debug
-rwxrwxr-x 1 steam steam  29M Feb  7 00:56  UnityPlayer.so
-rwxrwxr-x 1 steam steam 119K Feb  7 00:56 'Valheim Dedicated Server Manual.pdf'
drwxrwxr-x 6 steam steam 4.0K Feb  7 00:56  valheim_server_Data
-rwxrwxr-x 1 steam steam 6.2K Feb  7 00:56  valheim_server.x86_64
```
Now all you need to do is modify the file `start_server.sh` which contains:
```bash
export templdpath=$LD_LIBRARY_PATH
export LD_LIBRARY_PATH=./linux64:$LD_LIBRARY_PATH
export SteamAppId=892970

# Tip: Make a local copy of this script to avoid it being overwritten by steam.
# NOTE: Minimum password length is 5 characters & Password cant be in the server name.
# NOTE: You need to make sure the ports 2456-2458 is being forwarded to your server through your local router & firewall.
./valheim_server.x86_64 -name "My server" -port 2456 -world "Dedicated" -password "secret" -public 1 > /dev/null &

export LD_LIBRARY_PATH=$templdpath

echo "Server started"
echo ""
read -p "Press RETURN to stop server"
echo 1 > server_exit.drp

echo "Server exit signal set"
echo "You can now close this terminal"
```
Replace using a text editor(anything from `vim`, `nano` etc. for Ubuntu Server and you can use `gedit` for Ubuntu Desktop):
- `My server` with your server name;
- `Dedicated` with the name of your world file database (Optional - not required);
- `secret` with your password that needs to be at least 5 characters long.
Save the file and now you can start your server:
```bash
./start_server.sh
```
Hit `RETURN`(also known as `Enter` key) if you want to stop the server.
This is fine to get started as quick as possible, but I like to build a service around this just in case my server crashes and I want better control than just using the same command over and over again, so if you are interested, you might want to check `Step 7`.

## Step 7 - Creating a service(Optional)
It's optional because you are free to chose whatever option you are comfortable to use.
THIS is my comfort zone, so let's begin.
As steam user, create the following files:
- `/home/steam/start_valheim_server.sh` with content:
```bash
#!/bin/bash

# Tip: Make a local copy of this script to avoid it being overwritten by steam.
# NOTE: Minimum password length is 5 characters & Password cant be in the server name.
# NOTE: You need to make sure the ports 2456-2458 is being forwarded to your server through your local router & firewall.
./home/steam/valheim/valheim_server.x86_64 -name "My server" -port 2456 -world "Dedicated" -password "secret" -public 1
```
Same rules apply for `My server`, `Dedicated`,`secret` as described in `Step 6`.

- `/home/steam/stop_valheim_server.sh` with content:
```bash
#!/bin/bash

echo 1 > /home/steam/valheim/server_exit.drp
```
Make them executable:
```bash
chmod a+x *_valheim_server.sh
```
As your sudo user, create a file in `/lib/systemd/system/valheim-server.service` , for example with `vim`:
```bash
sudo vim /lib/systemd/system/valheim-server.service
```
Press `i` and insert the content:
```bash
[Unit]
Description=Valheim Server Service
[Service]
Environment="LD_LIBRARY_PATH=/home/steam/valheim/linux64:$LD_LIBRARY_PATH"
Environment="SteamAppId=892970"
ExecStart=/bin/bash /home/steam/start_valheim_server.sh
ExecStop=/bin/bash /home/steam/stop_valheim_server.sh
User=steam
Restart=always
[Install]
WantedBy=multi-user.target
```
Enable the service:
```bash
sudo systemctl enable valheim-server.service
```
Now you can start the service:
```bash
sudo systemctl start valheim-server.service
```
An voila! You service is now working, and it will also work on reboot. Just give it a test and see if it works.
If you want to manually restart it, just run:
```bash
sudo systemctl restart valheim-server.service
```
You can also check logs, useful for developers, In the below example I'll display the last 100 lines of server logs:
```bash
 journalctl --unit=valheim-server.service -n 100
```

## Step 8 - Connecting to your server
For this step you might want to get your external IP address. I usually use [SpeedTest](https://www.speedtest.net/) to get my IP as fast as possible, but you can use any method you like.
You can use a service like [CHECK-HOST](https://check-host.net/check-udp?lang=en) and enter `<your_ip>:2457` and test if it gives you `Open or filtered`, if so, then you are good to go.
On your Steam Client, go to `View > Servers > FAVORITES` and click `ADD A SERVER` and add `<your_ip>:2457` then click `ADD THIS SERVER TO FAVORITES`, then it will appear in your list. Click connect and enter the password that you configured in the previous steps(`Step 6 - Step 7` the field `secret`), after that your game will start automatically then click `Start` in game, it will ask you for the password again and it should start your session.

# THE END!



