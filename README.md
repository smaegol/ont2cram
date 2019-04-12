**Python 3 or higher is required**
## ont2cram
Oxford Nanopore HDF/Fast5 to CRAM conversion tool

INSTALLATION: 
~~~
git clone https://github.com/EGA-archive/ont2cram
cd ont2cram
python setup.py install
~~~

USAGE: 
~~~
ont2cram -i INPUTDIR -o OUTPUTFILE [-f FASTQDIR] [-s]
  -i INPUTDIR, --inputdir INPUTDIR
                        Input directory containing Fast5 files
  -o OUTPUTFILE, --outputfile OUTPUTFILE
                        Output CRAM filename
  -f FASTQDIR, --fastqdir FASTQDIR
                        Input directory containing FASTQ files
  -s, --skipsignal      Skips the raw signal data
~~~

Implementation details:

There is a mapping table in the header that maps ONT attributes/dataset columns to lowercase SAM aux tags eg:
**ATR:'Analyses/Basecall_1D_000/Configuration/calibration_strand/genome_name':S TG:b5 CV:'Lambda_3.6kb'**

general format is : 
~~~
[ATR|COL]:'<hdf_attribute_or_column_pathname>':<original_datatype> TG:<2_letter_lowecase_tag> CV:<constant_value>

ATR - mapping between hdf group/dataset attribute and SAM aux tag
COL - mapping between hdf dataset column and SAM aux tag
~~~
**<original_datatype>** is represented using Numpy datatype character codes ( https://docs.scipy.org/doc/numpy/reference/arrays.dtypes.html#specifying-and-constructing-data-types )  

CV part is optional - currently present only for attributes(skipped for dataset columns) and only if >50% of fast5 files have this value. Thus, in current implementation CV is more like 'common value' than constant ( it can be overwritten on the read level for those reads that have different value ).

Tag names are generated sequentially ( a0,a1....aa.....az,aA...aZ...zZ ). If 'zZ' is reached the program exists with an error. 

HDF datasets are stored in Cram as separate columns - each column in a separate tag

Optional **"--skipsignal"** flag allows to skip raw signal(and Events dataset which is derived from Signal) and produce _much_ smaller Crams

Optional **"--fastqdir"** arg allows to specify input folder(can be the same as "--inputdir") for FASTQ sequences. If "--fastqdir" is specified each FAST5 file is expected to have corresponding FASTQ file in this dir with identical name but different extension(".fastq")

## cram2ont ( reverse converter )

*CRAM to Fast5 conversion utility (this is a reverse converter which allows to
restore original Fast5 collection from Cram generated by ont2cram)*

usage:
~~~
cram2ont -i INPUTFILE [-o OUTPUTDIR]
  -i INPUTFILE, --inputfile INPUTFILE 
    Input CRAM filename
  
  -o OUTPUTDIR, --outputdir OUTPUTDIR
    Output directory for generated Fast5 files
~~~    
If output directory is not specified generated files will be saved to the current folder.

Open questions/problems:
* Multi-read Fast5 are not supported ( yet )
* HDF has sveral datatypes for strings : Null-padded/Null-terminated, Variable length/Fixed length.The restored type is not always identical to the source e.g. "37-byte null-terminated ASCII string" vs "36-byte null-padded ASCII string"(the content is identical "00730cca-2ff9-4c03-b071-d219ee0a19b8")
* Does it make sense to store HDF layout info(dataset compression method/chunkig settings/dataset max dimensions)?
* Some string values in HDF attributes have line breaks inside - is it valid for Cram tags or better to remove them?
