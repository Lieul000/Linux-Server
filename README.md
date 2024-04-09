# Linux-Server

Project context
The local library in your little town has no funding for Windows licenses so the director is considering Linux. Some users are sceptical and ask for a demo. The local IT company where you work is taking up the project and you are in charge of setting up a server and a workstation. To demonstrate this setup, you will use virtual machines and an internal virtual network (your DHCP must not interfere with the LAN).

You may propose any additional functionality you consider interesting.

Must Have
Set up the following Linux infrastructure:

One server (no GUI) running the following services:

DHCP (one scope serving the local internal network) isc-dhcp-server
DNS (resolve internal resources, a redirector is used for external resources) bind
HTTP+ mariadb (internal website running GLPI)
Required
Weekly backup the configuration files for each service into one single compressed archive
The server is remotely manageable (SSH)
Optional
Backups are placed on a partition located on separate disk, this partition must be mounted for the backup, then unmounted
One workstation running a desktop environment and the following apps:

LibreOffice
Gimp
Mullvad browser
Required
This workstation uses automatic addressing
The /home folder is located on a separate partition, same disk
Optional
Propose and implement a solution to remotely help a user
