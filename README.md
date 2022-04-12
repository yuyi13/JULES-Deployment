# Tutorial for deployment of JULES model on NCI accessdev

This is a brief induction for the deployment of the Joint UK Land Environment Simulator (JULES) model on NCI accessdev.

## 1. Register essential accounts
As a prerequisite, the user needs to register two accounts, including:

1. NCI account: https://my.nci.org.au/mancini/signup/0

2. MOSRS account: http://jules-lsm.github.io/access_req/JULES_access.html

Both registrations are free, but the MOSRS registration might take longer to get back to you as it is managed by the UK government. Once the user has these two accounts, please log into the NCI account (https://my.nci.org.au/mancini/login?next=/mancini/) and request the membership of the 'access' group.

![enter image description here](https://github.com/yuyi13/JULES-Deployment/blob/main/images/1_find_project.png?raw=true)

Once we have these two accounts and the membership of access group, we can log into the NCI accessdev server through a terminal. For windows users, I recommend the MobaXTerm (https://mobaxterm.mobatek.net/download-home-edition.html) which is a free terminal software. For MacOS and Linux users, please just directly use the system terminal.

## 2. Setup your accessdev account

Now use your terminal to login to accessdev:
`ssh username@accessdev.nci.org.au`

On Acessdev run
`mosrs-setup`
to setup a GPG agent (a program that stores your passwords).
*Note: you must already had a MOSRS account to run this setup.*

Then log out and back in again, and run
`mosrs-auth`
to save your MOSRS password into the agent. The password will be saved for 24 hours, so you'll need to run it again each day, however it's generally only required to check out a new configuration, not to run one that's already downloaded.

To be able to see the status of jobs Rose+Cylc needs to be able to SSH to and from Gadi
Run the script  `/g/data/hh5/public/apps/nci_scripts/accessdev-gadi-setup`  on accessdev to have this set up automatically.

If you fail at this step, it is likely to be the issue of ssh keys. Please refer to: https://accessdev.nci.org.au/trac/wiki/gadi **SSH Setup** section.

## 3. Run JULES WITHOUT rose
**(1) Download the JULES model and compile it**
Now create a directory under the home directory called 'MODELS', where I usually put different versions of operational models. Here I downloaded a standard version of JULESvn6.1:
`mkdir ~/MODELS`\
`cd ~/MODELS`\
`fcm co fcm:jules.x_tr@vn6.1 jules-vn6.1`\
`export JULES_ROOT=~/MODELS/jules-vn6.1`\
`cd $JULES_ROOT/etc/fcm-make`\
`vi make.cfg`\

Modify the line 22 of the `make.cfg` file from
`$JULES_PLATFORM{?}        = custom`
to
`$JULES_PLATFORM{?}        = ceh`

Compile the JULES model 
`cd $JULES_ROOT`
`fcm make -j 2 -f etc/fcm-make/make.cfg --new`

**(2) Download the suite**
Download a suite designed for tutorial purposes called 'u-cg242', which is a point run of a flux site located in British:
`mkdir ~/roses`
`cd ~/roses`
`rosie checkout u-cg242 ## copy a suite from the server`
`export RSUITE=~/roses/u-cg242`

**(3) Run the model**
Now run the model. There are two ways to do this, with or without rose. The rose suite does not work for me. Therefore here I only introduce the approach to run the model through the command line.
`cd ~/roses`
`mkdir nlists`
`cd nlists`
`rm *`
`rose app-run -i -C $RSUITE/app/jules`
`export NAMELIST=~/roses/nlists`
`cd ~`
`$JULES_ROOT/build/bin/jules.exe $NAMELIST`

Now you should see something like this:
![enter image description here](https://github.com/yuyi13/JULES-Deployment/blob/main/images/2_JULES_run.png?raw=true)

Check the output:
`cd $RSUITE/output`
`ls -l`
You should see something like these (yy is my name):
`yy.Base.asc`
`yy.dump.19961231.82800.asc`
`yy.dump.19971231.82800.asc`

Running a suite through rose seems to be the recommended way. Unfortunately it does not work for me in this case, possibly due to the absence of correct netcdf module paths. However, the CWS IT guys informed me that I should not modify any 
