# KG Course Competition on Inductive Link Prediction

This inductive link prediction competition accompanies the [KG Course](https://github.com/migalkin/kgcourse2021) and 
welcomes students' attempts to improve the link prediction performance on a newly constructed dataset.

This repo contains:
* The dataset in the `./data` folder
* A boilerplate code with 2 baselines that you can base your implementations on

## Installation

The code employs the [PyKEEN](https://github.com/pykeen/pykeen) framework for training KG link prediction models.

Main requirements:
* python >= 3.9
* torch >= 1.10

You will need PyKEEN 1.8.0 or newer.
```shell
$ pip install pykeen
```

By the time of creation of this repo 1.8.0 is not yet there, but the latest version from sources contains
everything we need
```shell
$ pip install git+https://github.com/pykeen/pykeen.git
```

If you plan to use GNNs (including the `InductiveNodePieceGNN` baseline) make sure you install [torch-scatter](https://github.com/rusty1s/pytorch_scatter)
and [torch-geometric](https://github.com/pyg-team/pytorch_geometric) 
compatible with your python, torch, and CUDA versions.

Running the code on a GPU is strongly recommended.

## Dataset
Inductive link prediction is different from the standard transductive task in a way that at inference time
you are given a new, unseen graph with unseen entities (but known relation types). 
Here is the schematic description of the task:

![](https://pykeen.readthedocs.io/en/latest/_images/ilp_1.png)

The dataset in `./data` consists of 4 splits:
* `train.txt` - the training graph on which you are supposed to train a model
* `inference.txt` - the inductive inference graph **disjoint** with the training one - that is, it has a new non-overlapping set of entities, the missing links are sampled from this graph
* `inductive_validation.txt` - validation set of triples to predict, uses entities from the **inference** graph
* `inductive_test.txt` - test set of triples to predict, uses entities from the **inference** graph
* a held-out test set of triples - kept by the organizers for the final ranking 😉 , uses entities from the **inference** graph

## Baselines

Training shallow entity embeddings in this setup is useless as trained embeddings cannot be used for inference over unseen entities.
That's why we need new representation learning mechanisms - in particular, we use [NodePiece](https://arxiv.org/abs/2106.12144) for the baselines.

NodePiece in the inductive mode will use the set of relations seen in the training graph to *tokenize* entities in the training and inference graphs.
We can afford tokenizing the nodes in the *inference* graph since the set of relations **is shared** between training and inference graphs 
(more formally, the set of relations of the inference graph is a subset of training ones).

We offer here 2 baselines:
* `InductiveNodePiece` - plain tokenizer + tokens MLP encoder to bootstrap node representations. Fast.
* `InductiveNodePieceGNN` - everything above + an additional 2-layer [CompGCN](https://arxiv.org/abs/1911.03082) message passing encoder with the attention mechanism. Slower but attains higher performance.

For more information on the models check out the [PyKEEN tutorial](https://pykeen.readthedocs.io/en/latest/tutorial/inductive_lp.html) on inductive link prediction with NodePiece

Both baselines are implemented in the `main.py`. 

CLI arguments:

```shell
Usage: main.py [OPTIONS]

Options:
  -dim, --embedding_dim INTEGER  
  -tokens, --tokens_per_node INTEGER  # for NodePiece
  -lr, --learning_rate FLOAT
  -m, --margin FLOAT  # for the margin loss and SLCWA training
  -negs, --num_negatives INTEGER  # negative samples per positive in the SLCWA regime 
  -b, --batch_size INTEGER
  -e, --num_epochs INTEGER
  -wandb, --wandb BOOLEAN
  -save, --save_model BOOLEAN
  -gnn, --gnn BOOLEAN  # for activating InductiveNodePieceGNN
```

## Baselines Performance

| **Model**             | MRR (Inverse Harmonic Mean Rank) | Hits @ 10 | Hits @ 5 | Hits @ 3 | Hits @ 1 | Mean Rank |
|-----------------------| -------------------------------- | --------- | -------- | -------- | -------- | --------- |
| InductiveNodePieceGNN |                                  |           |          |          |          |           |
| InductiveNodePiece    |                                  |           |          |          |          |           |


## Submissions

1. Fork the repo
2. Train your inductive link prediction model
3. Save the model weights using the `--save True` flag
4. Upload model weights on GitHub or other platforms (Dropbox, Google Drive, etc)
5. Open an issue in **this** repo with the link to your repo and model weights

