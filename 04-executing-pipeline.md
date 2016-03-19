# The ACES pipeline on lisa

## Outline
This section will guide you through the steps to execute the pipeline in parallel on a large compute cluster employing a portable batch system.
We will create a pipeline python script, which will be embedded in a shell-script.
The shell-script takes care of setting the environment of your compute cluster.

We assume that the code of this repository already resides on an access node or home directory of the compute cluster.
If not, checkout the git repository again.

## Setting the environment
The compute nodes of lisa need to be equipped with the code you want to run and the dependencies for the python compiler.
We will ship the code as a tgz-file.

### Preparing the code
1) Open *pipeline.py* and enter your couchdb username and password and save the file.
2) Create the tar-ball:
   ```sh
   cd /home/<user>/<ACESPATH>
   tar cvzf /home/<user>/code.tgz code/ 
   ``` 

### Creating the shell-script

First we need to set some parameters for lisa to choose the number of nodes, memory and running time.

```sh
#PBS -lwalltime=3:00:00
#PBS -lnodes=1:cores16:mem64gb
#PBS -S /bin/bash
``` 

Then we need to install the python dependencies.

```sh
easy_install --user couchdb
easy_install --user scikit-learn
```

We will also create a working directory, which will be removed after our runs finished.

```sh
D=$TMPDIR/christine_$RANDOM
echo $D
mkdir $D

cd $D
```

Now we copy our code to the working directory, unpack it and change our working directory to there.
```sh
cp ~/code.tgz code.tgz
tar xzf code.tgz
cd code/
```

Finally we can start our python pipeline.
```sh
python pipeline.py
```

### Example shell script
Save the cammonds above in a shell script called *go.sh*

```sh
#PBS -lwalltime=3:00:00
#PBS -lnodes=1:cores16:mem64gb
#PBS -S /bin/bash

# Install python dependencies
easy_install --user couchdb
easy_install --user scikit-learn

# Print all commandos
set -e

# Create a working directory
D=$TMPDIR/christine_$RANDOM
echo $D
mkdir $D

cd $D

# Copy and unpack the code
cp ~/code.tgz code.tgz
tar xzf code.tgz

cd code/

# Start the pipeline
python pipeline.py
```

## Submitting a job to the queue

```sh
qsub go.sh
```

If you need to start the same script several times you can use
```sh
qsub -t 1-100 go.sh
```

To monitor your jobs, use
```sh
qstat -u <username>
```

## Inspecting the log files

After you job finished, whether siccessfully or not, you will find two files in your home directory
```sh
#output file
go.sh.o<jobnumber>
#error file
go.sh.e<jobnumber>
```
The outputfile contains all output the python pipeline produces, the errorfile contains system errors.

## Fetching the computed data from couchdb
This section does not require any HPC. You can use your own python compiler, but make sure it is equipped with the *couchdb* module.
Remember that we computed an outer and an inner crossvalidation pipeline. In this section we will select the correct tokens to make a plot for one algorithm on all splits of the outerloop's results.
And we will reset pending tokens

1) Connecting to couchdb
```py
import couchdb
couch = couchdb.Server("https://nosql01.grid.sara.nl:6984")
couch.resource.credentials = (<user>, <password>)
db = couch["hpc-training"]
```

2) Fetch all tokens from our outer crossvalidation loop which are computed and those which are still pending.
```py
from SetUpGrid import getDoneTokens
done, pending = getDoneTokens(db, "TrainingOuter")
```
Inspect one of the tokens. You will see that *lock* and *done* set. In fact that is the sytem time when the calculations started and when they stoped. So you can calculate how long it took to run the algorithm on this specific instance. Furthermore, you can see on which machine the computations were executed.


3) Reset pending tokens. TOkens that are marked as pending, can still be processed but couls also have stopped with an error. In both cases we can reset the tokens and put them again in the todo list.

```py
from SetUpGrid import resetTokens
ids = [token['_id'] for token in pending]
resetTokens(ids, db)
```

Plotting the AUC for one token
```py
token = done[0]
token["output"][6][u'BinaryNearestMeanClassifier_V1']
```

Now we have a dictionary mapping from number of features to AUC and CI.
Create a numpy array with features versus AUC values
```py
import numpy
import matplotlib.pyplot as plt
featVsAucs = numpy.array([[key, value[0]] for key, value in token["output"][6][u'BinaryNearestMeanClassifier_V1'].items()], dtype=numpy.float64)
plt.plot(featVsAucs[:, 0], featVsAucs[:, 1], 'ro')
plt.ylabel("AUCs")
plt.xlabel("#Features")
plt.savefig("<PATH>/AUC.png")
plt.clf()
```








