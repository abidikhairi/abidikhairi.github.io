---
title: "How To Write Custom PyTorch Datasets"
description: "A tutorial on how to write custom PyTorch datasets."
date: 2022-09-04T12:19:19:00
image: pytorch-cover.png
math: 
categories:
    - Tutorials
tags:
    - pytorch
---

## Introduction

In this tutorial, we will go over the basics of writing custom PyTorch datasets. We will go over the `Dataset` and `DataLoader` classes and how they are used to fetch data. We will also cover how to write custom datasets that can be used with PyTorch's `DataLoader` class.

## The `Dataset` Class

The `Dataset` class is an abstract base class that represents a dataset. It is the base class for all other datasets. The `Dataset` class has two abstract methods that need to be overridden:

* `__len__` which is used to get the size of the dataset.
* `__getitem__` which is used to get an item from the dataset.

The `__len__` method takes no arguments and returns the size of the dataset. The `__getitem__` method takes an index as an argument and returns the item at that index.

The `Dataset` class also implements the `__iter__` method which returns an iterator over the dataset. The `__iter__` method is implemented using the `__getitem__` method. The iterator returned by the `__iter__` method returns the items of the dataset one at a time. The iterator raises a `StopIteration` exception when there are no more items to return. The `__getitem__` method is used to implement the `__iter__` method. The `__getitem__` method is also used to implement indexing. The `__getitem__` method is called whenever an item is retrieved from the dataset using indexing. For example, the following code retrieves the first item from the dataset:

```python
dataset[0]
```

The `__getitem__` method is also called when the dataset is iterated over:

```python
for item in dataset:
    print(item)
```

The `__getitem__` method can be called using the `dataset[i]` syntax. The `__getitem__` method can also be called using the `iter(dataset)` syntax. The `__getitem__` method is also called by the `next(iter(dataset))` syntax. The `next` function calls the `__iter__` method of the dataset to get an iterator over the dataset. The `__iter__` method returns an iterator which is used to iterate over the dataset.


## Dummy Dataset
Suppose we have a dataset $\mathcal{D} = \{(x_1, y_1), \dots, (x_m, y_m)\}$ that contains pairs of features $x_i \in \mathbb{R}^4$ and target $y_i \in \mathbb{R}$. We can create a dataset class that represents this dataset, *i.e,* `dataset[i]` returns the pair $(x_i, y_i)$, as follows:


```python
import torch as th
from torch.utils.data import Dataset

class DummyDataset(Dataset):
    def __init__(self, size):
        self.size = size
        self.features = th.randn(size, 4)
        self.targets = th.randn(size)

    def __len__(self):
        return self.size

    def __getitem__(self, index):
        x = self.features[index]
        y = self.targets[index]
        return x, y

idx = 5
dataset = DummyDataset(100)
item = dataset[idx]
print(item)
```
```
    (tensor([ 0.2027, -0.2010,  0.2010,  0.2010]), tensor(-0.2010))
```

## Cats-Dogs Kaggle Dataset
The [Cats-vs-Dogs](https://www.kaggle.com/datasets/shaunthesheep/microsoft-catsvsdogs-dataset) dataset is a dataset that contains images of cats and dogs. The dataset contains 25,000 images of cats and dogs. The dataset is available on Kaggle and it can be downloaded from [this link](https://www.kaggle.com/datasets/shaunthesheep/microsoft-catsvsdogs-dataset).

After downloading the dataset and extracting the archive file (`archive.zip`), a `PetImages` folder is extracted containing two subfolders `Cat` and `Dog`. The `Cat` folder contains 12,500 images of cats and the `Dog` folder contains 12,500 images of dogs. The images in the `Cat` folder are named `0.jpg`, `1.jpg`, ..., `12499.jpg`. The images in the `Dog` folder are named `0.jpg`, `1.jpg`, ..., `12499.jpg`. 

Next, we create a csv file containing the metadata and the labels for the images. The csv file contains two columns: `path` and `label`. The `path` column contains the path to the image and the `label` column contains the label for the image. The label is `0` for cats and `1` for dogs. The csv file is saved as `metadata.csv`. The following code creates the csv file:

```python
import os
import pandas as pd

def create_metadata_file(root_dir):
    root_dir = os.path.realpath(root_dir)
    subdirs = os.listdir(root_dir) # ['Cat', 'Dog']

    data = {
        "path": [],
        "label": []
    }

    for label, subdir in enumerate(subdirs):
        subdir_path = os.path.join(root_dir, subdir)
        for file in os.listdir(subdir_path):
            file_path = os.path.join(subdir_path, file)
            data["path"].append(file_path)
            data["label"].append(label)
        
    df = pd.DataFrame(data)

    df.to_csv(os.path.join(root_dir, "metadata.csv"), index=False)


if __name__ == '__main__':
    root_dir = "./data/PetImages" # path to PetImages folder 
    create_metadata_file(root_dir)
    # the metadata.csv file will be created in the PetImages folder
```

Once the metadata file is created we can create a dataset class that represents the Cats-vs-Dogs dataset. The `__getitem__` method should return the image and the label for the image at the given index. The `__getitem__` method should return the image as a `torch.Tensor` object representing the image and the label as a `torch.Tensor` object. The following code creates the dataset class:

```python
import pandas as pd
import torch as th
from PIL import Image
from torch.utils.data import Dataset


class CatDogDataset(Dataset):
    def __init__(self, metadata_file, transform=None):
        self.metadata = pd.read_csv(metadata_file) 
        self.transform = transform
    
    def __len__(self):
        return len(self.metadata)

    def __getitem__(self, index):
        path = self.metadata.iloc[index, 0]
        label = self.metadata.iloc[index, 1]

        image = Image.open(path).convert('RGB')
        label = th.tensor(label)

        if self.transform is not None:
            image = self.transform(image)

        return image, label

if __name__ == '__main__':
    transform = transforms.Compose([
        transforms.Resize((224, 224)), # resize image to 224x224
        transforms.ToTensor() # convert to tensor
    ])
    metadata_file = "./data/PetImages/metadata.csv"
    dataset = CatDogDataset(metadata_file, transform=transform)
    print(len(dataset))
```
```
    25000
```
```python
    id = 5
    image, label = dataset[id]
    print(image.shape, label)
```
```
    (torch.Size([3, 224, 224]), tensor(0))
```