# Processing tokens in the couchdb

## Outline
In this section we will finally process the tokens we created.
1) We will inspect the code which will translate our tokens into instances of classes for datasets, classifiers and feature extraction algorithms. In short we will answer the question how do I come from a combination of strings, parameters and flags to something that really does the computation.

2) We will start the computations on an HPC cluster.
2a) Connect to the database
2b) Fetch a token
2c) Create the real python instances
2d) Start the computation
2e) Store the result in the token and push it to the couchdb
3) Iterate over the steps under 2) until there are no more tokens in our todo view.

4) **Watch out**
 In the todo view there are also tokens which will not run properly and end with an error. So you might receive error messages during the computation. In that case try to find out what went wrong and either fix the token so that it can be processed or remove it from the pending list to the weird list.
You can do that in the futon view of the couchdb or via python code employing the [couchdb module](https://pythonhosted.org/CouchDB/getting-started.html).

## Translating words to python instances

We created a dictionary *input* for each of the tokens. This dictionary is read and serves as input to set up the data and algorithms

```py
dataset     = token['input']['dataset']
network     = token['input']['network']
method      = token['input']['method']
specific    = token['input']['specific']
repeat      = token['input']['repeat']
fold        = token['input']['fold']
shuffleNr   =  token['input']['shuffleNr']
```
For the tokens that define the inner crossvalidation loop we also have

```py
innerfold   = token['input']['innerFold']
innerrepeat = token['input']['innerRepeat']
```
The function **SetUpRun** in **SetUpGrid** loads the respective data and instantiates the algorithms for each token.
Please note that tokens are processed sequentially in each call of the *pipeline.py* script. If you want to run more of them you need to start the pipeline several times.

## Execute the workflow for an example token
First we need to get a token from the pool.

```py
import couchdb

couch = couchdb.Server("https://nosql01.grid.sara.nl:6984")
couch.resource.credentials = (<username>, <password>)
db = couch["hpc-training"]

token = db['TrainingInner_U133A_combat_DMFS_None_SingleGenes_None_0_0_0_0_None']
```

Read the input

```py
dataset     = token['input']['dataset']
network     = token['input']['network']
method      = token['input']['method']
specific    = token['input']['specific']
repeat      = token['input']['repeat']
fold        = token['input']['fold']
shuffleNr   =  token['input']['shuffleNr']

innerfold   = token['input']['innerFold']
innerrepeat = token['input']['innerRepeat']
```

Instantiate the data
```py
from SetUpGrid import SetUpRun
(data, net, featureSelector, classifiers, Dataset2Time) = SetUpRun(dataset, network, method)
```
We can get some information on the data and algorithms
```py
print(data)
print(net)
print(featureSelector)
print(classifiers)
```

Now we can calculate the result, with number of folds 
```py
from SetUpGrid import RunInstance
(dataName, featureExtractorproductName, netName, shuffle, featureExtractor, AucAndCi) = RunInstance(
    data, net, featureSelector, specific, classifiers, repeat, 10, fold, shuffleNr, Dataset2Time)
```

The output contains some simple data types like strings and numbers.

```py
print(dataName)
print(featureExtractorproductName)
print(netName)
```

But also some complicated data structures:

```py
type(AucAndCi)
```

Some very complicated datastructure were converted to json strings but we cab revive them.

```
import json
type(featureExtractor)
FE = json.loads(featureExtractor)
type(FE)
```

*FE* is a list containing
- Name
- Namespace of the features
- Ranked indexes of the features according to their performance
Converting these datastructures to json and having functions to restore them allows us to store the output in the tokens.

## Executing the pipeline
Now that we are convinced that the pipeline works, we can execute it on a compute cluster like lisa. 








