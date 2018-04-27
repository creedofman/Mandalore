# The Big Picture #

*Disclaimer: This is what works for me. There's a good chance that with minimal changes this will also work for you, but I made this for me, so don't complain that this doesn't work for you. On the other hand, I would love to answer questions and get feedback on any of my code - feel free to reach out/make a pull request, etc!*


### The Hardware

- Asus Q87M-E motherboard
- Intel Core i5-4570
- 16GB RAM (non-ECC)
- 2x 256GB SSDs
	- 1 for boot, 1 for temp files (in-progress downloads, etc)
- 3x 3TB + 1x 4TB HDDs internal
- External Mediasonic ProRAID 4-bay DAS, with 4x3TB in RAID5

### The Software

- Ubuntu Server 16.04 LTS (fully patched, kept updated bi-weekly)
- Docker-CE (current stable version, kept updated bi-weekly)
	- Plex (https://plex.tv/)
	- Sonarr (https://sonarr.tv/)
	- Radarr (https://radarr.video/)
	- Transmission (https://transmissionbt.com/)
	- NZBGet (https://nzbget.net/)
	- Ombi (https://ombi.io/)
	- Tautulli (http://tautulli.com/)
	- Portainer (https://portainer.io/)
- BTRFS (to create a 22TB storage array)

### Other Notes

- Sonarr and Radarr require one or more "indexers" and "trackers" to find new content. I'm not going to get into what those are or how to find/join them. Check out /r/trackers or /r/indexers on Reddit.
- All of the above services can run without Docker - however: I like Docker, I like how it works, I think it's better than running all those services independently. Docker has a learning curve, but once you grasp the key concepts, it's great to work with. I recommend checking out the "Get Started" docs here: https://docs.docker.com/get-started/
- I'm still fine-tuning my own server, and I'm already building my next (replacement) upgrade. I started out with a Raspberry Pi 3 and some external drives, and currently have ended up here.
- In addition to all the hardware/software listed above, I also have a spare machine set up as an Automated Ripping Machine with minor script customizations. I did not write the original scripts, to learn more about ARM go here: https://b3n.org/automatic-ripping-machine/
	- A.R.M. lets me pop in a disc (cd/dvd/blu-ray), wait a little while, then it pops the disc back out. Every night a cron job rsyncs the newly ripped files to a directory on my main media server machine.
	
---

# Basic Configuration Steps

1) Install Ubuntu Server 16.04 LTS
	- Install on SSD, use the whole disk, let the Ubuntu installer automatically create all the default partitions.
2) Configure storage drives
	- I like BTRFS for my current hardware, but it's not important.
	- Configure the `/etc/fstab` so your drives mount automatically on boot to a directory
		- I mount my drives to `/mnt`, but whatever. 
3) Create necessary directories
	- I keep my docker container configs in `/home/$user/docker`
		- e.g. `/home/$user/docker/plex`, `/home/$user/docker/sonarr`, etc.
	- I have a `downloads` directory for completed downloads on my main (22TB) array, and a `downloads` directory on my second SSD for incomplete/in progress downloads
	- I suggest a "Plex" directory, with sub-directories for Moves and TV Shows and whatever other media you'll want in Plex.
		- e.g. I have `/mnt/internal/Plex-Mandalore` with `/mnt/internal/Plex-Mandalore/Movies/` and `/mnt/internal/Plex-Mandalore/TV Shows/` as well as some other directories.
	- Make sure that your user owns all of the directories that you've created, and set `770` or `774` permissions (`chmod +x 770 /path/to/directory`)
4) Install Docker-CE
	- Follow the instructions here: https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-docker-ce-1
5) Start building your Docker containers
	- I'm working on my docker-compose.yml, and I'll have one in this repo eventually, but I don't have it yet.
	- LinuxServer.io has great containers for most of the applications
	- All of the docker containers (except for Portainer) need to have the `PUID` and `PGID` environment variables set to the same value as your main user (the one that has permissions set on the directories)
		- This is super important, otherwise your applications will not be able to work with the media files properly
6) Configure all of your containers to point to the correct directories
	- Sonarr, Radarr, Transmission, and NZBGet all need volume mounts to your complete downloads directory and incomplete downloads directory.
		- e.g. `-v /mnt/internal/downloads:/downloads/complete` and `-v /mnt/scratch_ssd/downloads:/downloads/incomplete` for each container. Once I put Dockerfiles or docker-compose.yml in this repo, you can refer to that for specific configs.
	- Plex needs a volume mount to your `Plex` directory, but not to the download directories
	- Sonarr and Radarr need a volume mount to your `Plex` directory
	- Most containers need a `/config` directory mounted - I keep all of mine in `/home/$user/docker/$containername/config`
		- e.g. `/home/$user/docker/sonarr/config`, `/home/$user/docker/radarr/config`, etc.
		- Plex works a bit differently, I specified the config directory as well as a location on my non-OS SSD for the metadata files (so that the libraries load faster in apps/web interface)
	- Portainer isn't necessary, but it makes restarting and updating the containers easier, as well as making some minor adjustments to the container configs if necessary. Also gives a nice visible reference to what is actually on your machine.
7) If you have existing media to migrate, I highly recommend doing so utilizing Sonarr and Radarr.
	- I'm not going to explain how to configure those, check out their websites as well as /r/sonarr and /r/radarr on Reddit for good reference and to ask questions
8) Use Ombi to search for new media to add to your library!
	- Once everything is set up how you want it, you can make Ombi publicly accessible - meaning you can pull it up on your phone or computer from anywhere, search for the movie/tv show you want, add it to your request list, and then let Sonarr/Radarr handle searching your indexers/trackers, sending the download info to Transmission/NZBGet, and then renaming and sorting it into the right directory for Plex to catalog it. By the time you get home, you'll have some new media to watch!
	
---

I probably won't make many changes to the above, but I am planning to start building out this repo with configuration steps for all of the applications I use, along with Dockerfiles/docker-compose.yml for my setup. If I get really ambitious I may create a script to do it all "automagically", however the group over at https://plexguide.com/ already have a pretty good thing going. We'll see.
