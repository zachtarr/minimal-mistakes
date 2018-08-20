---
title:  "Deploying FontAgent Pro (and Smasher) using Munki"
date:   2016-02-14
blog: true
---
Many of the clients I work with use [FontAgent Pro](https://insidersoftware.com) for font management. Unfortunately, setting it up has previously been a manual process that involved touching each workstation to install, license and configure the app. Here are some things that I have done to start to automate this a bit.

Note: While everything seems to work ok in my tests, I have not rolled any of this out in production yet. I will edit this once I do the next deployment. YMMV.

# AutoPkg
I wrote an [AutoPkg](http://autopkg.github.io/autopkg/) recipe for FontAgent Pro 6 [here](https://github.com/autopkg/zachtarr-recipes). 

Insider Software publishes a [single link](http://insidersoftware.com/downloads/FontAgentPro6.dmg) that always points to the latest release. The release is packaged as a .pkg inside of a .dmg, which means the recipe is pretty basic. Except for one thing- Smasher.pkg is bundled into the same .dmg. 

When the install is done through Munki, both FontAgent Pro and Smasher are installed. However, if you trigger an uninstall through Munki, it _only_ removes FAP. To address this, I added a one-line post-uninstall script to the .munki recipe that triggers the Smasher uninstall script in /Applications/Smasher/Uninstall.app/Resources/.

# License
When FontAgent Pro is activated, it drops two files (Activation and Registration) into /Library/Application Support/FontAgent Pro/. If you are using a volume license, you can copy these files to the same location on another Mac to complete the activation. This was suggested to me by Insider Software as the best way to deploy the license to multiple Macs.

To do this in a repeatable fashion, I made a [template Makefile](https://github.com/zachtarr/fontagent-pro-6-tools/tree/master/FAP6-Activation-Luggage-Pkg) for [The Luggage](https://github.com/unixorn/luggage). This way I can simply copy the Activation and Registration files into the same directory as the Makefile, edit the TITLE and REVERSE_DOMAIN at the top of the Makefile, and run 'make pkg'.

Smasher [works the same way,](https://github.com/zachtarr/fontagent-pro-6-tools/tree/master/Smasher-Activation-Lugage-Pkg) although it only requires the Registration file.

# Updates
I don't want the built in updater to run. [I made a .mobileconfig profile](https://github.com/zachtarr/fontagent-pro-6-tools/tree/master/FAP6-Disable-Updates) using [mcxToProfile](https://github.com/timsutton/mcxToProfile), and then stripped out every preference except for 'CheckForUpdatesAutomatically' to accomplish this.

[Here](https://github.com/zachtarr/fontagent-pro-6-tools/tree/master/Smasher-Disable-Updates) is a profile for Smasher as well.

# Deployment
All 3 of these pieces (the application, the activation pkg, and the profile) can be deployed with Munki (or in my case, [Gruntwork](https://mac-msp.com)) to the correct workstations. When users launch the application, they just have to click through the initial setup screens, and then login to the font server with their credentials.

# Future Plans
I'd love to streamline this process even more, especially by bypassing the initial setup screens and auto-installing the Adobe Plugins.

