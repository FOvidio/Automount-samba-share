# Automount-samba-share
Automount samba share linux Debian

1)Install cifs-client and smbclient (or samba-client on samba distros)

2)create file /etc/smbcredentials with the username and password of your samba share

username=USER
password=PASSWORD

sudo chmod 600 /etc/smbcredentials

3)Add to /etc/fstab the line

//192.168.2.60/install /install cifs credentials=/etc/smbcredentials,noperm,file_mode=0777,dir_mode=0777,iocharset=utf8,noauto,nofail 0 0

4) Create mount dir

sudo mkdir /install

5)Create executable file /usr/local/bin/samba-mount

#!/bin/bash
while true
do
    if ! smbclient -A /etc/smbcredentials //192.168.2.60/install -c '' &> /dev/null
    then
    	umount -f -l /install &> /dev/null
    else
    	if [[ -z $(mount | grep /install) ]]
   	then
 	    mount /install &> /dev/null
        fi
    fi
    sleep 15
done


6)Create /etc/systemd/system/samba-mount.service

[Unit]
Description=Samba Mount
Wants=network.target
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/samba-mount
ExecStop=/usr/bin/umount -f -l /install &> /dev/null

[Install]
WantedBy=multi-user.target


7) sudo systemctl daemon-reload

8) sudo systemctl enable --now samba-mount 
