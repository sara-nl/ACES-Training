# ACES training pipeline

## Synopsis
This tutorial teaches master and PhD students how to coordinate so-called embarassingly parrallel computational tasks across different infratsructures.
The tutorial shows students how to create tokens and process tokens which code for the single runs.
The pipeline makes use of couchdb as a token pool server and uses python and the [picasclient](https://github.com/jjbot/picasclient).

## Technology requisites
To follow the tutorial you need a python distribution and access to a couchdb instance.
We provide an an old python distribution epd.tar.gz pre-installed with all required additional packages.

Using the provided python distribution:
You can download the [epd](https://ndownloader.figshare.com/files/4851667) and install it:
```sh
tar xzf epd.tgz
export PATH=$D/epd-7.3-1-rh5-x86_64/bin:$PATH
```

If you want to use an own python distribution, please install the following packages.

Module | Version
-------|---------------
numpy | 1.6.1.
scipy | 0.10.0
sklearn | 0.11
h5py | 2.0.0
xlrd | not known
couchDB | 0.9

## Context
The training will make use of a double-looop crossvalidation pipeline which is described in detail in [Staiger et. al](http://dx.doi.org/10.3389/fgene.2013.00289). 
We will create tokens for the Single gene classifier and the Lee classifiers. Furthermore and for didactical reasons we will also create tokens which will fail to be processed by the pipeline.

