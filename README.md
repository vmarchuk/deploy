Deploy is an enterprise deployment via Bittorrent.  Designed for fast transfer of large amount of data over many hosts.

This script is an improved deployment over twitter's murder(https://github.com/lg/murder) and Herd(https://github.com/russss/Herd).
Its designed to be self contained, easy to use and many other features.

Features:
* Written in perl
* Separated network support, ie: (dmzA1 <--> dmzA2) <--> ndn <--> (dmzB2 <--> dmzB1)
* Uses compression for faster transfer
* Designed to look like a filesystem
* Allows for wrapper scripts
* No neeed to worry about seeding or distribution of torrents, deploy is completely self contained

Requirements:
* Transmission - http://www.transmissionbt.com
* Opentracker - https://erdgeist.org/arts/software/opentracker
* Rsync - http://rsync.samba.org
* Zip - http://www.info-zip.org

Technical Details:
```
  Deploy uses several programs: transmission, opentracker, rsync and zip. Instead of 
  reinventing the wheel, deploy is using 'transmission' as its torrent program.  
  Opentracker is used as a lightweight tracker.  
  Rsync is used for distribution of torrent files.
  Zip is used for compressing data to speedup overall process.

  There are 2 modes in deploy: regular or tracker.  Tracker's job is not only act as 
  torrent's tracker but also a seeder. Regular host will have the main deploy daemon 
  and transmission processes running. On a tracker, additional processes will be 
  running as well: rsync and opentracker.

  A torrent can be deployed from anywhere in the network by running, deploy add "name".  
  The "name" corrents to directory name under DEPLOY, ie: /usr/local/deploy/DEPLOY.
  At that point a zip file is created in DEPLOY/.deploy/base/name.VERSION/ and a 
  torrent created in DEPLOY/queue/.  Local hosts transmission process is told to start 
  seeding.  Tracker is contacted next via rsync.  The rest of the hosts on the network 
  connect to tracker every 15 seconds(default).  When these hosts contact tracker, 
  they will notice that a new torrent is created, will download the torrent file and 
  have their local transmission start the torrent download.
```

Configuration details:
  Default location is /usr/local/deploy.  
```
  Main configuration file (written as perl include): etc/deploy-system.conf
	APP_BASE: main location where all data lives, its a good idea for this location 
                  to have plenty of disk space
	LOGDIR: location where all log files are kept
	LOGDAYS: maximum number of days for logs to be kept
	@TRACKERS: set a list of trackers, if you are going to use this on split network 
                   then comment out this line and use lib/hooks/tracker.sh
		   Separated network support diagram example: 
                       (dmzA1 <--> dmzA2) <--> ndn <--> (dmzB2 <--> dmzB1)
		   tracker.sh is a shell script that you need to modify to return 
                   different trackers based on network location
	PORT_rsync: port for rsync process
	PORT_opentracker: port for opentracker process
	PORT_transmission: port for transmission process
	API: password key, this needs to be changed as its used as authentication
	%REQUIRED: a hash where all you can provide paths of all required programs

  Torrent control: etc/deploy.conf
  Control of torrents 
  	MAIN.onlyInDeploy: if set to true it will only allow torrents that are listed 
                           below.
	DEPLOY.name.hosts: a list of hosts that can download the specific torrent.  In 
                           this instance the torrent's name is 'name'.
		           If the value is empty then all hosts are allowed to work 
                           with this torrent.  If you set 'onlyInDeploy' above then 
		           you need to have this config line.
		           It is a good idea to include trackers in this list.  Trackers 
                           can be dynamically included by using $TRACKERS value
	DEPLOY.name.exclude: a list of hosts that may not download this specific torrent
	
  Transmission (used as template during startup): etc/transmision.conf
  	This file should not be touched unless you know what you are doing.
  Rsync (used as template during startup): etc/rsyncd.conf
  	This file should not be touched unless you know what you are doing.
  Opentracker (used as template during startup): etc/opentracker.conf
  	This file should not be touched unless you know what you are doing.
```

Install and Usage:
```
  Install deploy in /usr/local/deploy.
  Make sure you have firewall holes for ports: 6998-7000.  These ports are configurable 
    in etc/deploy-system.conf.
  Modify /usr/local/deploy/etc/deploy-system.conf and set @TRACKERS with hostnames of 
    hosts responsible for seeding, ie: your depot hosts
  Start deploy on all hosts including tracker hosts by running:
    '/usr/local/deploy/bin/deploy start'
  Make sure its running by running: 'deploy status'
  ie: your data is located in /var/foo/bar.  To deploy this data, on any one host run:
    'deploy add bar from /var/foo/bar/'
  You can also use 'deploy add bar' if the latest 'bar' already exists in DEPLOY/bar
  Run 'deploy status' to check progress
  Data should start appearing in /usr/local/deploy/DEPLOY/bar
  If you need wrapper scripts around this, ie: check when the data has copied, you can 
    grep for 'versionid:done' in /usr/local/deploy/DEPLOY/bar/.deploy/state.
  'versionid' can be retrieved during 'deploy add' process

  To remove bar, run 'deploy delete bar'.
```
