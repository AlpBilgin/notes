# wsl

## enable remote desktopping

stolen from https://dev.to/rescenic/comment/obea

In Ubuntu WSL:
sudo apt-get purge xrdp

then

sudo apt-get install xrdp
sudo apt-get install xfce4
sudo apt-get install xfce4-goodies

configure :
sudo cp /etc/xrdp/xrdp.ini /etc/xrdp/xrdp.ini.bak
sudo sed -i 's/3389/3390/g' /etc/xrdp/xrdp.ini
sudo sed -i 's/max_bpp=32/#max_bpp=32\nmax_bpp=128/g' /etc/xrdp/xrdp.ini
sudo sed -i 's/xserverbpp=24/#xserverbpp=24\nxserverbpp=128/g' /etc/xrdp/xrdp.ini
echo xfce4-session > ~/.xsession

sudo nano /etc/xrdp/startwm.sh
comment these lines to:
#test -x /etc/X11/Xsession && exec /etc/X11/Xsession
#exec /bin/sh /etc/X11/Xsession

add these lines:
# xfce
startxfce4

sudo /etc/init.d/xrdp start

Now in Windows, use Remote Desktop Connection
localhost:3390
then login with Xorg, fill in your username and password.
