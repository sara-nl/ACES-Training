# Creating tokens

## Outline
In this section we will create database entries which code for single runs.
1) Make the right combination of key words
2) Convert this list of keywords into json and store in couchdb

Start an ipython or interactive python session in the folder ***code*.

## Coding for the tokens
The function **CombineData** in **SetUpGrid.py** creates a list
```sh
[(<dataset>, ((<algorithm>, <specific paramteres>), <network or pathway data>), <#shuffles of the network data>)]
``` 

```py
from SetUpGrid import CombineData
DataAndFeatureExtractors = CombineData()
```
Now we have the right combination of algorithms, their special parameters and the two types of data.

From this list we will create tokens which will be saved in a couchdb. The tokens also need to contain information on the status of the cross validation, i.e. the split of data which is used for training (extracting features) and the split of data which is used for testing the classifier which is employing the extracted features.
This becomes even more complicated since we try to run a double loop cross validation.

<img src= width="400px">

The python file **createTokens.py** contains these functions.


