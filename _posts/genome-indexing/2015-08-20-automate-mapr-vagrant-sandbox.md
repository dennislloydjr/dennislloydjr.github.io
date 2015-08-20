---
layout: page-fullwidth
title:  "MapR Sandbox + Vagrant, automagically"
subheadline: Turning MapR's Sandbox for Hadoop into a Vagrant Box
teaser: "MapR's sandbox is already prepped for vagrant, all you have to do is download it and figure out how to login..."
breadcrumb: true
header:
   image_fullwidth: "header_3d_printable_sand_play_set_creative_tools.jpg"
tags:
    - genome-indexing hadoop mapr powershell dev-ops vagrant virtualbox scoop
categories:
    - genome-indexing
---
I'm using [MapR's Sandbox for Hadoop](https://www.mapr.com/products/mapr-sandbox-hadoop) to test out the <a href="/genome-indexing">genome indexing project<a/>. It is a virtual machine image with MapR's version of Hadoop already installed and is available for both VM Ware and Virtual Box. As an added bonus it is already setup for vagrant, with most of the work already done for us. All we need to do is download the latest version, create a vagrant box from it and login to play around with Hadoop.

When I started playing around with MapR, the latest sandbox was on version 3.1.0. At some point I upgraded to 4.1.0 and decided now that I'm moving to 5.0.0 I should just automate the 'getting' and 'prep' for my vagrant box. Here's the Powershell script that I use:

{% gist 69edf2aae60994a782df %}

##scoop##
I'm using [Luke Sampson's scoop](https://github.com/lukesampson/scoop) for downloading and installing developer tools. If you're on Windows and not using scoop, check it out. It's quite cool!

The scoop commands here just make sure all the right tools are installed, are up to date and on the path. It assumes that you already have scoop installed. It also attempts to add my devbox bucket in case you don't already have it [devbox scoop bucket](https://github.com/dennislloydjr/scoop-bucket-devbox) so that you can install virtualbox.

##Downloading##
Line 6 specifies the MapR sandbox file to download. When they come out with a new version, this is the only line that should need updating.

Note the magic on line 11. In Powershell, the command wget is an alias for the Invoke-WebRequest method. Normally that wouldn't be a problem, Invoke-WebRequest will download most files just fine. However, it has a performance optimization that attempts to stream the entire file into memory as it downloads it. For large files, it runs out of memory. The current MapR sandbox is about 2.7GB, enough to cause an out of memory error for many. 

So the work around is to use a real wget command. I don't want to remove Powershell's wget alias because other scripts might be expecting it, so I use (scoop which wget) to get the EXE path of the wget program.

##Creating the Vagrant Base Box##
Before creating the vagrant base box, the OVA file needs to be imported into Virtual Box. Then we can package it up and initialize it. If we stopped here, we would have a problem logging in when we execute 'vagrant up'. That is because 'vagrant ssh' attempts to login using an SSH key (surprise, right?). There is already a public key for SSH on the MapR sand box, but we don't have the private key for it. Fortunately, they left the vagrant user with the default username and password. So the next step is to set the username and password in the VagrantFile. That will allow the box to come up. Then when the box is halted, vagrant will generate a new key for the box so that you can access it with SSH in the proper way. Finally, the VagrantFile is cleaned up again so that the username and password is removed.

##Using the Box##
After you've run this script, you should be able to simply bring the box up using 'vagrant up'. If you want to interact with the box once it is up, you can use 'vagrant ssh'.


###About the Image###
The header image on this page is ["3D-printable sand play set - by Creative-Tools.com v2"](https://www.flickr.com/photos/creative_tools/19195690768) by [Creative Tools](https://www.flickr.com/photos/creative_tools), licensed under [CC BY 2.0](https://creativecommons.org/licenses/by/2.0/). 