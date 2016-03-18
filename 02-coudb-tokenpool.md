# The couchdb as token pool server

## Outline
In this section we will shortly explain what a token pool server is and how couchdb can be employed. For an extensive tutorial on couchdb we refer to their [official website](http://couchdb.apache.org/)

## Token pool server
A token is something that encodes something else. In our case the tokens encode a computational problem that need to be calculated. All of our tokens that we created can be run independently from each other. However, all of the tokens need to be computed and their solution need to be combined to solve the whole problem. This is what we also call *embarassingly pararllel*.
A token pool server needs to be able to store tokens, iterate over them and to either remove tem from the pool as soon as the computations finished or label them differently so that they will not be recalculated.

# couchdb as a token pool server
To use couchdb as a tokenpool server, we need to pass some extra *flags* to our tokens.
In the previous section we pointed out, that apart from the input parameters some other key-value pairs were stored in tokens.
In couchdb we can use these parameters to sort tokens and create so-called views.

Flag | meaning
-----|---------
done | The token was processed successfully.
lock | The token was picked up to be processed.

Combinations of these two flags tell us something about the state of a token.

done | lock | meaning
------|-----|---------
0 | 0 | Token needs to be tackled and will end up in a todo list.
0 | 1 | Token is being processed, or during the computation an error occurred.
1 | 1 | Token was successfully processed.
1 | 0 | This is weird state of the token which should never occurr.

For each of the combinations we can create a so-called view in the couchdb and only process tokens belonging to a specific view, e.g. 
the combination for the todo list.
The flags get updated in the pipeline automatically according to their state. The picas client is taking care of it. This allows us to keep tokens and even store some output data in them.

## Creating a view in a couchdb

1) Go to the button view
2) Click on *Temporary view*
3) Use javascript to define the view, e.g. for the todo tokens
 ```sh
 function(doc) {
  if(doc.type == 'token') {
    if(doc.lock == 0 && doc.done == 0) {
      emit([doc._id],[doc._id]);
    }
  }
}
 ```
4) Save the view under _design/run with the name *Todo*
