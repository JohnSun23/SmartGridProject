# Overview #
A system for compressing smart grid data, and allowing queries on the compressed representation.  

# Notebooks with Results #

### Piecewise ###
Profiles/visualize optimal piecewise-constant (wrt MSE) 1D signal fits.  

### MetaDecompressionExample ###
This gives an example of loading meta-compression, and reconstructing an example. 

### ChooseBestCompressors ###
A notebook that loads space/error statistics, and selects the compressor/complexity hyperparameter to use per sensor.  This is saved in a DataFrame, to be loaded into a meta-compression mapreduce job. 

### TradeoffBasedAnalysis ###
A reasonably-well-documented analysis of sensor grouping, using the space/error tradeoff information

# Getting Started #
### Download, build, set env ###
Create a directory, let's say SmartGridProject.  

Clone this github directory into SmartGridProject/src

Create another directory SmartGridProject/data.  Create sub-directories:
```bash
mkdir $SMART_GRID_DATA/summary_data
mkdir $SMART_GRID_DATA/compression
```

In `setup.sh` replace `ion_username` with your ion-21-14 username.  The directions below are for using the system with username melkherj on ion-21-14.  If you need to preprocess this data again from scratch, you'll need to change `melkherj` to your username.  You will also need to change `melkherj` references in `stream_config.sh` to your username.  

In SmartGridProject/src, run `source setup.sh`

Build our cython libaries:
```bash 
./build.sh
```


### Load Data into HDFS ###
Run 
```bash 
hdfs -ls /user/melkherj/unprocessed_power_csvs
```
This should report: `No such file or directory.`  If it doesn't, you should either: skip this step and use the existing data, or `hdfs -rmr` the existing data and 

```bash
hdfs -copyFromLocal /oasis/projects/nsf/csd181/melkherj/PI_data/PI_datasets/oledb_phase1 /user/melkherj/unprocessed_power_csvs
```
This won't work if the file `/user/melkherj/unprocessed_power_csvs` in hdfs already exists

### Preprocess data in HDFS ###
Run: 
```bash
./stream.sh preprocess_power_data/stream_config.sh
```

### Create Seek Location Hash ###
This allows for more efficient random access to the original sensor data on HDFS
    
```bash
./stream.sh model_power_data/get_tag_part_seek/stream_config.sh
hdfs -getmerge /user/melkherj/tag_part_seek "$SMART_GRID_DATA/summary_data/tag_part_seek"
````

### Run Compression Evaluation ###
Run all compression methods on hadoop.  Copy the '^'-separated file giving space/error tradeoff per compressor from HDFS to local.  Then process resulting error/space statistics into a pandas dataframe.  

``` bash
./stream.sh ./model_power_data/evaluate_all_tags/stream_config.sh
hdfs -getmerge /user/melkherj/all_space_err.txt "$compression_data_dir/all_space_err.txt"
```

Load the notebook `notebooks/ChooseBestCompressors.ipynb`

If you wish to copy this evaluation data to your local machine:
```bash
scp <your username>@ion-21-14.sdsc.edu:/oasis/projects/nsf/csd181/melkherj/PI_data/compression/all_space_err.txt <your local root>/data/compression/all_space_err.txt
scp <your username>@ion-21-14.sdsc.edu:/oasis/projects/nsf/csd181/melkherj/PI_data/summary_data/tag_part_seek <your local root>/data/summary_data/tag_part_seek
```


### Run Meta-Compression ###

