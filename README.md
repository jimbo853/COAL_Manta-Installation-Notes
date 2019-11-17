# COAL_Manta-Installation-Notes
Notes and logs detailing the installation of Manta on Triton/COAL

All the procedures outlined below have been tested on both Windows 10 and iMac Pro (Catalina) machines. I'll include software version numbers wherever possible as well as other requirement details such as disk and memory.

# Step #1: Install Triton/Coal on VMware Workstation 15.5.1

https://github.com/joyent/triton/blob/master/docs/developer-guide/coal-setup.md

Note: The coal-setup as configured above works well with a basic installation but doesn't work with a Manta install. I'm not sure what the limit is on memory or disk but I can say that 16 GB and 400 GB does work.

Follow all the steps exactly down to the completion of the "Adding external Nics" step.

I didn't set up the "Cloud API" or "Fabrics" although I'm sure this might be worth doing eventually.

Also, I would recommend doing a once through the section on "Maintaining Coal" as this includes the steps for performing updates on the various sub-systems.

If you're doing this on VMware Workstation, I would recommend installing WinSCP to enable access (traversing, uploading, downloading, editing) to the file system through a visual interface. The required IP should be 10.88.88.200

Also, WinSCP provides access to a PuTTY terminal which I find to be better than VMware as a command line interface.

I would recommend shutting the system down manually from the command line as allowing VMware to shutdown the OS seems to cause corruption.

The command to shutdown gracefully is: shutdown -y -g0 -i5 halt

I would recommend that whenever you are restarting COAL, allow the boot to finish starting all zones before doing anything of consequence. The best way to know when the startup is complete is to check the health.

sdcadm health

... will tell you if all the zones are running without issue.

Finally, all this assumes that you are logging in as "root" and using the "rootpass" password as defined when you performed Step #1. 

# Step #2: Install Manta

https://joyent.github.io/manta/

The relevant section in the "Operator's Guide" is "Deploying Manta"

I found installing Manta to be more difficult than installing COAL because some of the command usage was not that clear to me. As such I'll include a bit more detail in itemizing the steps below. I would still recommend reading the Op Guide as you go. The initial steps are all done from the global zone of the headnode:

/usbkey/scripts/setup_manta_zone.sh
/zones/$(vmadm lookup alias=manta0)/root/opt/smartdc/manta-deployment/networking/gen-coal.sh > /var/tmp/netconfig.json
ln -s /zones/$(vmadm lookup alias=manta0)/root/opt/smartdc/manta-deployment/networking /var/tmp/networking
cd /var/tmp/networking
./manta-net.sh netconfig.json

The next part of the installation must be done from within the new Manta zone 0. The Op Guide doesn't make this very clear but you'll need to login to the zone, before doing any further Manta installation steps:

Login to manta0:

sdc-login manta0

You can logout; i.e. back to the global zone of the headnode with:

exit

Once you're logged in, the following commands should complete the basic Manta deployment:

(Replace the given email with your own...)

manta-init -s "coal" -e "jimbo.cooper853@gmail.com"
manta-deploy-coal

The manta-deploy-coal takes a long time to finish and this is where I experienced errors until I had allocated enough memory and disk within VMware prior to the initial install (Step #1).

When the deployment is complete, logout of the mant0 zone with "exit" 

Once you're back into the headnode global zone, you can check the disk space allocation with:

zfs list zones

...and you can also list the Manta storage zones with:

manta-adm show storage

At this point I would recommend rebooting the system with "reboot" and after waiting until the system fully restarts and logging in again as root, you can install nodejs.

# Step #3: Install pkgin and node

Following login, backout to the root of the headnode with:

cd ../

acquire the package manager source with:

curl -k http://pkgsrc.joyent.com/packages/SmartOS/bootstrap/bootstrap-2019Q2-x86_64.tar.gz | gzcat | tar -xf -

.. and note that the above command is case sensitive

Build the code:

pkg_admin rebuild
pkgin -y up

You can now install nodejs with:

pkgin -y install nodejs
 .. and
pkgin -y install npm

.. to install the node package manager

I'm still having a problem getting the latest version of node (the above installs 10.6.0)

