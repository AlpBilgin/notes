.\VBoxManage.exe guestproperty get <device_name> "/VirtualBox/GuestInfo/Net/0/V4/IP"

## forward port to host port


Open your VirtualBox Manager. Select your VM, open the “Settings” menu, and go to the “Network” tab. You should see that for at least one of your adapters, “Enable Network Adapter” is checked, and the “Attached to:” field is set to “NAT”.

Open up the “Advanced” section, and select “Port Forwarding”.

You should see a dialogue window appear with a table of port forwarding rules. Add a new rule using the button with a “+” sign on the right, and fill in the following:

Name: this can be whatever you want, e.g. sshd
Protocol: TCP
Host Port: some unused port on your host machine, e.g. 4118
Guest Port: whatever port your guest machine is listening for ssh requests on, i.e. 22 by default.

Now you should be able to ssh into your local VM! If you’re working in a terminal on your host machine, you may login to your guest machine using:

[host]$ ssh -p <host-port> <username>@localhost
The -p flag tells the ssh client to connect to the specified port
The username should be whatever you set up your local VM with
We login to localhost because we’re just connecting to our local machine
