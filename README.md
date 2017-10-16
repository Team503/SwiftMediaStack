# SwiftMediaStack
This is a project to provide tutorial and instructions to setting up a full media server centered around Plex's closed source player, utilizing Ubuntu LTS and Dockers.

# What the Hell Is This?

A fully functional Plex environment, including automatic content downloading and library maintenance, as well as automated handling of user requests for content.  Features:

* Plex Media Server
* Automated downloading and file handling of movies, television shows, and music via Usenet and Torrents
* User-friendly GUI for viewers to request new content and report issues
* Reverse proxy server with SSL to allow external access

# What's Actually In These Instructions?

You'll be setting up the following applications:

First you'll install Docker Project's Docker Community Edition, which you'll need to host all the following Dockers:

* Plex
* PlexPy for usage statistics
* Ombi for user requests and newsletter functions
* Radarr for movie tracking
* Sonarr for television tracking
* Headphones for music tracking
* Deluge for torrenting
* NZBget for Usenet groups
* Hydra and Jackett for additional indexer support
* MusicBrainz for indexing music to support headphones

Running standalone is:

* Nginx for reverse proxy and certbot for Let's Encrypt support

The instructions all are designed to run with Ubuntu 16.04 LTS, but can probably work with most any distro with only minor changes.

# Uh, okay, so... Why did you do -blank- this specific way?

I wrote these with /r/homelab users in mind, and so I assume that anyone reading this is using ESX, KVM, ProxMox, or even VirtualBox.  So I assume that standing up VMs is quick, easy, and relatively resource light.  Thus I designed this to run across three separate Linux boxes (and read files from Windows file servers).  You can, if you so desire, combine these all together into one media solution.  I don't for a few reasons.  

First and foremost among them is that I believe in layered stability.  Similar in concept to perimeter networking (in which you establish security in layers, rather than just one single large firewall), layered stability means that I am utilizing multiple methods to insure uptime.  In this case, I start from the hardware layer and go up.  My hardware itself is redundant (I have multiple servers and redundant drive arrays because I'm *that* kind of nerd).  Virtual machines make sure my operating system continues to run in case of hardware failure (I utilize VMware ESX with the VMUG license to enable HA and DRS between my pair of hosts).  At the application level, I separate each application into its own virtual container with Docker.

What's the end result?  If a container fails, it doesn't take all the others with it.  If the Docker-Engine fails, it doesn't take the OS with it.  If the OS fails, it doesn't take the other virtual machines with it.  If the hardware fails, the VMs are vMotioned to good hardware.  I'm as protect as I can be.

Second, this allows me to control both load and connection states.  I install MusicBrainz to its own server because the single most common problem with MusicBrianz servers is that they get hammered to all hell and oblivion by a million and ten requests every microsecond when they're public.  My server *is*, so I am careful to segment it off so if it is overwhelmed, it won't take anything else down with it.  Additionally - and more importantly for most of you - is that on my seedbox, I can install OpenVPN (or another VPN client) at the OS level, and all the Dockers will have no choice but to connect via a single VPN connection.  Can you do this with OS native level applications?  Sure.  But it makes me feel better.  It also lets me run the VPN connection on the seedbox *only*, which means I don't have to force all my Plex traffic to bounce all over the world through other people's slower pipes, and allows me to make full use of my home connection, which happens to be synchronous gigabit.

Third, it's super easy to version things!  I don't have to worry about anything if I want to run an upgrade - I just pull the new image, stop the Docker, delete it, and recreate it.  Which for me, is as simple as typing "bash upgrade-dockers.sh", or copy/pasting from a text file into a PuTTy window.

Fourth, honestly, as someone new to Linux I just wanted to play with Dockers.

And finally, fifth, Deluge is a flopping piece of sh#t that crashes constantly when it has more than a few hundred torrents running, and I cannot count how many times I've had to stop the Docker, rm session.state, and restart the Docker.  Vastly easier than other forms of troubleshooting it, both on Windows AND on Linux.

# Why did you choose to use external file servers, especially Windows ones?

Three main reasons, really.  One, that I believe storage should stand alone, siloed and inviolate, unsullied by the brazen code that calls itself an application.  Just on general principle.  And also because my NAS may be homebuilt, but it's not at all unlike the petabytes of storage in my office datacenter.

Second, I utilize many applications which access those files, some Active Directory aware, some LDAP aware, and some not at all.  I find it much easier to use AD's nested grouping and permissions structures than Linux's kludgy half-ass attempts at LDAP implementation.

Third, I think a lot of people out there are likely to have a Windows machine they want to play content from, and this way, it's already covered for them.

Oh, and even though I didn't say four, number four is because I *already had them built*, and with 40TB of storage, rebuilding them isn't a small task.

# Right, you're stupid, you should've used so-and-so's setup or done it another way or blah

Honestly, that's great for you, but I did it this way for well thought out reasons that I just explained.  Other people's YMLs used CouchPotato instead of Radarr, or some such, or didn't include another bit I wanted, or had something I didn't.

Also, I like using instructions so that you gals and guys can see what's happening, line by line, and perhaps gain an understanding of it.  Anyone can run "install program instructions.list" - trying to learn and understand it by going through each line yourself makes you a better user/admin/nerd/whatever.

This isn't likely to be the final form of these scripts, either.

# What's Coming in the Future?

I'll be adding instructions to build an NGINX reverse proxy to allow external traffic in.  Once I get a hang of LetsEncrypt and am able to deploy it repeatedly without fail, I'll add that.  When Lidarr (a replacement for the venerable but aging Headphones) makes it into at least beta, I'll include that too.

I'll definitely add installing OpenVPN on the seedbox soon, too.

I may even eventually expand this project to be a fully functional home production environment, to include things like Pydio or MatterMost or some such, I haven't decided.

# So What Do I ACTUALLY DO

* Stand up four Ubuntu 16.04 LTS virtual machines
* Configure each with an encrypted home directory
* Set up a user with the appropriate permissions (if you're lazy and don't mind being insecure, just use the one created during the Ubuntu install process)

I recommend the following specs for each VM.  This is variable - you can add or remove CPU as appropriate for your needs.

*MusicBrainz* - 2CPU 1GB RAM 20gb HDD  
*Plex* - 4CPU 4GB RAM 20gb HDD + enough space for your library cache (varies wildly depending on size of library)  
*Seedbox* - 2CPU 8GB RAM 20gb HDD + a dedicated, single platter, nothing-else-uses-it scratch disk of any size (note that Deluge will absolutely shit itself if this disk is full, so mind your settings!).  This guy gets RAM because he does a LOT at once, especially during release heavy days.  
*PROXY* - 1CPU 1GB RAM 16gb HDD

If you're using ESX, I strongly recommend:

* Configure the hard drives with VMware Paravirtual controllers
* Use the VMXNet network adaptor for best performance
* Place all these machines on the same vSwitch and dvSwitch (if you use them) to keep routing and traffic to a minimum
* The scratch disk for the seed box should be physically installed in the host machine that seedbox will usually run on

# Generic Responses to Comments

* Will sign up for a Github and do that soon - maybe today or tomorrow
* Will update with OpenVPN instructions soon
* Taking requests for additional stuff
* Moving NGINX to a Docker is my next trick

# Special Thanks

* Plex for publishing a Docker
* Linuxserver.io for all their Dockers!
* /r/homelab for being my reddit place of learning and support
* /r/plex for being critical and honest and helping me improve
