Manage Your CloudAtCost VM with Cloud66
=======================

## Introduction

Cloud66 is a brilliant service that helps us developers stick to the development instead of system administration.
We've all struggled through configuring  a rails environment from scratch by rummaging through a millions
stack overflow posts.  When you're finished, if you managed to get your app running at all, you always
feel like maybe you left something out. You might be wondering if you've sealed all the security holes.  
Then in 2 months when you update a package and everything stops working the real panic and frustration starts.

Cloud66 does all the scary stuff for you!  

Cloud66 has a few providers built in, but I decided to do things the hard way.
In this document I'm going to be going through the steps to show you how to prepare a VM purchased from
CloudAtCost.com for Cloud66.  

The steps in this guide are geared towards a CloudAtCost VM, but its likely that they are releveant 
for any similar cloud provider.

## Prepare Your VM
When you first purchase your VM from CloudAtCost.com you'll be asked to select your OS.
You'll need to make sure you pick Ubunutu 12.04.XX.  Currently this is the only choice 
Cloud66 will support.

If you've already built your VM with another choice its okay.  Login to your management panel 
for CloudAtCost and rebuild the VM with Ubuntu.

Once you have the VM loaded with the correct OS you'll want to log in to the box.
You can do this through the management panel but I prefer using a terminal application vs. the web 
terminal emulators.

      ssh root@162.xxx.xx.xxx

Your root password can be found by logging in to CloudAtCost.com and finding your VM under
the products menu.

Once you're logged in the first thing you'll need to do is create the user you'll want Cloud66 to use.

      sudo useradd -d /home/joe -m joe
      
Now lets set joe user's password:

      passwd joe
      
Cloud66 requires that the user you provide can connect without using a password.  We'll need to do 
a few things to set this up securely.

Lets create a new group:

      groupadd sudoers
      
And put joe user in that group:

      sudo usermod -a -G sudoers joe


Now lets make it so anyone in that group won't have to enter a password for sudo:
We'll edit /etc/sudoers file by entering:

      visudo
      
Then below this line:

      %sudo   ALL=(ALL:ALL) ALL
      
Add the line:

      %sudoers ALL=(ALL) NOPASSWD:ALL
      
Type ctrl+x to exit, and 'Y' to save and you're all set.

Now that we have joe user all ready to go lets take him for a spin:

      su joe

You might notice that your default profile isn't bash. You'll want to change that by typing:

      bash
      
That's a temp fix though.  Lets make your profile always use bash.  This is recommended to a smoother
deploy through cloud66.

     chsh
     
After entering joe's password enter the path:

      /bin/bash

You should now have a prompt something like:

      joe@ubuntu1204:/home$

Okay, now its time to generate some ssh keys.  There are several guides on this subject so 
i'll leave out the details and just list the steps.  

Make sure you are still the joe user.  

      whoami
      
Then type the following substituting your email address:

      ssh-keygen -t rsa -C "your.email@example.com"

You'll have a series of about 3 prompts after that.  Just hit enter leaving them all blank.
This will eventually bring you back to your prompt.

Now enter the following ( being sure to enter your VM's real IP address ):

      scp ~/.ssh/id_rsa.pub joe@162.xxx.x.xxx:/home/joe/.ssh/uploaded_key.pub

And:

      ssh joe@162.xxx.x.xxx "echo `cat ~/.ssh/uploaded_key.pub` >> ~/.ssh/authorized_keys"

This is all the setup you will need to prepare your VM for Cloud66!

## Building The Cloud66 Stack

Now that we have our VM ready and waiting for our rails environment to be setup its time to build our stack!

Login to Cloud66.com and from the 'Dashboard' click 'New Stack'.
The UI for Cloud66 is pretty easy to use.  
The first screen is pretty straightforward.

You'll need to provide the github url for your app, which environment you want to use, and the name 
of your stack.  
The stack name is used for identification within Cloud66 only so enter something like:

      MySweetApp - Staging

When you're ready click the big blue 'Analyze' button.

It'll take a few seconds while Cloud66 figures out what kind of DB you're using along with some other items.

The first thing you'll see is a section where you can add some environment variables. 

Cloud66 will analyze your database.yml and determine what your database user and password should be.
If you don't have your database.yml checked in to github ( which you shouldn't ) then Cloud66 will 
generate a random password for you.  You can edit this information in the environment variables.

Next up is the ruby version.  Pay close attention here because if you choose the wrong ruby its not easy
to change later.

The "Where are you Deploying To?" section is where it gets interesting for us.

Choose: "Deploy to my own server" for Deployment target.

Here's where all the SSH key business comes into play.  

Click 'Add new Key'.

Now from back in your VM ( logged in as joe user ) type:

      vi /home/tiki/.ssh/id_rsa
      
Copy the resulting text ( including the beginning and ending comments ) into the Private SSH Key text area.
You can enter anything you want for the name.  I usually name mine so I have a hint to what server its from:

      CloudAtCostVM1
      
Next enter the IP address of your VM in the “Deploy Web Server” input.

For this app i’m just going to install the DB on the same VM so for “Deploy Mysql”
I’m going to leave the default as “Locally -- Same as the web server”

Give everything a quick look.  Especially the ruby version, username ( in our case joe ), and web server IP.

Then click the pretty blue ‘Deploy’ button.

If you get an error right away that says something like:

      Failed to download host file from server
      
Its likely due to a user permission problem.  Make sure you have created the user and groups as recommended in the CloudAtCost section.

If you got passed that step kick back and relax for a few minutes while cloud66 does
everything for you!  



