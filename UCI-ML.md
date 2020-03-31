# UCI Machine Learning Repository Datasets

UCI maintains machine learning datasets that you can quickly ingest into Hazelcast using the Pado CSV file importer. This article provides step-by-step instructions for downloading datasets, generating schema files, generating `VersionedPortable` source code, compiling generated source, deploying generated jar files, and running the Hazelcast Desktop app to browse the datasets in Hazelcast.

[https://archive.ics.uci.edu/ml/index.php](https://archive.ics.uci.edu/ml/index.php)

## Dataset Downloads

For convenience, the `download_uci_ml` script has been provided in the `bin_sh` directory. You can run that script to download the datasets used in this article. If the script does not work then manually download the datasets by following the download links shown in each dataset section below.

```console
cd_app pado
cd bin_sh
./download_uci_ml
```

After you have downloaded files, you should have the directory list similar to the following:

```console
data/uci-ml/
├── forestfires
│   ├── forestfires.csv
│   └── forestfires.names
├── incident-event-log
│   └── incident_event_log.zip
├── poker-hand
│   ├── poker-hand-testing.data
│   ├── poker-hand-training-true.data
│   └── poker-hand.names
└── thyroid-disease
    ├── HELLO
    ├── Index
    ├── allbp.data
    ├── allbp.names
    ├── allbp.test
    ├── allhyper.data
    ├── allhyper.names
    ├── allhyper.test
    ├── allhypo.data
    ├── allhypo.names
    ├── allhypo.test
    ├── allrep.data
    ├── allrep.names
    ├── allrep.test
    ├── ann-Readme
    ├── ann-test.data
    ├── ann-thyroid.names
    ├── ann-train.data
    ├── costs
    │   ├── Index
    │   ├── ann-thyroid.README
    │   ├── ann-thyroid.cost
    │   ├── ann-thyroid.delay
    │   ├── ann-thyroid.expense
    │   └── ann-thyroid.group
    ├── dis.data
    ├── dis.names
    ├── dis.test
    ├── hypothyroid.data
    ├── hypothyroid.names
    ├── new-thyroid.data
    ├── new-thyroid.names
    ├── sick-euthyroid.data
    ├── sick-euthyroid.names
    ├── sick.data
    ├── sick.names
    ├── sick.test
    ├── thyroid.theory
    ├── thyroid0387.data
    └── thyroid0387.names
```

## Dataset Ingestion

We'll ingest several datasets into Hazelcast in the form of `VersionedPortable` objects. We need to first generate schema files using the `generate_schema` command which runs per data directory. Since some of the data files may not contain the header row, we won't be able to place all the files in the same directory. Let's create directories to which we can split header and no-header files.

```console
cd_app pado
cd pado_<version>

# For files that have no header row
mkdir -p data/h0

# For files that have the header row
mkdir -p data/h1
```

### Forest Fires

[https://archive.ics.uci.edu/ml/machine-learning-databases/forest-fires/](https://archive.ics.uci.edu/ml/machine-learning-databases/forest-fires/)

```console
cd_app pado
cd pado_<version>
cp data/uci-ml/forestfires/forestfires.csv data/h1
``` 

### Incident Event Log

[https://archive.ics.uci.edu/ml/datasets/Incident+management+process+enriched+event+log](https://archive.ics.uci.edu/ml/datasets/Incident+management+process+enriched+event+log)

```console
cd_app pado
cd pado_<version>
unzip -d data/h1 data/uci-ml/incident-event-log/incident_event_log.zip
```

### Poker Hand

[https://archive.ics.uci.edu/ml/datasets/Poker+Hand](https://archive.ics.uci.edu/ml/datasets/Poker+Hand)

```console
cd_app pado
cd pado_<version>
cp data/uci-ml/poker-hand/*.data data/h0/
```

### Thyroid Disease

[https://archive.ics.uci.edu/ml/datasets/Thyroid+Disease](https://archive.ics.uci.edu/ml/datasets/Thyroid+Disease)

```console
# Copy allhyper.data to data/ml/h1
cd_app pado
cd pado_<version>
cp data/uci-ml/thyroid-disease/allhyper.data data/h0/
```

### Dataset Files

You should have the following files in the `data/h0` and `data/h1` directories:

```console
data/h0
├── allhyper.data
├── poker-hand-testing.data
└── poker-hand-training-true.data
```

```console
data/h1
├── forestfires.csv
└── incident_event_log.csv
```

### Generate Schema Files

```console
cd_app pado
cd pado_<version>/bin_sh/hazelcast

# Generate schema files. We need to specified the -headerRow option 
# to indicate that the data/h0 directory contains no-header files.
# If not specified then it assumes the first row is the header row.
./generate_schema -package org.hazelcast.data.ml -dataDir data/h0/ -headerRow 0
./generate_schema -package org.hazelcast.data.ml -dataDir data/h1/
```

:exclamation: The column information for `allhyper.data` is in `data/uci-ml/throid-disease/allhyper.names`. Let's replace the generated field names in the `allhyper.schema` file with the column names found in the `allhyper.names` file as follows:

```console
cd_app pado
cd pado_<version>

# Edit the schema file and replace the field names at the bottom of the file
# with the following (Note that Age is String because there are some rows with
# that column set to '?'. This can be filtered via Pado importer but for now,
# let's set it to String.
vi data/schema/generated/allhyper.schema 

Age, String
Sex, String
On_thyroxine, String
Query_on_hyroxine, String
On_antithyroid_medication, String
Sick, String
Pregnant, String
Thyroid_surgery, String
I131_treatment, String
Query_hypothyroid, String
Query_hyperthyroid, String
Lithium, String
Goitre, String
Tumor, String
Hypopituitary, String
Psych, String
TSH_measured, String
TSH, String
T3_measured, String
T3, String
TT4_measured, String
TT4, String
T4U_measured, String
T4U, String
FTI_measured, String
FTI, String
TBG_measured, String
TBG, String
Referral_source, String
Classes, String
```

:white_check_mark: As you can see from the above steps, ingesting CSV files can be a challenge if they are malformed or the header row is missing. Always check the top few lines of each data file to verify their format and make adjustments to command arguments and schema files as necessary. As a last resort, if all the options failed, you may need to adjust the data files.

### Generate and Deploy `VersionedPortable`

```console
cd_app pado
cd pado_<version>

# Move the genrated schema files to the data/schema directory
mv data/schema/generated/* data/schema/

# Move the data files to the import directory
mv data/h0/* data/import/
mv data/h0/* data/import/
```
The `data/schema/` directory now has all the generated schema files.

```console
data/schema/
├── allhyper.schema
├── forestfires.schema
├── generated
├── incident_event_log.schema
├── poker-hand-testing.schema
└── poker-hand-training-true.schema
```

The `data/import/` directory now has all the data files.

```console
data/import/
├── allhyper.data
├── forestfires.csv
├── incident_event_log.csv
├── poker-hand-testing.data
└── poker-hand-training-true.data
```

Let's generate and deploy `VersionedPortable` classes.

```console
# Generate VersionedPortable source code
cd bin_sh/hazelcast
./generate_versioned_portable -fid 30001 -cid 30021

# Compile the generated source code
./compile_generated_code -jar uci-ml-generated.jar

# Copy the generated jar file to the workspace plugins dir so that
# it gets included in the cluster class path
cp ../../dropins/uci-ml-generated.jar $HAZELCAST_ADDON_WORKSPACE/plugins/
```

### Start Cluster

```console
# Add serialization config
switch_cluster myhz
vi etc/hazelcast.xml
          <portable-factory factory-id="30001">
          org.hazelcast.data.ml.PortableFactoryImpl
          </portable-factory>

# Start cluster
start_cluster
```

### Import Datasets

```console
# Import data
cd_app pado
cd pado_<version>/bin_sh/hazelcast
./import_csv
```

Upon successful run, you should see an output similar to the following:

```console
          Total file count: 5
Total processed file count: 5
    Total error file count: 0
    Total path entry count: 1,170,039
  Total elapsed time (sec): 12
```

### Run Desktop

```console
# If you haven't installed the desktop app then install and build it.
create_app -app desktop
cd_app desktop
cd bin_sh
./build_app

# Change directory to hazelcast-desktop
cd ../hazelcast-desktop_<verson>

# Add serialization configuration in etc/pado.properties
vi etc/pado.properties

hazelcast.client.config.serialization.portable.factories=1:org.hazelcast.demo.nw.data.PortableFactoryImpl,\
10000:org.hazelcast.addon.hql.impl.PortableFactoryImpl,\
30001:org.hazelcast.data.ml.PortableFactoryImpl

# Copy the uci-ml-generated.jar file in the dekstop plubins dir
cp $HAZELCAST_ADDON_WORKSPACE/plugins/uci-ml-generated.jar plugins/

# Run desktop
cd bin_sh
./desktop
```