# Polypharmacy Side Effect Prediction


```
singularity build --fakeroot decagon.sif decagon.def
```


![alt text](https://github.com/diitaz93/polypharm_predict/blob/main/images/pipeline.png "Pipeline")

1. Download data
```
$ ./data/download_data.sh
```
2. Singularity
```
singularity shell decagon.sif
```
3. Generate Decagon data structures
```
$ cd data
$ ./DS_generator.py 964
```
