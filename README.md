# A tutorial of deployment of JULES model on NCI accessdev

This is a brief tutorial of the deployment of the Joint UK Land Environment Simulator (JULES) model on NCI accessdev server.

## 1. Register essential accounts
As a prerequisite, the user needs to register two accounts, including:

1. NCI account: https://my.nci.org.au/mancini/signup/0

2. MOSRS account: http://jules-lsm.github.io/access_req/JULES_access.html

Both registrations are free, but the Met Office Science Repository Service (MOSRS) registration might take longer to get back to you as it is managed by the UK government rather than NCI. Once the user has these two accounts, please log into the NCI account (https://my.nci.org.au/mancini/login?next=/mancini/) and request the membership of the 'access' group.

![enter image description here](https://github.com/yuyi13/JULES-Deployment/blob/main/images/1_find_project.png?raw=true)

Once you got these two accounts and the membership of access group ready, you can log into the NCI accessdev server through a terminal. For windows users, I recommend the MobaXTerm (https://mobaxterm.mobatek.net/download-home-edition.html) which is a free terminal software. For MacOS and Linux users, please just directly use the system terminal.

## 2. Setup your accessdev account

Now use the terminal log into accessdev:\
`ssh username@accessdev.nci.org.au`

On acessdev run\
`mosrs-setup`\
to setup a GPG agent (a program that stores your passwords).\
*Note: you must already had a MOSRS account to run this setup.*

Then log out and back in again, and run\
`mosrs-auth`\
to save your MOSRS password into the agent. The password will be saved for 24 hours, so you'll need to run it again each day, however it's generally only required to check out a new configuration, not to run one that's already downloaded.

To be able to see the status of jobs Rose+Cylc needs to be able to SSH to and from Gadi
Run the script `/g/data/hh5/public/apps/nci_scripts/accessdev-gadi-setup` on accessdev to have this set up automatically.

If you fail at this step, it is likely to be due to the issue of ssh keys. Please refer to: https://accessdev.nci.org.au/trac/wiki/gadi **SSH Setup** section.

## 3. Run JULES WITHOUT rose
In order to run a JULES model, you need three softwares namely cylc, rose, and fcm. At the NCI accessdev server we do not need set up them as they were already been pre-installed.

**3.1 Download the JULES model and compile it**

Now create a directory under the home directory called 'MODELS', where you can put different versions of operational models. In this tutorial we are going to download a standard version of JULESvn6.1:
```
mkdir ~/MODELS
cd ~/MODELS
fcm co fcm:jules.x_tr@vn6.1 jules-vn6.1 # download JULES model vn6.1 from MOSRS repository
export JULES_ROOT=~/MODELS/jules-vn6.1
cd $JULES_ROOT/etc/fcm-make
vi make.cfg
```

Modify the line 22 of the `make.cfg` file from\
`$JULES_PLATFORM{?}        = custom`\
to\
`$JULES_PLATFORM{?}        = ceh`

Compile the JULES model \
```
cd $JULES_ROOT
fcm make -j 2 -f etc/fcm-make/make.cfg --new
```

**3.2 Download a suite**

Download a suite designed for tutorial purposes called 'u-cg242', created by Toby Matthews, which is a point run of a flux site (namely Loobos) located in Netherlands:
```
mkdir ~/roses
cd ~/roses
rosie checkout u-cg242 # copy a suite from the server
export RSUITE=~/roses/u-cg242
```

The location of the flux site:
![enter image description here](https://github.com/yuyi13/JULES-Deployment/blob/main/images/3_flux_site.png?raw=true)

**3.3 Run the model**

Now run the model. There are two ways to do this, with or without rose. The rose suite does not work for me at NCI accessdev. Therefore here I only introduce the approach to run the model through the command line.
```
cd ~/roses
mkdir nlists # create the name lists of pars and inputs for the model
cd nlists
rm *
rose app-run -i -C $RSUITE/app/jules
export NAMELIST=~/roses/nlists
```

At the path of `$NAMELIST`, modify (`vi`) the specific paths in all these four files: `ancillaries.nml`, `drive.nml`, `initial_conditions.nml`, and `output.nml`.

For example:
```
cd ~
pwd # This is to get your home directory
```

Copy your home dir, then\
```
cd $NAMELIST
vi ancillaries.nml
```

Then replace the `/home/users/tmarthews` at line 2 with your own home path. In my case, that is `/home/603/yy4778`. After that do the same things for `drive.nml`, `initial_conditions.nml`, and `output.nml`.

Now we can run the JULES model\
`$JULES_ROOT/build/bin/jules.exe $NAMELIST`

You should be able to see something like this:
![enter image description here](https://github.com/yuyi13/JULES-Deployment/blob/main/images/2_JULES_run.png?raw=true)

Check the output:
```
cd $RSUITE/output`
ls -l
```

You should see `.asc` files like these (Toby is the name of the suite author, which was used as an run id):
```
Toby.Base.asc # the main output
Toby.dump.19961231.82800.asc # snapshot for the start time step
Toby.dump.19971231.82800.asc # snapshot for the end time step
```

Use a simple `vi Toby.Base.asc` to check the attributes of the output:
```
#
# CREATED BY JULES LSM
#
# Global attributes:
#     latitude = 52.167999 ;
#     longitude = 5.7440000 ;
#
# Dimensions:
#     x = 1 ;
#     y = 1 ;
#     tile = 9 ;
#     type = 9 ;
#     pft = 5 ;
#     nt = 2 ;
#     time = UNLIMITED ;
#
# Variables:
#     time_bounds(nt,time) ;
#         long_name = Time bounds for each time stamp ;
#         units = seconds since 1996-12-31 23:00:00 ;
#
#     time(time) ;
#         standard_name = time ;
#         long_name = Time of data ;
#         units = seconds since 1996-12-31 23:00:00 ;
#         bounds = time_bounds ;
#         calendar = standard ;
#
#     pstar(x,y,time) ;
#         missing_value = -1.00000002E+20 ;
#         _FillValue = -1.00000002E+20 ;
#         coordinates = latitude longitude ;
#         long_name = Gridbox surface pressure ;
#         units = Pa ;
#         cell_methods = time : mean ;
#
#     .......
```

**NOTE**: Running a suite through rose is still the recommended way as there are many benefits in doing so. Unfortunately it does not work for me in this case, possibly due to the absence of correct netcdf module paths. Meanwhile, the CWS IT guys suggest that 

*''All the work of porting the codes and loading the right modules has been done for you when using JULES on NCI machines. But to benefit from that work, you need to run JULES via Rose and Cylc only.''*

Therefore there might be some ways to use JULES with GUIs on accessdev alone, but until now I have not found an approach to achieve this. Will keep investigating.

## Reference documents
Accessdev http://climate-cms.wikis.unsw.edu.au/Accessdev

JULES Documentation https://jules-lsm.github.io/

JULES Training Resources https://jules.jchmr.org/content/training

JULES Tutorial https://jules-lsm.github.io/tutorial/bg_info/tutorial_julesrose/index.html

Marthews, T., 2021. JULES From Scratch, available at: https://jules.jchmr.org/content/scratch

