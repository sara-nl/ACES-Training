# ACES training pipeline

## Synopsis
This tutorial teaches master and PhD students how to coordinate so-called embarassingly parrallel computational tasks across different infratsructures.
The tutorial shows students how to create tokens and process tokens which code for the single runs.
The pipeline makes use of couchdb as a token pool server and uses python and the [picasclient](https://github.com/jjbot/picasclient).

## Technology requisites
To follow the tutorial you need a python distribution and access to a couchdb instance.
We provide an an old python distribution epd.tar.gz pre-installed with all required additional packages.

Using the provided python distribution:
```sh
tar xzf epd.tgz
export PATH=$D/epd-7.3-1-rh5-x86_64/bin:$PATH
```

If you want to use an own python distribution, please install the following packages.
python dependencies:
----------------------
numpy version 1.6.1.
scipy version 0.10.0
sklearn version 0.11
h5py version 2.0.0
xlrd
couchDB version 0.9



