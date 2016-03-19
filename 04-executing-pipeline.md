# The ACES pipeline on lisa

## Outline
This section will guide you through the stepa to execute the pipeline remotely on a large compute cluster remotely.
We will create a pipeline python script, which will be embedded in a shell-script.
The shell-script takes care of setting the environment of your compute cluster.

## Setting the environment
The compute nodes of lisa need to be equipped with the code you want to run and the dependencies for the python compiler.
We will ship the code as a tgz-file.

### Preparing the code
1) Open *pipeline.py* and enter your couchdb username and password and save the file.
2) Create the tar-ball:
   ```sh
   tar czf /home/<user>/code.tgz /home/<user>/<ACESPATH>/code
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
cd $D/code/
```

Finally we can start our python pipeline.
```sh
python pipeline.py
```

## Example shell script
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

cd $D/code/

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

