https://askubuntu.com/a/60395

# 0. Introduction and Background

This answer is based on an extensive research done by various Ubuntu users that worked together in almost all issues related to Broadcom. Special thanks to [chili555][1] who helped in the Ubuntu forums and on this site with many questions related to Wireless devices and to others who have contributed through E-Mail, chats, IRC and more in testing various drivers with several of the most popular Broadcom Wireless cards (Huge Thanks to Chili555 really. This guy knows his stuff).

 In total we wanted to offer an answer that could be easy to follow and covered most Broadcom Cards / Drivers. After you follow this guide, you will **NEED** to test your wireless connection for at least 2 hours (I actually recommend 8 hours) with another device in either Ad-Hoc Mode, Infrastructure Mode or Both. Common problems that will be solved (Apart from drivers not installing) are:

+ Connections timeout after several minutes or hours
+ Stops searching for other devices (Does not see any other device)
+ Keeps asking for password even on cases where AP does not have any
+ Stops any receiving/transmitting traffic (Needs reboot to temporarily fix)
+ Crashes system with dmesg errors in log (Link 1 Below)
+ System freezes completely (You can only press Reboot/Power button) (Link 1 Below)
+ Creates huge log reports trying to correctly configure or connect
+ Fails when installed via **Additional Drivers** / **Additional Hardware** (Link 3 Below)
+ Connects and disconnects continuously every X amount of seconds
+ Appears connected on Network Manager but does not receive Internet
+ Tries to connect many times without correctly finishing connection 
+ Takes too long to connect
+ After upgrading from a previous version (eg: 12.04 to 12.10) it stops working
+ Wireless card does not turn on, enable or disable (Link 2 Below)
+ Wireless card blocked by hardware
+ More problems found in Launchpad, Ubuntu Forum and Askubuntu

Link 1 -  https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1060268  
Link 2 - https://bugs.launchpad.net/ubuntu/+source/bcmwl/+bug/732677  
Link 3 - Gives an error similar to "Sorry, installation of this driver failed."

So with that in mind, the following is what we have right now which is simplified in just 3 steps:


# 1. Knowing what Broadcom Wireless Card you have

There are dozens of Broadcom wireless cards and more seem to appear every day. The key to finding the correct driver for any network card is what is known as the **PCI ID** (PCI.ID). To find out which PCI.ID you have, we proceed to opening the terminal by pressing <kbd>CTRL</kbd>+<kbd>ALT</kbd>+<kbd>T</kbd> (It should open a window with a blank background) and inside this terminal we run the following command:

    lspci -nn -d 14e4:

You will get something like the following if you have a Broadcom Wireless Adapter (The ID 14e4 used in the example above in most cases is a Broadcom Wireless Card):

    Broadcom Corporation BCM4306 802.11bgn Wireless Network Adapter [14e4:4320] (rev 03)

The PCI.ID in this example is **14e4:4320** as seen inside the Brackets [...]. In some cases you will also need the revision version (if it appears) for some special cases. In this case, the revision version is **rev 03** as shown inside the Parentheses (...) at the end. So what you will need after this search is:

    [14e4:4320] (rev 03)

With this new information you can look in the table below and select the appropriate method to install your driver. For example, In this case, since you have the **14e4:4320 rev 03**, if we go down the list to the one that shows the exact same PCI.ID you will see that in the columns for Ubuntu 18.04 or 20.04 it shows the `firmware-b43-installer` package driver. This means that you will only have to install this particular package since it appears in all Ubuntu version columns.

**NOTE** - Before proceeding, if you have previously installed any drivers, have blacklisted or uncommented any driver files or configuration files or have done any changes whatsoever to the system to make the drivers work in previous attempts, you will need to undo them in order to follow this guide. We assume you are doing this from scratch and have not changed any configuration files, modules or drivers in the system in any way (apart from updating the system). This includes any installations using apt-get, aptitude, synaptic, dpkg, software center or manual compilation and installation of the packages. The system has to start from scratch in order for this to work and to avoid any conflicts that may appear if earlier work was done.

For example, if you have previously installed the `bcmwl-kernel-source` package, you will need to remove it by using the purge method:

    sudo apt-get purge bcmwl-kernel-source


# 2. Preparing the System

If you have just installed Ubuntu, you will need to build an index of available packages before we can install your driver if you have not done so already:

    sudo apt update

I would even go so further as to update the Ubuntu list of PCI.IDs:

    sudo update-pciids

Just in case the ID of a particular new Broadcom Device you are using has just appeared.

Now using the PCI.ID you found in the steps above, we then search in the list below to find the matching PCI.ID and the method to install the driver associated with it in a simple and correct way. The terminal will be used to avoid any GUI related issues. This applies with all cases, except as noted. The installation procedure is done only via terminal and also while connected to the internet with a temporary wired ethernet connection or USB modem or any means possible that can give your PC, for the time, Internet access. After you find in the list below the correct package we then proceed with the installation.

# 3. Installing the Package (online)

Assuming you used the PCI.ID **14e4:4320 rev 03** as found in your search above, and then looked at the table below and found that the correct package to install is the `firmware-b43-installer` (Specific to Broadcom) and the `linux-firmware` (Carries over Broadcom related drivers along with other types of drivers), we then proceed to simply install this package in the terminal:

    sudo apt install firmware-b43-installer

    sudo apt install linux-firmware

and then reboot

    sudo reboot

The format to install is pretty simple, it's just:

    sudo apt install <PACKAGE_NAME>

In the example above, the **PACKAGE_NAME** is `firmware-b43-installer`.


## BROADCOM WIRELESS TABLE (Updated Oct 29, 2020)

    PCI.ID      	    18.04 LTS 	 	                 20.04+
    ------------------------------------------------------------------------------------
    14e4:0576	    	Special Case #1                   UNKNOWN      
    14e4:165f  	    	UNKNOWN                           UNKNOWN
    14e4:1713  	    	firmware-b43-installer            firmware-b43-installer / linux-firmware      
    14e4:4301  	    	firmware-b43-installer            firmware-b43-installer / linux-firmware      
    14e4:4306	    	firmware-b43-installer            firmware-b43-installer / linux-firmware      
    14e4:4306 rev 02	firmware-b43-installer            firmware-b43-installer / linux-firmware      
    14e4:4306 rev 03	firmware-b43-installer            firmware-b43-installer / linux-firmware      
    14e4:4307	    	firmware-b43-installer            firmware-b43-installer / linux-firmware      
    14e4:4311	    	firmware-b43-installer            firmware-b43-installer / linux-firmware      
    14e4:4311 rev 01   	firmware-b43-installer            firmware-b43-installer / linux-firmware      
    14e4:4312	        firmware-b43-installer            firmware-b43-installer / linux-firmware      
    14e4:4313	    	firmware-b43-installer            firmware-b43-installer / linux-firmware      		   
    14e4:4315       	firmware-b43-installer            firmware-b43-installer / linux-firmware      
    14e4:4315 rev 01    firmware-b43-installer            firmware-b43-installer / linux-firmware
    14e4:4318	        firmware-b43-installer            firmware-b43-installer / linux-firmware      		    
    14e4:4318 rev 02    firmware-b43-installer            firmware-b43-installer / linux-firmware      		    
    14e4:4319	        firmware-b43-installer            firmware-b43-installer / linux-firmware      		   
    14e4:4320 rev 02	firmware-b43-installer            firmware-b43-installer / linux-firmware      		   
    14e4:4320 rev 03	firmware-b43-installer            firmware-b43-installer / linux-firmware      		
    14e4:4321	    	firmware-b43-installer            firmware-b43-installer / linux-firmware  
    14e4:4324           firmware-b43-installer            firmware-b43-installer / linux-firmware      	
    14e4:4325		    firmware-b43-installer            firmware-b43-installer / linux-firmware      
    14e4:4328		    firmware-b43-installer            firmware-b43-installer / linux-firmware
    14e4:4328 rev 03	bcmwl-kernel-source               bcmwl-kernel-source      
    14e4:4329		    bcmwl-kernel-source	              bcmwl-kernel-source        
    14e4:432a	       	bcmwl-kernel-source	              bcmwl-kernel-source        
    14e4:432b           bcmwl-kernel-source	              bcmwl-kernel-source        
    14e4:432c  	       	bcmwl-kernel-source	              bcmwl-kernel-source        
    14e4:432d	    	bcmwl-kernel-source	              bcmwl-kernel-source       
    14e4:4331	    	firmware-b43-installer            firmware-b43-installer / linux-firmware          
    14e4:4335	    	firmware-b43-installer            firmware-b43-installer / linux-firmware      
    14e4:4350	    	firmware-b43-installer            firmware-b43-installer / linux-firmware  
    14e4:4353	    	Special Case #1		              UNKNOWN        
    14e4:4353 rev 01   	Special Case #1		              UNKNOWN                 
    14e4:4357	    	Special Case #1		              UNKNOWN        
    14e4:4358	    	bcmwl-kernel-source	              bcmwl-kernel-source
    14e4:4359	    	firmware-b43-installer	          firmware-b43-installer / linux-firmware       
    14e4:4360	    	firmware-b43-installer            firmware-b43-installer / linux-firmware    
    14e4:4365	    	firmware-b43-installer            firmware-b43-installer / linux-firmware      
    14e4:4365 rev 01	bcmwl-kernel-source               bcmwl-kernel-source      
    14e4:43a0 	    	bcmwl-kernel-source	              bcmwl-kernel-source
    14e4:43a3           UNKNOWN                           firmware-b43-installer / linux-firmware
    14e4:43ae rev 02    UNKNOWN                           UNKNOWN  	  
    14e4:43ba rev 01  	UNKNOWN	                          firmware-b43-installer / linux-firmware
    14e4:43b1 	    	bcmwl-kernel-source	              bcmwl-kernel-source      	 
    14e4:43b1 rev 03    bcmwl-kernel-source               bcmwl-kernel-source              
    14e4:43c3 rev 04    UNKNOWN                           firmware-b43-installer / linux-firmware / Special Case #2                     
    14e4:4727	    	bcmwl-kernel-source		          bcmwl-kernel-source        
    14e4:4727 rev 01   	Special Case #1                   Special Case #1        
    14e4:a962	    	firmware-b43-installer            firmware-b43-installer / linux-firmware      
    ------------------------------------------------------------------------------------

For All cases, always install the `linux-firmware` package. This will always be up-to-date with the latest Broadcom drivers along with other binary files that could be needed depending on the driver PCIID.
    
**Special Case #1** - Uses `bcma` and `brcmsmac` driver combination. Required firmware is installed by default in the package `linux-firmware`.
    
**Special Case #2** - For the **ASUS PCE-AC88 AC3100** the steps are:

1. [Download This file][2] and after extracting it, put it in /lib/firmware/brcm   
 `sudo cp brcmfmac4366c-pcie.bin /lib/firmware/brcm/brcmfmac4366c-pcie.bin`   
2. Then `sudo nano /etc/rc.local` add **modprobe brcmfmac** and save
3. `sudo reboot`

In hardware like the Lenovo S10-2, if your wireless card gets stuck trying to connect to an SSID (keeps trying to connect), then the alternative to get it working would be to install the `bcmwl-kernel-source` package (Remove any other installed packages related to it). Read the Debugging section below for more information regarding this wireless device.

**IMPORTANT NOTE** - After September 2014, if you follow this answer and still you have problems installing the correct driver, please try the `firmware-b43-installer` package and the `linux-firmware` package and notify us via comments. There were some changes and some drivers will only work with this package. Remember to have a clean system before installing it:

    sudo apt install firmware-b43-installer

    sudo apt install linux-firmware

In some particular cases, after installing the `firmware-b43-installer` you need to remove the b43 module, enable it again and even proceed to unblock with rfkill:

     sudo modprobe -r b43
     sudo modprobe b43    
     sudo rfkill unblock all  

If you have a Broadcom card that has a different pci.id, please ask a new question. Once solved, the solution will be added to this howto.

# 4. Installing the Package (offline)

## 4.1 Installing `firmware-b43-installer`

To install `firmware-b43-installer` offline see [this answer](https://askubuntu.com/a/730813/167850).

## 4.2 Installing `bcmwl-kernel-source`

For cases where you need to install `bcmwl-kernel-source` but you are offline, [here](https://askubuntu.com/questions/626642/how-to-install-broadcom-wireless-drivers-offline/626653#626653) is an excellent answer about it.
But basically follow this steps:

1. Search for the package in the [Ubuntu Package Repositories][5]
2. Make sure you select the correct architecture (32-Bit, 64-Bit, etc..)
3. Download package and dependency packages related to it on the same folder.
4. When you have all packages needed (and their dependencies), proceed on going to the folder that has all packages and run `sudo dpkg -i *.deb`. This will install all packages in that folder. If it gives any errors, read the error and follow the steps it mentions.

To give an example, after going to point 1 mentioned above, If you had the 
Broadcom `14e4:43a0`, you would search for the `bcmwl-kernel-source` package and after selecting the corresponding Ubuntu version (In my case 16.04 or Xenial) I would land on the following page:

http://packages.ubuntu.com/xenial/bcmwl-kernel-source

On this page I would select the corresponding architecture (32 or 64) but would also need to download the 3 package dependencies mentioned on that page as seen in the following image:

[![enter image description here][6]][6]

After downloading all packages and dependencies, you can proceed on copying all packages to a single folder and running the `dpkg` command as mentioned on step 4 above.

# NOTE

In some computers, before performing the commands, you will need to deactivate the Secure Boot Options in your BIOS. This applies for cases, for example, where the bcmwl-kernel-source is already installed but the driver does not yet work. You can do a reinstall like so, or disable Secure Boot by going to your BIOS Setup:

    sudo apt-get install --reinstall bcmwl-kernel-source


# DEBUGGING

The following information is additional material to read about solving various issues related to Wireless Management and conflicts with other Network devices. Know that it some cases you need to have an updated Kernel version, since each new version of the Kernel introduces either new Network drivers, improvements over existing drivers or solves bugs regarding them. 

Before reading the points mentioned below, be sure to have all repositories enabled on your Ubuntu system. To check, run on the terminal `software-properties-gtk` and make sure all options on the Ubuntu Software Tab are enabled.

* Make sure the Wireless card is not hardware disabled. For example on some laptops you need to press <kbd>Fn</kbd> + <kbd>F2</kbd>
* To configure your wireless devices through the terminal I recommend https://askubuntu.com/questions/16584/how-to-connect-and-disconnect-to-a-network-manually-in-terminal/16588#16588

* If your connection drops every so often some users have suggested to set IPv6 to **Ignore**. Just go to Network Manager (The network icon on the top panel). Click on it then select **Edit Settings**. Then go to the Wireless connection you are using, select it. Now go to the last Tab in there that mentions **IPv6 Settings**. In the Method field select **Ignore**.

* If your laptop does not detect your wireless card some users have mentioned that using `rfkill unblock all` will solve the problem. Others simply turned the WiFi switch on their laptops off and then on again (Physical switch available on this laptops). For more information about `rfkill` please read https://askubuntu.com/questions/211035/rf-kill-unblock-all-does-not-work/211162#211162

* If you are getting **b43-phy0 ERROR: Fatal DMA error /  b43-phy0 warning: Forced PIO** do the following:

        sudo rmmod b43     
        sudo modprobe b43 pio=0 qos=0  
 If it works then add it to you RC files so it is executed every time you boot. You can change PIO to 1 if you need to it.

* If you are having a **Required key not available** when installing a DKMS module (Like Nvidia, Broadcom or others) you can go to [Pilot's Answer Here][3]

* If your wireless card see/not see the router and gets stuck in an endless "Trying to connect (Try 1/3)" loop the solution might be proper configuration of your router or wireless SSID device. 

 For all Wireless cards in general, it is very important to also take into consideration the network devices you are using (Routers, Switches, Wireless Channels and Wireless Bands, etc..). With this information you will be able to evaluate better what the source of the problem could be when you arrive at a dead end. An example would be the Lenovo S10-2 which uses the **14e4:4315 rev 01** PCIID. Even after installing the correct driver the user would end up in a "trying to connect" loop. It would see the wireless SSID but when trying to connect to it, it would enter an reconnecting loop. 

 The solution was that this particular wireless device did not support 40 Mhz channels nor does it support 802.11N. The router in that case was actually broadcasting with a forced 40 Mhz and on WiFi-N only. When the router was set to Auto mode and 20/40 Mhz Channel, the wireless card worked correctly. This is a case scenario that also repeats in other cases, so a proper evaluation of the network equipment would help a lot.

 For cases where you get repeated:

 **ERROR @wl_cfg80211_get_station : Wrong Mac address...**

 when doing a `dmesg` and your wireless connection drops often (Several times an hour or a day), the issue here might be that you are inside a wireless signal that is used as a Wireless Bridge (2 Routers sharing the same SSID and connection). This can happen with modern Routers that have the ability to extend the wireless connection by offering the same SSID. your wireless connection might drop because you might be between both routers and the signal strength between both is almost the same.

 If your connection drops very often, it means you are almost in the middle of both router devices. To lower or eliminate the dropping rate of your wireless device, try to position yourself where your wireless card can see only one router or at least one of the routers has a higher signal strength than the other one.

 There are also some techniques to force the wireless device to only connect to a specific router by setting the BSSID to the MAC Address of the router you wish to connect to. This will force your wireless device to ONLY connect to it.

 [![enter image description here][4]][4]

**Secure Boot Issues**

On some specific scenarios, installing the drivers, be it in offline mode through various DEB packages or through apt-get with internet access, will not work if Secure Boot is not disabled. 

This is because the access needed is denied by Secure Boot so the drivers will look like they are installed correctly when in fact the did not. So for VERY specific cases, you will need to temporarily disable Secure Boot in order for the drivers to work.

**Linux Firmware Update**

On other cases looking for and installing the [latest Linux Firmware][7] would solve the issue. Either solving minor problems that were happening with a working card or making the card work for the first time.


  [1]: https://askubuntu.com/users/19421/chili555
  [2]: https://drive.google.com/open?id=0By-7Mn6Yc0iuZ3RzclhGdnJTYWs
  [3]: https://askubuntu.com/questions/762254/why-do-i-get-required-key-not-available-when-install-dkms-modules-in-ubuntu-16
  [4]: http://i.stack.imgur.com/UR0kQ.png
  [5]: http://packages.ubuntu.com/
  [6]: http://i.stack.imgur.com/e7u38.png
  [7]: https://packages.ubuntu.com/search?keywords=linux-firmware
