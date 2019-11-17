# COAL_Manta-Installation-Notes
Notes and logs detailing the installation of Manta on Triton/COAL

All the procedures outlined below have been tested on both Windows 10 and iMac Pro machines. I'll include software version numbers wherever possible as well as other requirement details such as disk and memory used. 

Step #1: Install Triton/Coal on VMware Workstation 15.5.1

https://github.com/joyent/triton/blob/master/docs/developer-guide/coal-setup.md

Note: The coal-setup as configured above works well with a basic installation but doesn't work with a Manta install. I'm not sure what the limit is on memory or disk but I can say that 16gb and 400gb does work.

Follow all the steps exactly down to the completion of the "Adding external Nics" step.

I didn't set up the "Cloud API" or "Fabrics" although I'm sure this might be worth doing eventually.

Also, I would recommend doing a pass through the section on "Maintaining Coal" as this includes the steps for performing updates on the various sub-systems.

If you're doing this on VMware Workstation, I would recommend installing WinSCP to enable access (traversing, uploading, downloading, editing) to the file system through a visual interface. The required IP should be 10.88.88.200

Also, WinSCP provides access to a PuTTY terminal which I find to be better than VMware as a command line interface.

I would recommend shutting the system down manually from the command line as allowing VMware to do seems to cause corruption.

The command to shutdown gracefully is: shutdown -y -g0 -i5 halt

I would recommend that whenever you are restarting COAL to allow the boot to finish starting all zones before doing anything of consequence. The best way to know when the startup is complete is to check the health:

sdcadm health

will tell you if all the zones are running without issue.

Step #2: Install Manta

https://joyent.github.io/manta/

The relevant section in the "Operator's Guide" is "Deploying Manta"

I found installing Manta to be more difficult than installing COAL because some of the command usage was not that clear to me. As such I'll include a bit more detail in itemizing the steps below. I would still recommend reading the Op Guide as you go. The initial steps are all done from the global zone of the headnode:

/usbkey/scripts/setup_manta_zone.sh
/zones/$(vmadm lookup alias=manta0)/root/opt/smartdc/manta-deployment/networking/gen-coal.sh > /var/tmp/netconfig.json
ln -s /zones/$(vmadm lookup alias=manta0)/root/opt/smartdc/manta-deployment/networking /var/tmp/networking
cd /var/tmp/networking
./manta-net.sh netconfig.json

The next part of the installation must be done from within the new Manta zone 0. The Op Guide doesn't make this very clear but you're able to login to the zone, next set of steps fall into place:

Login to manta0:

sdc-login manta0

You can logout; i.e. back to the global zone of the headnode with:

exit

Once you're logged in, the following commands should complete the basic Manta deployment:

Replace the given email with your own...
manta-init -s "coal" -e "jimbo.cooper853@gmail.com"
manta-deploy-coal




