# Investigations

### Description
This application is designed to create investigation files from data of various facilities.
There are utilities which are present that can further enrich data in the investigation files.

For PDBe
The model files can be provided as a folder path, or as PDB Ids.
Where PDB ids are specified, the data is fetched from FTP Area of EBI PDB archive

For MaxIV:
SqliteDB file is required

### Setup and Installation
```
git clone https://github.com/PDBeurope/Investigations.git
cd Investgations
pip install -r requirements.txt
```
### Usage
The script requires to specify a facility as the first argument:

```
python investigation.py --help
usage: Investigation [-h] {pdbe,max_iv,esrf} ...

This creates an investigation file from a collection of model files which can be provided as folder path, pdb_ids, or a csv file. The model files can be provided

positional arguments:
  {pdbe,max_iv,esrf}  Specifies facility for which investigation files will be used for
    pdbe              Parameter requirements for investigation files from PDBe data
    max_iv            Parameter requirements for investigation files from MAX IV data
    esrf              Parameter requirements for investigation files from ESRF data

```

Each facility have its own set of arguments. 
For MAX IV

```
python investigation.py max_iv --help
usage: Investigation max_iv [-h] [-o OUTPUT_FOLDER] [-i INVESTIGATION_ID] [-s SQLITE]

optional arguments:
  -h, --help            show this help message and exit
  -o OUTPUT_FOLDER, --output-folder OUTPUT_FOLDER
                        Folder to output the created investigation files to
  -i INVESTIGATION_ID, --investigation-id INVESTIGATION_ID
                        Investigation ID to assign to the resulting investigation file
  -s SQLITE, --sqlite SQLITE
                        Path to the Sqlite DB for the given investigation
```

For PDBE
```
python investigation.py pdbe --help  
usage: Investigation pdbe [-h] [-o OUTPUT_FOLDER] [-i INVESTIGATION_ID] [-f MODEL_FOLDER] [-csv CSV_FILE] [-p PDB_IDS [PDB_IDS ...]]

optional arguments:
  -h, --help            show this help message and exit
  -o OUTPUT_FOLDER, --output-folder OUTPUT_FOLDER
                        Folder to output the created investigation files to
  -i INVESTIGATION_ID, --investigation-id INVESTIGATION_ID
                        Investigation ID to assign to the resulting investigation file
  -f MODEL_FOLDER, --model-folder MODEL_FOLDER
                        Directory which contains model files
  -csv CSV_FILE, --csv-file CSV_FILE
                        Requires CSV with 2 columns [GROUP_ID, ENTRY_ID]
  -p PDB_IDS [PDB_IDS ...], --pdb-ids PDB_IDS [PDB_IDS ...]
                        Create investigation from set of pdb ids, space seperated
```

`--investigation-id` parameter is an optional parameter where the user wants to control the investigation ID that is assigned to the investigation file. It is not used where input is csv file. 

#### Importing data from Ground state file
For files where the data of misses are present in structure factor file, the `miss_importer.py` utility can be used to enrich the investigation data with new information.

```
$ python miss_importer.py --help
usage: Ground state file importer  [-h] [-inv INVESTIGATION_FILE] [-sf SF_FILE] [-p PDB_ID] [-f CSV_FILE]

This utility takes as an input investigation file, and sf file. And imports the data for all the misses from the sf file and adds that to the investigation file

optional arguments:
  -h, --help            show this help message and exit
  -inv INVESTIGATION_FILE, --investigation-file INVESTIGATION_FILE
                        Path to investigation file
  -sf SF_FILE, --sf-file SF_FILE
                        Path to structure factor file
  -p PDB_ID, --pdb-id PDB_ID
                        PDB ID to lookup to download the sf file
  -f CSV_FILE, --csv-file CSV_FILE
                        Requires CSV with 2 columns [investigation_file, Pdb Code (to fetch sf file)]
```

The utility requires the created investigation file, along with a sf file (or pdb code to automatically fetch the sf file) as input. 
And outputs a modified investigation cif file.

### Example

#### MAX IV

```
investigation.py max_iv --sqlite fragmax.sqlite -i inv_01
```


#### PDBE
PDB Ids can be passed in the arguments. The model file is fetched from EBI Archive FTP area temporarily stored. After the investigation file is created the files are deleted.
```
python investigations.py pdbe -p 6dmn 6dpp 6do8
```

A path can be given to the application. All cif model files in the folder are regarded as input.
```
python investigations.py pdbe -m path/to/folder/with/model_files
```

A CSV file can be provided as input. The csv file should have two columns `GROUP_ID` and `ENTRY_ID`.
Entries in the same groups are processed together, and an investigation file is created for each unique `GROUP_ID`
```
python investigations.py pdbe -f path/to/csv/file
```

### Working
The investigation file is created from the constituent model file. The data from the model file is parsed via Gemmi and stored in a in-memory SQLite database, which denormalises the data in the various categories amongst all the files.

The operations.json file is read by the program, and operations specified are ran sequentially. 
The operations generally specify source and target category and items, operation to perform, and parameter that the operation may require.
The operation may leverage the denormalised table created initially.

Once all operations are peformed the resultant file is written out where name of the file is the investigation_id.
Incase an operation cannot be performed due to missing data in the file, the operation gets skipped and the error is logged.