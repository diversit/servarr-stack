# Customized *arr stack setup

Based on the [Youtube video](https://youtu.be/1eqPmDvMjLY?feature=shared) and repository 'youtube-39-arr-apps-1-click'.
Thanks to 'Automation Avenue'.

Customized his setup by:
- Using a PostgreSQL database (on other LVM).
- Adding more services: Bazarr, Tautulli, Homarr, Overseerr. Disabled Readarr and removed JellyFin since I'm using Plex.
- Added a [Caddy Server](https://caddyserver.com) instance to automatically have HTTPS on local domain and allows to access *arr service on a hostname instead of having to remember the ports.

I'm running this setup on an 'old' MacMini running El Capitan (OSX 10.11, Intel Core 2 Due, 8GB memory) which is also running Plex.
Since this OS does not support the latest docker, it runs an Ubuntu 24.10 in an old VirtualBox (v5.2) which was installed on that machine.
Files are stored on a external USB drive which folders are shared with the VirtualBox VM. This requires the guest-additions to be installed on the guest VM!

Note that the guest-additions for the VirtualBox 5.2 do NOT work with Ubuntu 24.10. Luckily the latest guest-additions (v7.1) _do_ work with Ubuntu 24.10 and are also compatible with the VirtualBox 5.2 host allowing the host folders to be mapped to the VM.

The VM is configured to use 1 CPU and 4Gb memory and a network in 'bridge' mode allowing it to get it's own ip address.
Both the MacMini and VM are put in a special VLAN only allowing access to the internet and no further access to any other device on the VLAN or any other VLAN in my LAN (except for the PostgreSQL instance which is a LVM on the same VLAN) since I do not fully trust all these applications.
I encourage you to do the same.

The setup keeps the VM and host quite busy. It would run smoother on a large machine, but it's quite acceptable.
The videos streamed by Plex are fluent.

For monitoring, I setup a dashboard in Homarr allowing to see whether all services are up and easy buttons to access each service.
It also displayes lists of qBitTorrent and Overseerr.
The Homarr dashboard is added as a 'website' dashboard in my Home Assistant for easy access.
Also, Home Assistant now has support for Radarr, Sonarr and Overseerr via integrations and these have been added to HA for additional metrics directly in HA, to be used in a HA dashboard later.

The Caddy server is great for creating a HTTPS reverse proxy to all the *arr services.
In my local DNS an A record has been added for the '*.servarr.home' domain to map to the IP address of the VM.
Since Caddy does not support the '.home' domain natively as a 'local' domain, each entry in the `Caddyfile` required a `tls internal` setting to force Caddy to use its own local CA.
The root certificate can easily be exported and added as 'trusted' CA to tbe browser, or for Safari, to the Key Chain.
 
# youtube-39-arr-apps-1-click
Video 39 - Deploy ARR apps using just 1 command (full set with Jellyfin and qBittorrent !!!)

### Useful Links:
- [Servarr Wiki](https://wiki.servarr.com/)
- [Trash Guides](https://trash-guides.info/)
- [Ascii ART](https://patorjk.com/software/taag/#p=display&f=ANSI%20Shadow)

### Download and unzip Files from GitHub:
https://github.com/automation-avenue/youtube-39-arr-apps-1-click <br />
cd /home/marek/Downloads <br />
unzip youtube-39-arr-apps-1-click <br />

### Installation process:
Make sure you are in the same folder as docker-compose.yml and .env file, then 'up' to deploy, 'stop' and 'rm' to stop and remove the stack  :<br />

```bash
sudo docker-compose up -d 
sudo docker-compose stop
sudo docker-compose rm 
```

Go to the folder specified in .env file (if its /media/Arr then go to /media as root) and 
run chown command with the user id and group id configured in that .env file:<br />
`chown -R 1000:1000 Arr`<br />
Now you can log on and work with all services.<br />

First configure the qBittorrent service because its using temporary password only:<br />

**qBittorrent:**<br />
First - find the qbittorrent container id by running:<br />
`sudo docker ps`<br />
Then check logs for that container it:<br />
`sudo docker logs <qbittorrent-container-id>`<br />
You will see in the logs something like:<br />
*The WebUI administrator username is: admin<br />
The WebUI administrator password was not set. A temporary password is provided for this session: <your-password-will-be-here>* <br />
Now you can go to URL:<br />
http://localhost:8080<br />
and log on using details provided in container logs.<br />
Go to Tools - Options - WebUI - change the user and password and tick 'bypass authentication for clients on localhost' .<br />

Then first configure Prowlarr service (each of these services will require to set up user/pass):<br />

**Prowlarr:**<br />
http://localhost:9696<br />
Go to Settings - Download Clients - `+` symbol - Add download client - choose qBittorrent (unless you decided touse different download client)<br />
Put the port id matching the WebUI in docker-compose for qBittorrent (default is 8080) and username and password that you configured for qBittorrent in previous step<br />
Host - you have to change from localhost to ip address of the host machine (run 'ip address' command on your host system)<br />

**Sonarr:**<br />
http://localhost:8989<br />
Go to Settings - Media Management - Add Root Folder - set /data/tvshows as your root folder<br />
Go to Settings - Download Clients - click `+` symbol - choose qBittorrent and repeat the steps from Prowlarr.<br />
(there are also 'Remote Path Mappings' - use only if your qBittorrent and ARR stack are on different hosts / systems)<br />
Go to Settings - General - scroll down to API key - copy - go to Prowlarr - Settings - Apps -click '+' - Sonarr - paste  API key and change 'localhost' to ip address of the Ubuntu/Host again.<br />
Then Settings - General - switch to 'show advanced' in top left corner - scroll down to 'Backups' and choose /data/Backup (or whatever location you have in your docker compose file for Sonarr backups )<br />

**Radarr:**<br />
http://localhost:7878<br />
Go to Settings - Media Management - Add Root Folder - set  /data/movies as your root folder <br />
Then Settings- Download clients - click 'plus' symbol, choose qBittorrent etc - basically same steps as for Sonarr<br />
Settings - General - scroll down to API key - copy - go to Prowlarr - add same way as in sonarr<br />
Settings - General - switch to 'show advanced'- Backups - choose /data/Backup folder <br />

**Lidarr:**<br />
http://localhost:8686<br />
Follow the same steps for Lidarr and Readarr as for above applications.<br />

**Readarr:**<br />
http://localhost:8787<br />

**Homarr:**<br />
http://localhost:7575<br />

Now go back to Prowlarr and click 'Indexers at the top right, click 'Add indexer' - search for sth like 'rarbg' or 'yts' etc then test - save<br />
Then click 'Sync App Indexers  icon (next to 'Add indexer')<br />
If you go to Settings - Apps - you should see green 'Full sync' next to each application.<br />
Arr stack completed - you can now 'add movie' in radarr or 'add series' in sonarr etc and click 'search all' or 'search monitored' - that will trigger the download process.<br />

**Jellyfin:**<br />
http://localhost:8096<br />
If you run `docker-compose up` and have something running on port 1900 -  its most possibly rygel service, run:<br />
`sudo apt-get remove rygel` and run the `sudo docker-compose up -d` again.<br />
Then add media library in Jellyfin  matching folders configured in docker-compose.yml file, so in Jellyfin you should see them as: <br />
/data/Movies <br />
/data/TVShows <br />
/data/Music <br />
/data/Books <br />
