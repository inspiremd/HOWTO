# Summit Software Notes



## Introduction

The goal of the Summit Software INSPIRE is to provide a consistent software stack
for BIP178 project members. Note however, we are not an authoritarian project and you can 
use whatever software you desire. 

## First steps

Run the following command:

** source /gpfs/alpine/proj-shared/bip178/Inspire_Project_Software/bin/summit_inspire_environmental_variables.sh **

This will set two key environmental variables and modify your module file path to make the
Summit INSPIRE binaries available. The 2 key environmental variables are
    
* INSPIRE_TARGET_MACHINE
* INSPIRE_PROJECT_SOFTWARE_TOP_LEVEL

## Loading Software

One should now be able to load the Summit INSPIRE software. Do the following command to see what
is available

** module avail Summit **


### Core Software Stack

The core software sets the environment for the gcc compiler, swig, python, and other software
to build and run in an consistent environment.

To load the core software stack do

** module load Summit/inspire_project_environment **

This command need only be done once.

### Production Software

To load a production software load the  core software module then load the desired module. For
example to load AutoDock Vina do the following

** module load Summit/inspire_project_environment **  
** module load Summit/autodock_vina/1.1.2 **

Recall that one need only to load the core software stack once.