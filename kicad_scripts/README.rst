kicad_scripts
=============

This is a set of scripts for either Kicad or FreeCAD or both.
These scripts are highly unstable and comes with no warranty whatsoever.

configure_kicad.sh
------------------
This script will pull all the official kicad libraries for Github + the libraries located in osrf_hw repository. 
- it will create a kicad workspace and copy all the relevant files in a resource folder
- Pull the kicad-library-utils repo and install in on your machine
- Populate the ~/.config/kicad files to use this workspace architecture properly, add all the footprint libraries and add OSRF plugins like editSch.py
- It will add the KISYS3DMOD environment variable to your bashrc to allow a better crossmachine Kicad to Freecad export
Note: this script will try to pull private repositories (3d models from manufacturers which are under copyright) so you'll just have to skip this step

cleanBeforeCommit.sh
--------------------
- delete all the kicad temporary files (.bak .bck etc)
- make sure that the custom libraries are referenced relatively and not with absolute paths
- remove cache.lib references from your Schematic files
TODO:
*****
- reorganize library order in the project file (.pro) to be sure that for any duplicated library the OSRF version will override the default kicad library

editSch.py
----------
This is a PyQt application parsing schematic files and rendering them as a spreadsheet.

You can then modify any parameter of any component and populate your Sch file with these new data.

All the features/shortcut are explained in the help window of this application that you can open by hitting the '?' key.


NOTE: this script assumes that all the components of your schematic have all the parameters defined in OSRF libraries (MFN,MFP,D1,D2,D1PN,D1PL,D2PN,D2PL,Package,Description,Voltage,Power,Tolerance,Temperature,ReverseVoltagemForwardVoltage,Cont.Current,Frequency,ResonnanceFreq). These fields can have dummy values, but they shouldn't be empty otherwise kicad will delete them during the saving/export phase.

All these parameters are used to fill automatically the fields necessary to generate a full BOM to give to our fabhouse. 

If you hit the 'DB Autofill' button, your entire spreadsheet will be parsed and the script will try to find your components in the database. If there is any missing field they will be highlighted in red, otherwise they'll be highlighted in green. This allows to see easily what is missing and what fields are necessary to populate the information of a given component.

Note2: If you ran the configure_kicad.sh script before, this script should be available in the BOM window of eeschema (BOM icon in the eeschema toolbar). If so you will have 2 options:
- editBOM_SCH_PyQt: to open the spreadsheet and modify whatever you want
- generateBOM: to create a csv file based on the values in your sch file

Note3: This script is using the python classes from kicad-library-utils, they may not be found properly so you'll have to make sure that python can import them

generate_component.py
---------------------
This script allows you to generate schlib files automatically.

You have to feed it a configuration file like the configExample.txt file stored in the same folder.

You can also feed it a pinout file in csv format to generate a component with all the pin names and number you need (you can refer to pinoutFileExample.csv or PinoutFileExampleBGA.csv).

The help string shows all the different configuration you can have to represent a component/pin numbering.

If the destination library specified already exist the component will be added to that library (using kicad-library-utils/schlib/move_part.py).

The resulting files should be KLC compliant but may need a bit of manual tweaking for aesthetics

generate_multipart_component.py
------------------------------
This script is based on the generate_component.py script. This one allows you to create multipart component which is very useful for big chips like FPGAs.

This script expect a configuration file (e.g. configXilinx.txt). This configuration file should have the PinoutFile variable populated, the entire script will rely on the content of this pinout file, you can refer to xc7a50tcsg324pkg.csv for reference.
This csv file is a truncated version of the official Xilinx pinout files.

The script will parse the pinout file and create a different component part for each bank. this way you can generate convenient, smartly splitted components for your project instead of having a gigantic thousand-pin symbol on a single sheet.


generate_BGA_footprint.py
-------------------------
Script generating kicad_mod files automatically.

This script relies on using the IPC-7351 Naming Convention for Ball Grid Array’s (BGA)(http://www.pcb-3d.com/models/sm-standard/bga).
The resulting file should be KLC compliant.

Please refer to configFootprintBGA.txt to see the information needed in the configuration file

freecad_gen_BGA.py
-------------------
This is a script runned by the FreeCAD python console.

It assumes that you are using the IPC-7351 Naming Convention for Ball Grid Array’s (BGA)(http://www.pcb-3d.com/models/sm-standard/bga).

Actions performed by the script:
 * Create a BGA 3D model matching the filename provided (e.g. BGA900C80P30X30_2500X2500X150.wrl).
 * Export it as a STEP file
 * Scale it down to inches
 * Export it as a WRL file in the same desitnation folder


NOTE: all the following scripts have their equivalent in the footprint wizard inside kicad. If you prefer using the GUI and modify the footprint in the editor, check the folder kicad_plugins rather than these scripts

generate_standard_IC.py
-----------------------
This is a global script to generate footprint for all standard packages based on IPC-7351B naming convention.

It can generate footprints for BGA,QFN,QFP and SOP/SOIC packages. It shoud be soon generating SON packages too.

This script uses the functions stored in generate_standard_footprints.py.

Many things in these functions are highly experimental and it hasnt been tested on a big variety of packages.

One thing that may need to be tweaked by hand is the size of the pads given that it is not specified in the IPC-7351 naming convention.

This script also calls freecad python scripts to generate 3d models. Only the BGA models can be generated for now.

generate_standard_footprints.py
-------------------------------
Set of functions to generate strings matching the kicad_mod file format. Only a few functions are available for now and have not been tested properly.These functions are called by the generate_standard_IC.py script

How to use it
*************
#TODO update this readme to match latest script features
Edit the freecad_gen_BGA.py file l.18 and give the absolute path of where you want your 3d file to be created. 

Please provide a filename matching the BGA chip naming convention.
It can be runned by launching `freecad freecad_gen_BGA.py`

generate_BGA.sh
---------------
This script parse config file and calls sequentially generate_BGA_footprint.py and freecad_gen_BGA.py.
This it creates the footprint and the asociated 3d model that can be displayed directly in kicad.
to run it just launch: `./generate_BGA.sh <configFile>`. You can look at configGlobalBGA.txt for an example of valid config file.

KiCADToFreeCAD
-------------
This folder is a place holder for the stepUp script and configuration files created by Maurice: https://forum.kicad.info/t/kicad-stepup-new-exporter-for-3d-mcad-feedbacks-are-welcome/1048 
No modifications or improvement have been braught to this script, except that you don't need to provide the paths to your 3d models if you have the environment variable KISYS3DMOD defined, they are stored here for a matter on convenience.

How to export kicad boards to FreeCAD ?
*************************************
- in pcbnew (kicad pcb editor) export your board as Idfv3
- copy the content of KiCADToFreeCAD folder to your project directory
- modify ksu-config.cfg to put the name of your 3D model directory
- run `freecad <ProjectName>.emn ksu-config.cfg kicad_StepUp.FCMacro`

NOTE: This assumes that you have all your 3d models in STEP format in mm for Freecad and in WRL format in inches for Kicad

Requirement:
- Freecad 0.15 or higher
