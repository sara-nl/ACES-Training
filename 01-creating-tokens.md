# Creating tokens

## Outline
In this section we will create database entries which code for single runs.

1) Make the right combination of key words

2) Convert this list of keywords into json and store in couchdb

Start an ipython or interactive python session in the folder *code*.

```sh
ipython
```

## Coding for the tokens
The function *CombineData* in *SetUpGrid.py* creates a list
```sh
[(<dataset>, ((<algorithm>, <specific paramteres>), <network or pathway data>), <#shuffles of the network data>)]
``` 

```py
from SetUpGrid import CombineData
DataAndFeatureExtractors = CombineData()
```
Now we have the right combination of algorithms, their special parameters and the two types of data.

From this list we will create tokens which will be saved in a couchdb. The tokens also need to contain information on the status of the cross-validation, i.e. the split of data which is used for training (extracting features) and the split of data which is used for testing the classifier which is employing the extracted features.
This becomes even more complicated since we try to run a double-loop cross-validation.

<img src="https://github.com/chStaiger/ACES-Training/blob/master/DLCV.jpg" width="400px">

The python file *createTokens.py* contains these functions.
For this we need information on how often we want to **repeat** the cross-validation *nrRepeats* and in how **many parts** we want to split our dataset *nrFolds*. For a **5-fold crossvalidation** *nrFolds* would be five.
To label the runs according to a special experiment, we will also give a keyword for this experiment. This comes in handy when accommodating tokens of several experiments in the same couchdb. 
You can add your initials to the keyword to later find your tokens in the database.

First we create the tokens for the outerloop, we will run a 5-fold crossvalidation 10 times and label the tokens as "TrainingOuter", hence the additional parameters are *10*, *5* and *Training*:

```py
from CreateTokens import generate_tokens
```
First get some help on which parameters need to be set.
```py
?generate_tokens
outerTokens = generate_tokens(DataAndFeatureExtractors, 10, 5, "TrainingOuter<YourLoginName>")
```

Inspect the first token:
```py
outerTokens[0]
outerTokens[0]['input']
```
You see that the token is nothing more than a python dictionary, containing an ID and another dictionary, which contains all necessary input parameters. 
There are also the keywords *done*, *hostname*, *lock* and *scrub_count* which will be set and used when the token is processed.

Now let us create the tokens to execute the innerloop of the crossvalidation.
The first three parameters need to match the first three parameters for the outer loop. Additionally, we need to set the parameters for the inner loop, i.e. how often do we want to run the innerloop and in how many parts do we want to split the data. *Note* that when entering the innerloop, we do not receive the full data set but only the n-1 parts that are used for training in the outer loop.
You can first inspect the help to this function.
```py
from CreateTokens import generate_tokens_innerloop
innerTokens = generate_tokens_innerloop(DataAndFeatureExtractors, 10, 5, 3, 5, "TrainingInner")
```

Now we need to upload the data to a couchdb instance.

1) connect to de database and upload the tokens
```py
import couchdb

couch = couchdb.Server("https://nosql01.grid.sara.nl:6984")
couch.resource.credentials = ("<username>", "<password>")
db = couch["hpc-training"]

for token in outerTokens+innerTokens:
    db.create(token)
```

Now go to https://nosql01.grid.sara.nl:6984/_utils/ and access the training database *hpc-training* and inspect the tokens.

