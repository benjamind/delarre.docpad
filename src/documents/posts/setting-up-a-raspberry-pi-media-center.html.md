---
title: Setting up a Raspberry Pi media center
layout: post
tags: ['raspberry pi','raspi','weekend projects','xbmc','xbian','mopidy','spotimc','init.d','start-stop-daemon','media pc','htpc']
date: 03/02/2013
---
I finally got round to ordering a Raspberry Pi this week and this Friday it finally arrived. So this weekend I've been busy getting it setup to act as a media center for our living room. What I want ideally is a Xbox Media Center setup capable of playing 1080p media with DTS output, I then also want it to be capable of playing Spotify headless without having to use my TV as the screen. So really I want an Android client on my phone controlling spotify on the Raspberry Pi and XBMC for when I'm watching media on the TV.

### Xbian and XBMC Setup

I decided to use [Xbian](http://xbian.org/) since it comes with a nice minimalist pre-built image ready installed with XBMC and HDMI CEC support. Getting this installed was as easy as [downloading the installer](http://xbian.org/getting-started/install-and-connect/) and letting it do its thing to my 2gb SD Card. Simple. Within 10mins I had my XBMC installation up and running.

#### Remotes

Thankfully since Pulse Eight produced the [libCEC library](http://libcec.pulse-eight.com/) HDMI CEC control just works out of the box. So I immediately had my LG Magic Remote working with XBMC and was a very happy bunny. However, I have to say after some testing this really isn't the ideal way of controlling XBMC. Don't get me wrong HDMI CEC is very cool, and being able to startup the TV and 5.1 Receiver with XBMC is great, but the simple fact is that the buttons on the LG Magic Remote aren't ideal for this type of interface. If someone could get the pointer interface to control a mouse over HDMI then we'd be talking, but right now its too limited.

So back to my trusty Android devices to install the [Official XBMC Remote app](https://play.google.com/store/apps/details?id=org.xbmc.android.remote&hl=en). This app is great, a breeze to setup and its fast and snappy giving you good integration with your media center. The remote interface is a lot more comfortable to use than my LG Magic Remote, and the on-device browser for media is lovely (if a little slow with the images server from the Raspberry Pi).

#### Media Library

One thing that caught me out at first was that XBMC will not apply its library functions to media shares over UPnP which is a bit of a shame, so I've had to add the shares on my NAS to XBMC using smb shares. Once I did this though it happily chugged away indexing all my media and I was soon up and running.

Be warned, when XBMC is updating its library its basically unusable, the performance drops to multi-second responses, so if you've just added some new media to your shares just go get a cuppa or something and wait for it to do its thing.

### Spotify with Spotimc

Next up I wanted music in my living room, I had two options here, Spotimc which is a plugin for XBMC that was incredibly easy to setup thanks to the work of [welly_59 from the stmlabs forum](http://forum.stmlabs.com/showthread.php?pid=50586#pid50586), just (grab this file)[http://ge.tt/4RzFM9W/v/0?c], put it somewhere XBMC can reach it on your network then go to `System -> Addons -> Install from Zip` and browse to the file.. It won't show you much happening, but a short while later you'll get a little popup telling you the plugin was added. If you then go to `Music -> Addons` from the main menu you'll see Spotimc. On first start it will do some more installation and setup which will require a reboot, after that though you should be greeted with a login screen and be up and running.


### A better way - Mopidy

Thats all well and good, but I don't really want to have to put my TV on just to play music.

So I spent this morning looking into other alternatives. The best of which appears to be [Mopidy](http://www.mopidy.com/). Mopidy is a music server that will play Spotify streams as well as search and play local media. But best of all there's a variety of different mobile clients that will control it since its [MPD compatible](http://en.wikipedia.org/wiki/Music_Player_Daemon). This means once its setup you can control it from the comfort of your mobile and play music without the need to use your television!

Getting it setup was a little trickier than I had hoped though, first off there's no installation instructions for Xbian available. So I followed the rough outline of the [instructions for Debian Wheezy](http://docs.mopidy.com/en/latest/installation/raspberrypi/).

#### Installation

First install Mopidy and its dependencies from apt.mopidy.com, as described in the Wheezy installation documentation. In short:

```bash
	wget -q -O - http://apt.mopidy.com/mopidy.gpg | sudo apt-key add -
	sudo wget -q -O /etc/apt/sources.list.d/mopidy.list http://apt.mopidy.com/mopidy.list
	sudo apt-get update
	sudo apt-get install mopidy
```

Now install jackd1 since the new version jackd2 causes issues apparently:

```bash
	sudo apt-get install jackd1
```

Say 'yes' to allow installation with a realtime configuration.

Finally we want to ensure that all the relevant drivers are up and running, so lets modify our `/etc/modules` file to look something like this:

```bash
	snd-bcm2835
	bcm2708_wdog
	ipv6
```

#### Running on boot

This is where the basic installation instructions end and things got a little murky. I wanted to setup Mopidy to run automatically on startup, most instructions tell you to use upstart, but Xbian doesn't use this by default, it uses the old school init.d scripts. So lets setup a script to start Mopidy, and setup configuration for our Spotify account.

First lets add a user account for mopidy to run under:

```bash
	sudo adduser --system mopidy
	sudo adduser mopidy audio
```

Next we should create the required config files in the `/home/mopidy` folder for our new user:

```bash
	sudo mkdir /home/mopidy/.config
	sudo mkdir /home/mopidy/.config/mopidy

	vi ~/.config/mopidy/settings.py
```

Add something like this to your `settings.py` file:

```bash
	MPD_SERVER_HOSTNAME = u'::'
	MIXER = u'pulsemixer'
	SPOTIFY_USERNAME = u'yourspotifyuser'
	SPOTIFY_PASSWORD = u'yourspotifypass'
```

Notice here I've modified the default Mixer configuration for Mopidy to use the `pulsemixer` configuration. This is because I found my HDMI audio to be highly corrupted with lots of distortion and clicking and popping sounds. So I quickly installed Pulse Audio to fix this:

```bash
	sudo apt-get install pulseaudio
```

Finally we need to setup our `init.d` script to run Mopidy on startup:

```bash
	sudo vi /etc/init.d/mopidy
```

Then add something like this to the file:

```bash
	#!/bin/bash
	# mopidy daemon
	# chkconfig: 345 20 80
	# description: mopidy daemon
	# processname: mopidy
	### BEGIN INIT INFO
	# Provides:          mopidy deamon
	# Required-Start:    $remote_fs $syslog $network
	# Required-Stop:     $remote_fs $syslog $network
	# Default-Start:     2 3 4 5
	# Default-Stop:      0 1 6
	# Short-Description: Start mopidy daemon at boot time
	# Description:       Enable mopidy music server
	### END INIT INFO

	DAEMON_PATH="/usr/bin/"

	DAEMON=mopidy
	DAEMONOPTS=""

	NAME=mopidy
	DESC="My mopidy init script"
	PIDFILE=/var/run/$NAME.pid
	SCRIPTNAME=/etc/init.d/$NAME

	case "$1" in
	start)
	        echo "Starting Mopidy Daemon"
	        start-stop-daemon --start --chuid mopidy --background --exec /usr/bin/mopidy \
	                --pidfile $PIDFILE --make-pidfile \
	                -- 2>/var/log/mopidy.log
	;;
	stop)
	     echo "Stopping Mopidy Daemon"
	        start-stop-daemon --stop --exec /usr/bin/mopidy --pidfile $PIDFILE
	;;

	restart)
	        $0 stop
	        $0 start
	;;

	*)
	        echo "Usage: $0 {start|stop|restart}"
	        exit 1
	esac
```

This script makes use of the really handy [start-stop-daemon](http://man.he.net/man8/start-stop-daemon) to manage the starting and stopping of the Mopidy process.

Finally we need to tell the system when to start the script so we do this to setup the defaults.

```bash
	sudo update-rc.d mopidy defaults
```

Finally we should be able to reboot our box and have Mopidy up and running automatically. However...

### XBMC is now broken! Oh noes!

Dun dun duuun! It seems XBMC will no longer boot, it just sits at the loading screen.

Turns out that Mopidy uses some version of a dependency called libtag that is incompatible with the one that XBMC needs. This is really unfortunate and had me scratching my head for ages. But don't worry, it turns out there's an easy fix, we just need to get XBMC to look in `/usr/local/libs` for its dependencies first. So we just need to modify the xbmc startup script:

```bash
	sudo vi /etc/init.d/xbmc
```

After the comments at the top of the file, add the following line:

```bash
	export LD_LIBRARY_PATH=/usr/local/lib
```

Now try to restart xbmc with `sudo service xbmc start` and you should be good to go!

### Phew...

That was a long one. Took a whole day of messing around to figure out how to get all these things to work nicely together, but I'm quite happy with the result. I can now control my XBMC from my Android devices, and play music without having to use the TV which is lovely.

Sadly the Raspberry Pi is still a little underpowered for all this, I can't help but think if it just had a little bit more ram and a few hundred more mhz it would be the ideal media device. As it stands its usable, and ridiculously good value for money for the 40$ it costs to get up and running.

#### References

I had to consult an awful lot of the internet to get this running, so here's a few links I used to get to the above, they may come in useful if this information gets out of date.

* http://docs.mopidy.com/en/latest/installation/raspberrypi/
* http://docs.mopidy.com/en/latest/_modules/mopidy/settings/
* http://elinux.org/R-Pi_Troubleshooting#Sound
* https://github.com/mopidy/mopidy/issues/132
* http://blog.scphillips.com/2013/01/getting-gstreamer-to-work-on-a-raspberry-pi/
* https://github.com/raspberrypi/linux/issues/128
* https://github.com/mopidy/mopidy/issues/266