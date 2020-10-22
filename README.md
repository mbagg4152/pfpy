# pfpy - Protein & Flavonoid project code in Python
Former repository for CHEM-491 (Research Methods) class and is now being used in continuing the project.  
The goal was to modernize, clean and adapt a former student's thesis project.  


---
## Original Code Information
> *KEGG.py, Made using Python 2.7  
Originally Created on Wed Apr 17 2019. @author: vmoorman and Jordan Wilson    
JW started the code to get information from KEGG about the species we were interested in - April 2019*  
---
## Requirements
1. Make sure to have Python 3 installed on your computer.   
&emsp;For example, in Linux, you can check the version using:  
&emsp; ```user@computer:~$ python -V```  
&emsp; ```Python 3.8.3```  
&emsp; Windows now has Python 3 on the Microsoft store, so checking the current version could be done by finding the list of installed  
&emsp; programs by navigating to ```Control Panel --> Programs --> Uninstall a Program```

1. Install python package __bioservices__: ```[pip|pip3] install bioservices```  
&emsp; _Using pip should work, but a system may recognize ```pip3``` instead._   
&emsp; You can find the bioservices documentation [_here._](https://bioservices.readthedocs.io/en/master/)
1. Install python package __BioPython__: ```[pip|pip3] install biopython```  
&emsp; You can find the BioPython documentation [_here._](https://biopython.org/wiki/Documentation)
---
## Project Structure
> ```pfpy``` main folder  
```pfpy/projects/original/``` contains the code from when Jordan Wilson was developing  
```pfpy/projects/current/``` current code
```pfpy/projects/current/flavonoid``` code (and related files) for flavonoid prediction  
```pfpy/projects/current/protein``` code for the protein-flavonoid interaction project
```pfpy/projects/current/flavonoid/data/``` output data folder  
```pfpy/projects/current/flavonoid/data/Chemical_Data``` contains the files for the list of plants predicted per flavonoid  
```pfpy/projects/current/flavonoid/data/FASTA_Data``` contains FASTA EC number data  
```pfpy/projects/current/flavonoid/data/Gene_Data``` contains the data pulled from KEGG for each plant pathway  
```pfpy/projects/current/json_data```  holds the lists of plants & pathways used in the KEGG program (in JSON format).  
```pfpy/projects/current/lib``` contains the library/helper code and other assorted test code files.  
```pfpy/projects/current/misc_output``` contains output from testing programs not used by kegg-prog.py  
  
---
# Important Files and Functions in ```pfpy/projects/current/```
__Note:__ Not all files or functions are covered below.  


## ```flavonoid/kegg-prog.py```
This is the main program of the code project.  

#### ```gene_pathway_data```
This function is called and passed a pathway for a specific organism (e.g. adu00941). For each pathway that is passed 
in, the code uses the kegg plugin in order to get the gene data for the specific pathway, which is done using the 
following lines:  
```python
    raw = kegg.get(pathway_id)
    gd = kegg.parse(raw)
    ...
    fetched_genes = gd.get('GENE')
```
Once the gene data is collected,the data is split up into different list items. For reference, a single gene entry for 
cam00941 is shown below (the other entries in 'GENE' take similar form):  
```python
'GENE': {'101489106': 'chalcone synthase 1 [KO:K00660] [EC:2.3.1.74]', ...}
```
Then the data is added to a list such that this gene data is associated with the appropriate plant.


---
## ```lib/compoundinfo.py```
This contains the labeled EC numbers as well as the logic used in order to make the predictions.  
Some of the EC number variables are shown below:
```python
E1 = 'EC:4.3.1.24'
E2 = 'EC:4.3.1.25'
...
E17_2 = 'EC:1.1.1.219 1.1.1.234' # unique number for bifunctional dihydroflavonol 4-reductase/flavanone 4-reductase
...
E26 = 'EC:2.4.1.136'
```

### ```or_in```
Takes in a list and elements and returns true if at least ONE element is present in the list.

### ```and_in```
Takes in a list and elements and returns true only if all of the passed elements are in the list.

### ```The Logical Functions```
Different logical functions have been written not only for the flavonoids of interest, but also for the prerequisite 
compounds which are found on the map. The prerequisite functions are used to get the total result for the specific 
compound. If the prerequisite returns ```False```, the compound logic function will also return ```False```.  
  
Each function named using the compound's PDBj ID (or abbreviation, if no ID is available) in order to keep code lines 
at a decent length. Each function does have its compound's full name commented at the end of its respective line. 
For example, the function for Cinnamic acid, which requires ```EC:4.3.1.24``` OR ```EC:4.3.1.25``` is written as such:

```python
def tca(e): return or_in(e, E1, E2)  # cinnamic acid
```
The logical functions are used in the function ```finish_up``` in ```kegg-prog.py```. 
For each plant's total EC list, 
the program will loop through each of the flavonoids' logical requirements, which are held held in the list of 
```ChemData``` objects called ```data_lists```, where each item ```chem_data``` has a property ```chem_data.label``` 
that is passed to a function in ```compoundinfo.py``` called ```flav_check```, which then determines the logical 
function to be called. 
If the result of ```flav_check``` returns as true, then the current plant's name will be appended
 to the list of plants, which is held in ```chem_data.species```. 
The functions return ```True``` or ```False``` based on whether or not the required EC numbers are in the list 
(parameter ```e```) which was passed to ```flav_check```.    

---
## ```lib/knapsackinfo.py```
This program uses the ```wget``` command in order to pull the information for each of the species from KNApSAcK. 
For each plant, it compiles a list of entries from the database if the entry line contains one of the flavonoids of 
interest.  
After the HTML page for each plant is retrieved, the file is then parsed.
Any lines containing the names of the flavonoids of interest will be saved, stripped of HTML syntax and broken down into 
a list of lists for the plant.
This program was written with the purpose of making data collection for both the compounds and their relatives easier.  
More searching will be done in order to determine how easy or difficult it would be to update this code to work with 
other databases.

__Note:__ It will not work correctly if the ```wget``` system command is not installed (this has not yet been tested
on Windows, may potentially only be compatible with Linux systems). 



---
## ```lib/datatypes.py```
This contains the custom data types that are or have been used in the program.  

### ```ChemData```
This object type holds a flavonoid label, a file name and a list of species.  
A list of ```ChemData``` objects is used in ```kegg-prog.py``` in order to keep track of which plants 
were predicted for each flavonoid.

### ```Species```
This object has a species name, a list of predicted flavonoids and the number of associated flavonoids.  
The list of ```Species``` is populated in the same block of code as the list of ```ChemData``` objects.  
This object is better suited for focusing on specific plants based on prediction counts and predicted flavonoids.

---
## ```lib/jsondata.py```
This file calls the ```get_json_data(filename,key)``` function from util.py, which reads in the list of plant and 
pathway codes as well as the file containing the scientific name for each plant and the full name of each pathway map.




---
## ```lib/pathstrings.py```
This file just contains the strings which hold the dedicated output folder and file names.



---
## ```lib/util.py```
This file contains various utility functions used throughout the program.



#### ```get_json_data```  
Reads in a JSON (JavaScript Object Notation) file and converts the data into usable python variables.




#### ```remove_dupes```   
Removes duplicate elements from a list.



#### ```write_readme```  
Writes the program's original ```README``` file.


---
## Original Changelog 
Version | Change |
:-------|:-------|
<sub>v0p1 | <sub>VRM getting the iteration and parsing part of the code to work - April 2019 |
<sub>v0p2 | <sub>VRM making the count code work - April 2019 |
<sub>v0p3 | <sub>JW getting the file output to work, revising the species codes and adding a dictionary - April 2019 |
<sub>v0p4 | <sub>JW creating directory change function for Gene_data and adding additional plant species - multiple things still broken - May 2019 |
<sub>v0p5 | <sub>VRM cleaned up some duplicate issues, folder locations, and ensured that species are written out in full  - May 2019 |
<sub>v0p6 | <sub>JW creating function for master FASTA file (error occurs when running every entry) - May 2019 |
<sub>v0p7 | <sub>JW editing lists for master fasta function and updating fasta function - May 2019 |
<sub>v0p8 | <sub>VRM fixing and creating gene list for master fasta file - June 2019 |
<sub>v0p8p2 | <sub>JW made a fasta file for each EC#- June 2019 |
<sub>v1p0 | <sub>VRM cleaned up code |
<sub>v1p1p1 | <sub>JW added ReadMe |
<sub>v1p2 | <sub>JW added function to get plants that make Epicatechin - incorrectly |
<sub>v1p3 | <sub>VRM coded in epicatechin, catechin, eriodictyol, luteolin, naringenin listed in Chemical_Data --- leutolin actually wrong |
<sub>v1p3p1 | <sub>VRM tried to fix leutolin |




