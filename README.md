# Notes for usage with EnTdecker platform

Due to conflicting dependencies a new environment is required to run this code.
Use the following steps to create a new environment and install the required dependencies:

1. Create a new environment with python 3.6.0
```
conda create -n transformer-cnn python=3.6.0
conda activate transformer-cnn

pip install tensorflow==1.12.0
pip install rdkit-pypi==2021.9.4

python transformer-cnn.py config.cfg
```

# Transformer-CNN

The repository contains the source code for a new Transformer-CNN method described in our paper http://arxiv.org/abs/1911.06603. First, we trained the Transformer model on SMILES canonicalization task, e.g., given an arbitrary SMILES, the model converts it to a canonical one. Second, we use the internal representation of the Transformer (the output of the encoding stack with shape (BATCH, LENGTH, EMBEDDING)) as SMILES embeddings and build upon them CharNN model (Convolution and HighWay as it is done in DeepChem). The resulting model works both in classification and regression settings.

The "standalone" folder provides the implementation of the Transformer-CNN model for prognosis without TensorFlow (depends on only NumPy and RDKit). The solubility and AMES models are available. [Layerwise Relevance Propagation method](https://link.springer.com/chapter/10.1007/978-3-030-28954-6_10) is used to infer the model's reasoning behind a particular prediction.

# Dependencies

The code has been tested in Ubuntu 18.04 with the following components:

1. python v.3.4.6 or higher
2. TensorFlow v1.12
3. rdkit v.2018.09.2
4. molvs for tautomer enumeration

# How to use

The main program, transformer-cnn.py, uses the config.cfg file to read all the parameters of a task to do. After filling the config.cfg with the appropriate information, launch the python3 transformer-cnn.py config.cfg

# How to train a model

To train a model, one needs to create a config file like this.
```
[Task]
   train_mode = True
   model_file = model.tar
   train_data_file = train.csv
[Details]
   canonize = True
   gpu = 0
   seed = 100666
   n_epochs = 30
   batch_size = 16
```
If the canonize parameter is set, then all the SMILES will be worked up with RDKit. Then 10 non-canonical SMILES for each molecule will be generated (the real number of generated strings can be smaller depending on the compound). If this parameter is set to False, then the string is passed to the model as is without any treatment. The same is also valid for the prognosis step.

# Using the trained model

To use a model, the config file looks like:
```
[Task]
   train_mode = False
   model_file = model.tar
   apply_data_file = predict.csv
[Details]
   canonize = True
   gpu = 0
   seed = 100666
   n_epochs = 30
   batch_size = 16
```

# Using the standalone prognosis

The "standalone" folder contains scripts and models for execution without TensorFlow. Solubility regression and AMES classification models are available. To run a prognosis for a single molecule ([haloperidol](https://www.drugbank.ca/drugs/DB00502) here as an example) execute:

python3 ochem.py models/solubility.pickle "O=C(CCCN1CCC(c2ccc(Cl)cc2)(O)CC1)c1ccc(F)cc1"

In this case, the program produces 26 random SMILES (number of non-hydrogens atoms in the molecule) starting from each atom in the original SMILES. For each SMILES a target property is predicted as well as influences of a particular atom to the overall property. The output contains:

1. the estimated property with a confidence interval.
2. the file map.txt contains a gnuplot script to visualize the individual atoms' contributions.
3. the file mol.svg contains a drawing of the molecule with atoms' contributions.

For haloperidol molecule the outpus should be:

Solubility = 0.014 ± 0.002 g/L (experimental value 14 mg/L).

And visualization:

![LRP graph](https://github.com/bigchem/transformer-cnn/blob/master/standalone/lrp/lrp.png)

The green color contributes positively to the property. The higher the bar the more the impact of the corresponding atom. The red color works in the opposite direction.

Feel free to contact us if you have any suggestions or possible applications of this code.

