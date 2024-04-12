# LINUX SERVER PROJECT

### WORKSTATION
In order to realize the project, we have to create a VM with a linux distro first (in order to create a workstation). I chose debian. 

- Find an iso of the latest debian and create a new vm (allocating as usual at least 2048MB of RAM, 2 CPU and 30 GO hard drive.
- Install debian through the advanced options and do not forget to choose to separate the /home partition from the hard driv.
- Don’t extra scan installation media, select your country mirror (deb.debian.org) and untick the proxy
- Don’t forget to choose SSH on the software selection and to separate /home directory in his own partition

Then we have to set up the VM :

Add your user to the sudoers

``su root
sudo usermod -aG sudo YOUR_USERNAME
exit
``

Restart your VM


Let's add all the usual applications : 
##### Libreoffice (usually already included in debian distro)

``sudo apt install libreoffice``

##### GIMP (to easily manipulate image)

``sudo apt install gimp``

##### MULLVAD Browser (to navigate)
Download from MULLVAD WEBSITE then

``
cd Downloads
tar xf mullvad-browser-linux64-[YOUR_VERSION].tar.xf
rm mullvad-browser-linux64-[YOUR_VERSION].tar.xf
mv mullvad-browser ~/
cd
cd mullva./start-mullvad-browser-desktop --register-appd-browser
``

##### Then finally, let’s set a remote help with XRDP :

``
sudo apt update
sudo apt install xrdp
sudo systemctl status xrdp
``

to enable it if is not running : 

``sudo systemctl enable --now xrdp``

add the xrdp ussr to the ssl-cert group

``sudo adduser xrdp ssl-cert``

restart xrdp server : 

``sudo systemctl restart xrdp``


##### Remmina to be able to help from a another workstation :

``sudo apt install remmina``

##### UFW to set a firewall :

``
sudo apt install remmina
sudo ufw allow 3389
sudo ufw enable
``

TIPS : don’t forget to check your workstation ip :

``ip a``


### SERVER

The server VM creation and setup are pretty much the same as the Workstation's ones with one exception, deselect any desktop environment and select SSH and Web Server.


SSH daemon : The ssh server was installed during the Debian installation process. 

TROUBLESHOOT : 
reinstall sudo 
use the command su to access to root
trying to apt-get install sudo (but did not work cause CLI is asking to put the dvd)
going to the source list files to comment the line the the dvd name for the cli to ignore it

use the command

``cd /etc ``
``ls``
``cd apt`` 
``ls``
``nano sources.list``

add a # in front of the line mentioning the DVD name

(permission non accordé donc) 
``sudo chmod 666 /etc/apt/sources.list``

Add your user to the sudoers

``
su root
sudo usermod -aG sudo YOUR_USERNAME
``

##### ##### WEEKLY BACKUP

First we need to create a new volume for your VM.

Shutdown your VM.
In VirtualBox, while your VM is selected, go to Settings.
Go to storage.
Click on the little hard drive on the left of "Controller:SATA".
Create on the top left.
Leave VDI selected -> Next.
Allocate the space you want (at least 5GB).
Select your new volume and click on "Choose".
You now have a new volume attached to your VM, to check your available disks on the VM, type:

`lsblk`

If you see a /dev/sdb then your additional drive exists.

Format the new extra drive to ext4:
`sudo mkfs.ext4 /dev/sdb`

Create a backup directory in /mnt:
`sudo mkdir /mnt/conf_backups`


Fun part; writing a script to mount our disk and backup our configuration files before unmounting it:

``
su root
cd
mkdir scripts
nano /root/scripts/conf_backup.sh
``

Paste in:

``
sudo mount /dev/sdb /mnt/conf_backups/
sudo mkdir /tmp/$(date +%d-%b-%Y)
sudo cp -r /etc/bind /etc/dhcp /etc/apache2 /etc/mysql /etc/php /etc/resolv.conf /tmp/$(date +%d-%b-%Y)/
sudo tar -zcvf /mnt/conf_backups/$(date +%d-%b-%Y).tar.gz /tmp/$(date +%d-%b-%Y)/
sudo rm -rf /tmp/$(date +%d-%b-%Y)
sudo umount /dev/sdb
``

Then we create a cronjob to launch the script every week on sunday at 9PM.

``sudo crontab -u root -e``
``00 21 * * 7 /root/scripts/conf_backup.sh``
           
           
##### DHCP

Restart your VM

DHCP service through KEA (a more modern and still maintained service): 
https://www.youtube.com/watch?v=FGw06CSLizY

I use this tuto but change the code inside the nano kea-dhcp4.conf files.

1- Install KEA : 
go to the the website https://kb.isc.org/docs/kea-build-on-debian 

and do not forget to check for upgrade https://kb.isc.org/docs/upgrading-packages-beyond-kea-232


2- Installe dependencies
use the command
``apt install curl apt-transport-https -y``

3- Add a repo (to make sure we use the latest version and not what the OS knows about, the recommendation is to use the ISC repositories of the current stable version)

use the command
``curl -1sLf 'https://dl.cloudsmith.io/public/isc/kea-2-2/setup.deb.sh' | sudo -E bash``


4- Install DHCPv4 Server

WARNING: Installing a DHCP server into an existing network where DHCP already exists will lead to problems so it’s best to do this in a new subnet

To install an IPv4 DHCP server use the commands

``apt install isc-kea-dhcp4-server -y``

5- Configure the DHCP4 Server

use the command
``cd /etc/kea``

Each package has it’s own configuration file and the IPv4 configuration is held in the kea-dhcp4.conf file
The existing file isn’t practical to administer so we’ll back this one up and use it for reference

use the command
``mv kea-dhcp4.conf kea-dhcp4.conf.bak``

Then create your own by using the command 
``nano kea-dhcp4.conf``

Here is the script to put in nano in order to configure the server

````
{
    # DHCPv4 configuration starts on the next line
    "Dhcp4": {
 
        # Next we set up the interfaces to be used by the server.
        "interfaces-config": {
            "interfaces": [ "enp0s3/192.168.1.63" ]
        },

      
        # Finally, we list the subnets from which we will be leasing addresses.
        "subnet4": [
            {
                "id": 1,
                "subnet": "192.168.1.0/24",
                "pools": [
                    {
                        "pool": "192.168.1.2 - 192.168.1.253"
                    }
                ]
            }
        ],

       
        # We specify the default gateway and DNS server IP address
        "option-data":[
            {
                "name": "routers",
                "data": "192.168.1.1"
            },
            {
                "name": "domain-name-servers",
                "data": "192.168.1.254"
            },
        ]
    }
}

`````


##### GLPI + MariaDB
I follow this tuto (in french)
https://www.it-connect.fr/installation-pas-a-pas-de-glpi-10-sur-debian-12/


##### DNS

I follow this tutorial : https://www.youtube.com/watch?v=FGw06CSLizY

and BIND9 : https://www.youtube.com/watch?v=rl3bVXEWuuw

##### FIREWALL

We will install uncomplicated firewall by using the command:

```````
sudo apt install ufw
sudo ufw enable
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 53/tcp
sudo ufw allow 22/tcp
(80 for HTTP, 443 for HTTPS, 53 for Domain, and 22 for SSH)

``````
