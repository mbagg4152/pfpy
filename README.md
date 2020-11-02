# pfpy - Protein & Flavonoid project code in Python
Former repository for CHEM-491 (Research Methods) class and is now being used in continuing the work for said project 
in the context of a thesis project and a research assistant position. The initial goal was to modernize, clean and add 
additional flavonoids to the code.    
The original code and the version that was worked on during class mainly focused on utilizing KEGG data in order to 
predict whether or not a specific plant could be capable of synthesizing selected flavonoids.  
The new code being developed (located in ```projects/current/protein```) is being developed for the purpose of studying 
and analyzing the interactions between proteins and flavonoids or flavonoid-like compounds.

<!--#################################################################################################################-->
---
## Original Code Information
  > *KEGG.py, Made using Python 2.7  
  Originally Created on Wed Apr 17 2019. @author: vmoorman and Jordan Wilson    
  JW started the code to get information from KEGG about the species we were interested in - April 2019*  

<!--#################################################################################################################-->
---
## Requirements
  - Make sure to have Python 3 installed on your computer.   
    * For example, in Linux, you can check the version using:  
    ```user@computer:~$ python -V```  
    ```Python 3.8.3```  
    * Windows now has Python 3 on the Microsoft store, so checking the current version could be done by finding 
    the list of installed  programs by navigating to ```Control Panel --> Programs --> Uninstall a Program```

  - Install python package __bioservices__: ```[pip|pip3] install bioservices```  
    * _Using pip should work, but a system may recognize ```pip3``` instead._   
    * You can find the bioservices documentation [_here._](https://bioservices.readthedocs.io/en/master/)

  - Install python package __BioPython__: ```[pip|pip3] install biopython```  
    * You can find the BioPython documentation [_here._](https://biopython.org/wiki/Documentation)




<!--#################################################################################################################-->
---
## Project Structure
- ```pfpy``` main folder  
- ```pfpy/projects/original/``` contains the code from when Jordan Wilson was developing  
- ```pfpy/projects/current/``` current code
- ```pfpy/projects/current/flavonoid``` code (and related files) for flavonoid prediction  
- ```pfpy/projects/current/protein``` code for the protein-flavonoid interaction project
- ```pfpy/projects/current/flavonoid/data/``` output data folder  
- ```pfpy/projects/current/flavonoid/data/Chemical_Data``` contains the files for the list of plants predicted per 
                                                            flavonoid  
- ```pfpy/projects/current/flavonoid/data/FASTA_Data``` contains FASTA EC number data  
- ```pfpy/projects/current/flavonoid/data/Gene_Data``` contains the data pulled from KEGG for each plant pathway  
- ```pfpy/projects/current/json_data```  holds the lists of plants & pathways used in the KEGG program (in JSON format).  
- ```pfpy/projects/current/lib``` contains the library/helper code and other assorted test code files.  
- ```pfpy/projects/current/misc_output``` contains output from testing programs not used by kegg-prog.py    

<!--#################################################################################################################-->
---
# Important Files and Functions in ```pfpy/projects/current/```
__Note:__ Not all files or functions are covered below.  

## ```flavonoid/keggv1.py```
The original main file of the program for the flavonoid prediction portion of the project.  
 
## ```flavonoid/keggv2.py```
The current main file of the program for the flavonoid prediction portion of the project.  
  
#### How it works
<ol>
  <li><p>A list of plant pathways is built using the plant codes and the path map codes. For example, gmx00941 would refer to the flavonoid biosynthesis 
    pathway for Glycine max (soybean).</p></li>
  <li><p>After the list is built, the program then makes a call to <code>KEGG.get()</code>, which retrieved the specific entry the pathway.</p></li>
  <ol>
    <li><p>Each entry contains important information, for the plant's genes for the specfic pathway.</p></li>
    <li><p>For each gene, a line is saved with the format: <code>Species Name, Gene ID, Compound Name, EC number(s), Orthology Code</code></p></li>
  </ol>
  <li><p>After getting the gene entries for each plant pathway (which is saved in <code>data/Gene_Data</code>, each EC number is added to a list for
    each <code>Plant</code> object. These objects also contain information such as the plant name, KEGG code and list of genes.</p></li>
  <li><p>Now that each plant has an associated list of EC numbers, the flavonoid prediction process can be done. This is done by checking
    the list of EC numbers for each plant and determining whether or not it has the correct EC numbers needed to produce a flavonoid. 
    If a plant does meet the requirements, then the plants name is added to the list of plant for that specific <code>ChemData</code> 
    (flavonoid) object.</p></li>
    <ol>
    <li><p>Once the prediction process is done, each result is printed to a file named after the respective flavonoid and then the program
      displays the number of plants for each flavonoid as well as the path to the output file.</p></li>
  </ol>
  <li><p>Once the prediction process is done, the code then moves on to create a master count matrix/list. This list (and output file)
    contain the number of times each EC number appears in the gene entries for each plant. When checking for each plant and each EC
    number, the list of <code>EcCounts</code> for each plant will increment the <code>self.count</code> property each time an EC number is
    encountered. Once the list is filled, the program outputs the counts with the format <code>Species Name: [EC:#.#.#.# count #] ...</code> </p></li>
  <ol><li><p>If you look at the code, you may notice that there is no effort made to prevent adding duplicate EC numbers to the list for each plant, 
    this is why. This is the easiest way to keep track of the counts.</p></li></ol>
  <li><p>The final step is to build the master FASTA files and the FASTA files organized by EC number. For each gene that is encountered,
    its list of EC numbers, Gene ID and KEGG code are looked at. Using the partial DBGET URL, <code>KEGG_code:Gene_ID</code> is appended to
    the end to create the full REST URL. The program saves the HTML, parses it, then adds the FASTA entry to its own  
    <code>FastaEcEntry</code> object which is then added to a list for its appropriate <code>EcFastaCollection</code> object, which is determined
    by EC number. Once the parsing is done (which does take a long time to finish) then for each <code>EcFastaCollection</code> object, 
    a file is written to and the output is saved to a massive string, which is written to the MasterFASTA file.</p></li>
</ol>
  
  
  
#### ```path_parse```
This function is called and passed a pathway for a specific organism . For each pathway that is passed 
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


## ```flavonoid/keggv2_with_libs.py```
This file is a 'condensed' version of  ```keggv2.py```. Instead of relying on the external imports from the ```lib``` 
directory, all of the important variables, functions, etc. are located in within the main file. __Note:__ the  
```json_data``` directory is still needed for the program to work correctly.    
This file was made more for convenience since imports can sometimes behave strangely, plus this decreases the number of 
code files needed to accomplish the same tasks.  
When continuing development, if you wish to keep a condensed version, it is highly suggested to work on the main and 
library files first, then build the condensed version.   
It should also be noted that currently, the condensed program does take longer to finish than the modular one. 


<!--#################################################################################################################-->
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
  
### The Logical Functions  
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

<!--#################################################################################################################-->
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

<!--#################################################################################################################-->
---
## ```lib/datatypes.py```
This contains the custom data types that are or have been used in the program.  
### ```ChemData```
This class holds the data for each flavonoid. The objects are initialized with their file name and label and only
later in the program, their empty list of plants will be filled.  
__ATTRIBUTES__    
```self.label```: string that contains the flavonoids name  
```self.plants```: list of plants predicted to produce the flavonoid  
```self.file_name```: string that holds the flavonoids output file name  
__FUNCTIONS__  
```__init__```: constructor for the object  
```__eq__```: defines equality of the object  
```is_in```: determines if an identical or nearly identical object is already in the list  
### ```Plant```
This object holds information about each plant used in the program. The plant objects are initialized with their
scientific name and their code and then have different information added later.  
__ATTRIBUTES__    
```self.name```: scientific name of the plant  
```self.code```: KEGG code for the plant  
```self.genes```: the gene entries for the plant  
```self.ec_nums```: the EC numbers parsed from the plants gene entries  
```self.flavonoids```: the list of flavonoids that the plant could potentially produce  
```self.ec_counts```: list of objects that hold the number of times each EC number appears  
__FUNCTIONS__  
```__init__```: constructor for the object  
```__eq__```: defines equality of the object  
```is_in```: determines if an identical or nearly identical object is already in the list  
```has_ec_count```: used to determine whether or not a specific EC number is already in the list of EC counts  
```incr_ec_count```: used to increase the count for the EC count objects.  
### ```PathGene```
This object is used to hold Gene objects in a way such that they are sorted by the pathway from which they were
found.   
__ATTRIBUTES__  
```self.path```: the pathway that resulted in the gene entry   
```self.genes```: the list of gene entries from this pathway   
__FUNCTIONS__   
```__init__```: constructor for the object   
```__eq__```: defines equality of the object   
```is_in```: determines if an identical or nearly identical object is already in the list   
### ```Gene```
This object holds data gathered from KEGG for each plant's pathway (like aip00491).  
ATTRIBUTES  
```self.gene_id```: the ID of the gene from a plant  
```self.plant```: the scientific name of the plant that has this gene  
```self.plant_code```: the KEGG code for the plant  
```self.compound```: the compound name listed in the entry  
```self.ec_nums```: the list of EC numbers found in the entry  
```self.ortho```: the KEGG orthology code for the compound  
```self.path```: the pathway where the gene was found  
__FUNCTIONS__  
```__init__```: constructor for the object  
```__eq__```: defines equality of the object  
```is_in```: determines if an identical or nearly identical object is already in the list  
```simple```: returns a formatted string that contains information from the object  
```no_plant```: same as simple, but without including the plant name  
### ```EcFastaCollection```
This object is used to hold the associated FASTA entries for any given EC number.  
__ATTRIBUTES__  
```self.ec_name```: the EC number & name used when writing the file  
```self.ec_entries```: the list of associated FASTA entries (FastaEcEntry objects)  
__FUNCTIONS__  
```__init__```: constructor for the object  
```__eq__```: defines equality of the object  
```is_in```: determines if an identical or nearly identical object is already in the list  
### ```EcCounts```
This object is a property of the Plant object and is used to hold each EC number and the number of times it occurs
in gene entries of a given plant.  
__ATTRIBUTES__  
```self.number```: the EC number  
```self.count```: number of times that the EC number shows up in gene entries.  
__FUNCTIONS__  
```__init__```: constructor for the object  
### ```FastaEcEntry```
This object is a property of EcFastaCollection and contains the information for a specific FASTA entry.  
__ATTRIBUTES__  
```self.gene_id```: the gene ID associated with the sequence  
```self.plant```: the plant that the gene is from  
```self.dna_seq```: the dna sequence/FASTA entry for the specific gene  
__FUNCTIONS__  
```__init__```: constructor for the object  
```__eq__```: defines equality of the object  
```is_in```: determines if an identical or nearly identical object is already in the list  
```simple```: returns a formatted string  

<!--#################################################################################################################-->
---
## ```lib/jsondata.py```
This file calls the ```get_json_data(filename,key)``` function from util.py, which reads in the list of plant and 
pathway codes as well as the file containing the scientific name for each plant and the full name of each pathway map.

<!--#################################################################################################################-->
---
## ```lib/pathstrings.py```
This file just contains the strings which hold the dedicated output folder and file names.

<!--#################################################################################################################-->
---
## ```lib/util.py```
This file contains various utility functions used throughout the program.

#### ```get_json_data```  
This function uses the python JSON library in order to parse JSON files into usable python objects. Can return
lists or dictionaries, depending on the JSON file's structure.  

#### ```remove_dupes```   
Removes duplicate elements from a list.  

#### ```list_partition```
This function takes in a list and then splits it into as many parts as specified by parameter ```num```.  

#### ```write_readme```  
Writes the program's original ```README``` file.

<!--#################################################################################################################-->
---
## ```protein/protein.py```
This program reads in a ```JSON``` file of PDB IDs and then appends the IDs to the end of a specific URL in order to 
download each desired PDB or mmCIF file. 
When a PDB ID + the desired file extension (```.pdb```, ```.xml```, ```.cif```) is appended to the simple partial 
URL ```https://files.rcsb.org/view/```, the program can then call ```urllib.request.urlretrieve(url, file_path)```, 
where ```file_path``` is the name of the file that  ```urllib``` will save the web page content to.   
For example, if the current iteration is looking at ID 4V4D (large structure), the code would get the mmCIF file using 
the URL ```https://files.rcsb.org/view/4V4D.cif```.  
If a given file can be found in ```protein/output/pdb_files```, then the program will skip the download process and go 
straight to parsing the file's information.   
If ```urllib``` receives an error code from trying to get a .pdb file, that is most likely due to PDB not providing 
.pdb files for large structures. If an error is occurred, then the program attempts to fetch the appropriate .cif file 
and then attempt to convert the file to .pdb format.


<!--#################################################################################################################-->
---
## ```protein/Types.py```
This file contains two different classes and functions that are used to create new objects, which are used in 
```protein.py```.  
#### ```Record```
This object holds the information from PDB files for a single ```ATOM/HETATM``` line or record.  
For example, the following lines would be appropriately converted into ```Record``` objects using the function 
```new_record```:
```
ATOM   1258  CA  THR B  59      22.806  24.345  36.922  1.00 23.83           C 
HETATM 1815  O   HOH A 133      17.558  28.943  -4.426  1.00 23.32           O  
```
#### ```new_record```
In ```protein.py```, if a line begins with ```ATOM/HETATM``` then this function is called to create a new ```Record``` 
object. The function requires that the line of the file along with the PDB ID be passed as parameters.  
Since PDB files have dedicated column ranges for each value, it is then easy to assign the new object's properties with 
values from the passed in line.

#### ```Entry```
These objects contain data from the PDB files themselves, not just simple lines. Each ```Entry``` contains specific 
information such as PDB ID, classification, a list of ```Records```, associated organisms, EC numbers, etc.  

#### ```new_entry```
This function takes the PDB file as a list of lists and based on the value at the beginning of the line (HEADER, 
SOURCE, etc.) will parse the information and assign the parsed values to the object's properties. After filling the 
available properties, a new ```Entry``` is returned.

<!--#################################################################################################################-->
---
## ```protein/StringsAndConsts.py```
This file simply contains several strings & constant values for ```protein.py```.


<!--#################################################################################################################-->
---
## Notes for Continuing Development
### IDEs and Text Editors
- If you are continuing the work on this file, I would HIGHLY recommend using a Python IDE (integrated development 
environment). 
    * I used PyCharm CE (community edition, free) for this but other IDEs or advanced editors have modules that allow 
    for working on python code such as Visual Studio Code, Atom, XCode (MacOS only), Emacs, etc.  
- In using these editors, it becomes much harder to make simple mistakes such as incorrectly indenting sections of code, 
accidentally using incorrect parameters and incorrectly spelling variable or function names.  
    * PyCharm, along with some of the other editors features tab completion.
        + Tab completion is when suggested function names, variable names, etc show up while typing, similar to the 
        autosuggestion features on your smartphone's keyboard. 
        + You can navigate the suggestion list using the up & down arrow keys and then hit tab when you've found the 
        right suggestion.  

### Modularity
- A common practice in software development is to make sure that a code is modular, i.e., broken into simple blocks.  
    * You should always make sure to have a ```main``` function which can be used to call other functions.  
    * Functions are best written to achieve one main goal. If you have a large function that performs multiple actions, then 
    it would be more beneficial to separate the function into different functions.  
    * If the code is getting long and there are a lot of constants, variables and functions that you use often but take 
    up a lot of space in the file, it may be best to consider creating another file and then importing it in order to 
    still have access to these constants, variables and functions. 
        + This makes the code much easier to read and work on since there will 
        be much less text to look at in your file.  

### Googling
- There are so many resources online, so please don't forget it! If you're trying to do something but do not know how to 
implement it correctly, most times there is someone online who has already asked that question and then got an answer. 
    * I'm not saying that code should be copied directly, but rather there is a lot to learn from others online who 
    have been kind enough to provide answers.  
    * Also it doesn't hurt to Google since you may find out that there are libraries that exist that could save time by  
    allowing you to use them instead of writing your own version yourself.
<!--#################################################################################################################-->
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




