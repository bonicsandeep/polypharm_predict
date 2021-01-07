# Polypharmacy Side Effect Prediction

[Decagon](https://github.com/mims-harvard/decagon) is a multimodal graph convolutional autoencoder for polypharmacy side effect prediction. This repository contains a modified version of Decagon with the necessary pipeline to train the model with real data. This version uses the [original dataset](http://snap.stanford.edu/decagon/) enriched with algorithmic complexity features computed from the graph structure of the dataset. To run the model with the proposed dataset, it is necessary to have [Singularity](https://sylabs.io/guides/3.0/user-guide/installation.html) installed.

![alt text](https://github.com/diitaz93/polypharm_predict/blob/main/images/pipeline.png "Pipeline")

## Singularity container
The pipeline used in this project involves code in different versions of Python. To make a soft transition between the different version without incompatibilities we propose the use of Singularity containers. Before training it is necessary to build the container images using the [definition files](https://github.com/diitaz93/polypharm_predict/tree/main/containers).
```
$ singularity build --fakeroot decagon.sif decagon.def
$ singularity build --fakeroot python3.sif python3.def
$ singularity build --fakeroot GPU.sif decagon-gpu.def
```

## Pipeline
The following are the steps to train the model:

1. Download data.
```
$ ./data/download_data.sh
```
2. Get into a Python 3 Singularity container.
```
$ singularity shell python3.sif
```
3. Generate graph structures. Run the Python script `DS_generator.py` that gets as parameters the number of polypharmacy side effects.
```
$ cd data
$ ./DS_generator.py 964
```
4. Calculate the algorithmic complexity features of the dataset. Run one script for each dataset provided. Add as parameter the path to the `DS` file previously created.
```
$ ./ppi_bdm.py <path_to_graph_structure_file>
$ ./dti_bdm.py <path_to_graph_structure_file>
$ ./ddi_bdm.py <path_to_graph_structure_file>
```
5. Sparsify complexity feature vectors. Run the python script `bin_BDM.py` and add as parameter the number of genes, drugs ans polypharmacy side effects. 
```
$ ./bin_BDM.py <n_genes> <n_drugs> <n_se>
```
6. Calculate the data structures needed to train the graph convolutional encoder. Run the python script `DECAGON_struct.py` and add as parameter the path to the `DS` file. Use the flags `--dse` and `--bdm` followed by a non-zero int or a true boolean to use single drug side effects and algorthmic complexity features, respectively.
```
$ ./DECAGON_struct.py <path_to_graph_structure_file> --dse 1 --bdm 1
```
7. Exit the Python 3 Singularity container and get into a Python 2 container.
```
exit # exit previous container
$ singularity shell decagon.sif
```
8. Go to the main folder and run the python script `MINIBATCH_saving.py` and add as parameter the path to the `DECAGON` file previously created, the batch size and the fraction of the dataset used for validation and test.
```
$ cd ..
$ ./MINIBATCH_saving.py <path_to_data_structure_file> 512 0.15
```
9. If the model is going to be trained in GPU, change to the GPU container.
```
$ exit
$ singularity shell --nv GPU.sif
```
10. Train the model by running either `main.py` or `main_gpu.py`, which receives as parameters the path to the `DECAGON` file. The `main_gpu.py` additionally receives the relative id of the desired GPU. Use the flags `--epochs` and `--batch_size` to add number of epochs and batch size repectively.
```
$ ./main.py <path_to_data_structure_file> --epochs 50 --batch_size 512
```
or
```
$ ./main_gpu.py <path_to_data_structure_file> 0 --epochs 50 --batch_size 512
```
The performances of the model will be saved in the folder `results_training`.