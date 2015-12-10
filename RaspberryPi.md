
# Install notes

Based on <http://prjemian.github.io/epicspi/>  
  This assumes you have a clean install of raspbian (I'm using 2015-05-05)

## EPICS Base

-   Boot into raspbian and make sure you have a network connections

-   Log in as user: pi

-   Set up the date (if it's wrong!)
    
        sudo dpkg-reconfigure tzdata

-   Make a new user: enge (make whatever you want for the password)
    
        sudo adduser --home /home/enge enge

-   Add that user to the sudoers list
    
        sudo adduser enge sudo
        sudo adduser enge dialout

-   Log out and then in as that user

-   Make the file structure
    
        mkdir bin
        mkdir extensions
        mkdir project
        mkdir synApps

-   Download and untar the EPICS base package
    
        wget http://www.aps.anl.gov/epics/download/base/base-3.15.3.tar.gz 
        tar -xzvf base-3.15.3.tar.gz
        ln -s base-3.15.3 base

-   Paste the following into .bashrc
    
        ######################################################################
        ##  EPICS
        ######################################################################
        
        ## Base
        export EPICS_ROOT=/home/enge
        export EPICS_BASE=${EPICS_ROOT}/base/
        export EPICS_HOST_ARCH=`${EPICS_BASE}/startup/EpicsHostArch`
        export EPICS_BASE_BIN=${EPICS_BASE}/bin/${EPICS_HOST_ARCH}
        export EPICS_BASE_LIB=${EPICS_BASE}/lib/${EPICS_HOST_ARCH}
        if [ "" = "${LD_LIBRARY_PATH}" ]; then
            export LD_LIBRARY_PATH=${EPICS_BASE_LIB}
        else
            export LD_LIBRARY_PATH=${EPICS_BASE_LIB}:${LD_LIBRARY_PATH}
        fi
        export PATH=${PATH}:${EPICS_BASE_BIN}
        
        ## EPICS Extensions
        export EPICS_EXT=${EPICS_ROOT}/extensions
        export EPICS_EXT_BIN=${EPICS_EXT}/bin/${EPICS_HOST_ARCH}
        export EPICS_EXT_LIB=${EPICS_EXT}/lib/${EPICS_HOST_ARCH}
        if [ "" = "${LD_LIBRARY_PATH}" ]; then
            export LD_LIBRARY_PATH=${EPICS_EXT_LIB}
        else
            export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:${EPICS_BASE_LIB}
        fi
        export EPICS_SYNAPPS_BASE=${EPICS_ROOT}/synApps
        export EPICS_SYNAPPS_BIN=${EPICS_SYNAPPS_BASE}/support/utils
        export PATH=${PATH}:${EPICS_EXT_BIN}:${EPICS_SYNAPPS_BIN}

-   Load it
    
        source ~/.bashrc

-   Compile EPICS
    
        cd base
        make -j3

-   Buy and drink some coffee

-   Once finished, check that it works
    
        softIoc

-   Epics should have started. Now run the IOC
    
        iocInit

-   You should see something that looks like
    
        epics> iocInit  
        Starting iocInit
        ############################################################################
        ## EPICS R3.15.3 $Date: Thu 2015-05-14 14:09:28 +0200$
        ## EPICS Base built Jul 17 2015
        ############################################################################
        iocRun: All initialization complete
        epics> exit

## synApps (needed for streamdevice and serial connections)

-   Get the extensions and msi first
    
        cd ~
        wget http://www.aps.anl.gov/epics/download/extensions/extensionsTop_20120904.tar.gz
        tar -xzvf extensionsTop_20120904.tar.gz
        wget http://www.aps.anl.gov/epics/download/extensions/msi1-7.tar.gz
        cd extensions/src
        tar -xzvf ../../msi1-7.tar.gz
        cd msi1-7
        make

-   Install re2c (I don't know what it's for)
    
        sudo apt-get install re2c

-   Download and unzip synApps
    
        cd ~
        wget http://www.aps.anl.gov/bcda/synApps/tar/synApps_5_8.tar.gz
        tar -xzvf synApps_5_8.tar.gz
        ln -s synApps_5_8 synApps

-   We don't need all the junk included
    
        cd synApps/support/configure
        emacs RELEASE

-   Edit the SUPPORT line
    
        SUPPORT=/home/enge/synApps/support

-   Edit EPICS<sub>BASE</sub>
    
        EPICS_BASE=/home/enge/base

-   Comment out (with a '#') the modules we don't want
    
    -   `ALLEN_BRADLEY`
    
    -   `AREA_DETECTOR`
    
    -   `ADCORE`
    
    -   `ADBINARIES`
    
    -   `CAPUTRECORDER`
    
    -   `CAMAC`
    
    -   `DAC128V`
    
    -   `DXP`
    
    -   `IPUNIDIG`
    
    -   `OPTICS`
    
    -   `QUADEM`
    
    -   `SOFTGLUE`
    
    -   `VME`

-   Prepare the makefile
    
        cd ~/synApps/support
        make release

-   Compile!
    
        make -j3 rebuild

## Tidy up

-   Make a folder to keep zip files
    
        cd ~
        mkdir download
        mv *.tar.gz download
