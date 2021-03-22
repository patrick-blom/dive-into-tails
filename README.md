##Dive into Tails

Today, where Google knows everything, Facebook analyses your faces during a video chat and Edward Snowden is hunted by the United States, privacy on the Internet becomes more and more a privilege than a standard. Because of this grievance some developers decide to create a live distribution which allows you to use the Internet in a more anonymous and secure way. This distribution is called Tails an acronym for "The Amnesic Incognito Live System". In short it's a Debian based OS which routes all your network traffic through the TOR network mixed up with some nice OpenPGP encryption features and the possibility to set up a LUKS encrypted partition to persist your data. If you want more information about the Tails project take a look at the official project page https://tails.boum.org/

I decide to use Tails as my portable OS, stored on a USB thumb(-)drive which is attached to my keyring and providing me the opportunity to use every pc without leaving my digital fingerprint behind. The installation was simple and everything worked as expected. But there was something negative, the system doesn't look like "MY-System". The opportunity to customize Linux in the way I want is one of the main reasons for me to choose this operating system, so I start searching the documentation for possibilities to make Tails look more the way I want. 

My goals:
- change the background image
- set a custom icon theme
- change the default look for the terminal application
- being update-able

---

#### Here is how it works.
First activate the persistence mode and select the dot-files for persistence. If you don't know how to do this, check the documentation under https://tails.boum.org/doc/first_steps/persistence/configure/index.en.html

After that, create a bash script inside the dot-files folder which contains all commands to customize the system. This script will be executed at every login and sets your custom config. Because of the idea to run maybe multiple scripts after the login process I decided to create a special folder for my startup scripts. 

```bash
mkdir /live/persistence/TailsData_unlocked/dotfiles/scripts
touch /live/persistence/TailsData_unlocked/dotfiles/scripts/set-custom-settings.sh
chmod +x /live/persistence/TailsData_unlocked/dotfiles/scripts/set-custom-settings.sh
gedit /live/persistence/TailsData_unlocked/dotfiles/scripts/set-custom-settings.sh
```

Don't forget to set up the bash in the first line of your "set-custom-settings.sh"-file

```bash
#!/bin/sh
```

Now let's take a look at the background image. Tails comes with gnome which can easily be customized with the gsettings tool. Before set up the new wallpaper create a directory called wallpaper into the dot-files folder which will contain your custom wallpaper/s. 

```bash
mkdir /live/persistence/TailsData_unlocked/dotfiles/wallpaper
```

Then place your wallpaper inside the folder and update your set-custom-settings.sh script with the following command.

```bash
#set wallpaper
gsettings set org.gnome.desktop.background picture-uri file:///home/amnesia/wallpaper/FileNameOfYourFancyWallpaper.jpg
```


Next set up your custom icon theme. I choose a cool one from gnome-look.org .

Just as the wallpaper, place your theme in the dotfiles directory in a .icons folder.

```bash
mkdir /live/persistence/TailsData_unlocked/dotfiles/.icons
mv /myCurrentThemeLocation/coolTheme /live/persistence/TailsData_unlocked/dotfiles/.icons/coolTheme
```

Now update the set-custom-settings.sh script with this command

```bash
#set icon theme
gsettings set org.gnome.desktop.interface icon-theme coolTheme
```

This will set the gnome interface theme to your custom theme. 

Its time to change the ugly terminal colors to something cool. In gnome the application settings were stored for every user in a .gconf folder inside their home directory. So if you want to change the settings for the terminal we have to change these files. For the Terminal application the default config is stored by an xml-file and placed in the following folder.

```bash
~/.gconf/apps/gnome-terminal/profiles/Default/%gconf.xml
```

But wait, why changing the default config instead of creating an new profile ? Good question, the answer, because it works. I tried to set up a additional profile and first it seemed to work but when I started the command to set my profile as the default nothing happened. If someone can tell me why this problem occurs, feel free to contact me ( tails[at]bl0m[dot]de ). Usually I only use one color theme inside the terminal, so for me it's a good solution.

To change the default settings of the Terminal use the settings GUI or change the xml file directly. If everything looks as it should copy the xml file including the folder structure to the dot-files directory.

```bash
mkdir /live/persistence/TailsData_unlocked/dotfiles/.gconf/apps/gnome-terminal/profiles/Default/
cp ~/.gconf/apps/gnome-terminal/profiles/Default/%gconf.xml  /live/persistence/TailsData_unlocked/dotfiles/.gconf/apps/gnome-terminal/profiles/Default/ 
```

To ensure that the default profile is really set at the login process indeed, update your set-custom-settings script with this command.

```bash
#set terminal profile
gconftool-2 --set --type=string /apps/gnome-terminal/global/default_profile Default
```

Last but not least you have to include your custom script into the login process. The best way to do that is to edit the .xsessionrc inside the amnesia home directory. This script will be executed after every successful login. So open the file using gedit and add the following lines of code at the end of the file.

```bash
#set custom settings
if [ -f ~/scripts/set-custom-settings.sh ]; then
    . ~/scripts/set-custom-settings.sh
fi
```

This code will check if your file is present and execute it. Now save the file and copy it to the dot-files folder. 

```bash
cp ~/.xsessionrc /live/persistence/TailsData_unlocked/dotfiles/.xsessionrc
```


That's it, you're done. For the next time you will boot your Tails OS and login into your machine your custom settings will be loaded. But why does this work? The  clou in the startup process is, that the persistent partition is mounted before the user will be logged in. And so your custom dot-files will also be linked before the login process is completed. This means that your custom .xsessionrc file will be linked to the amnesia home directory before the X-Server executes it. If the X-Server executes the file your additional code is executed and runs all commands to set up the config.

The fact that we only use mechanisms provided by Tails, is the guarantee to be compatible with the coming Tails updates.

But the crux of the encrypted persistent convenience is, that the whole partition will be decrypted during the start up. This massively slows down the login process. But I think a system start up in 30-45 seconds (in my case) is acceptable for a nearly anonymous Operating System that looks the way I want.




