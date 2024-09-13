---
title: "Using ImageBind and ChromaDB to create an image search engine"
date: "2024-09-05"
---

In May 2023, Meta released the [ImageBind](https://github.com/facebookresearch/ImageBind) embedding model.
What is unique about this model is, that it embeds text, images, video and audio in the same embedding space.
This enables us to compare vectors across different modalities.

# Why
I ran into a problem when searching for images of a particular kind of stone age antler artifact. The danish 
national museum has a search function to find pictures of all their artifacts, but you're searching in the
metadata that human annotators have written. This metadata is often relatively sparse, for many reasons.
But most images where simply labeled as "anter hammer" or similar, making it very difficult to search for
artifacts with specific features. When I read about ImageBind it struck me, that the embedding of an image
probably contains more information than that, which we can search for using text.

# How
ImageBind takes a piece of text, an image, a sound or a video, and creates a 1024-dimensional vector out of it. This is similar to classical word embedding models, which has been used for a while now. The idea is, that similar objects should end up having vectors that are close to each other. Thus, to find a certain image, we just need a piece of text that describes it. This text should embed to a vector close to the desired image.

We need two components to make this work: the embedding model and a way to find vectors, that a similar to a given query vector. The embedding model has already been covered, we'll use ImageBind. To find similary vector, we'll use ChromaDB. The reason for this is that it is easy to set up and is supposed to scale pretty well, whould that be needed.

I whipped up the following command line tool:

```python
from chromadb.api.models.CollectionCommon import Embedding
from imagebind import data
import torch
import numpy as np
from imagebind.models import imagebind_model
from imagebind.models.imagebind_model import ModalityType
import chromadb
import argparse

parser = argparse.ArgumentParser(description="A image search engine powered by ImageBind")

parser.add_argument("-n", "--number", help="Number of search results returned", type=int, default=2)
parser.add_argument("-c", "--create", help="Create an image database from a list of files", type=str, nargs="+")
args = parser.parse_args()

client = chromadb.PersistentClient(path="imagedb_test")
if args.create:
    try:
        client.delete_collection(name = "imagedb")
    except:
        pass

collection = client.get_or_create_collection(name="imagedb", metadata={"hnsw:space": "ip"})

device = "cuda:0" if torch.cuda.is_available() else "cpu"

print("Loading ImageBind model (this may take a while)")
# Instantiate model
model = imagebind_model.imagebind_huge(pretrained=True)
model.eval()
model.to(device)


if args.create:
    image_paths = args.create
    print(f"Creating a database with the following images: {image_paths}")

    print("Embedding images. This may also take a while")
    images = {ModalityType.VISION: data.load_and_transform_vision_data(image_paths, device)}
    with torch.no_grad():
        embeddings = model(images)

    collection.add(
        embeddings = embeddings[ModalityType.VISION].tolist(),
        documents = image_paths,
        ids = [f"{i}" for i in range(len(image_paths))]
    )

while True:
    query = input("Enter query: (X to exit): ")
    if query == "X":
        exit()
    query_text = [query]
    query_input = {ModalityType.TEXT: data.load_and_transform_text(query_text, device)}
    with torch.no_grad():
        query_embedding = model(query_input)

    res = collection.query(
        query_embeddings = [query_embedding[ModalityType.TEXT].tolist()[0]],
        n_results = args.number
    )
    print(res)
    for i in range(args.number):
        print(f"Result {i+1}")
        print(f"Document: {res['documents'][0][i]}")
        print(f"Distance: {res['distances'][0][i]}")
        print("")
```

To use it, clone the [ImageBind repo](https://github.com/facebookresearch/ImageBind), and put the script above
in a file called `imagedb.py` in the root of the repo. Then put the images you want to index into the `.assets/` folder, and create
the database like so:
```bash
$ conda activate imagebind
$ pip3 install chromadb
$ python3 imagedb.py -n 3 -c .assets/*.jpg
```
Then you can search for images, and the engine will return the three best matches.

# Evaluation
So does it work? Well, yes and no. I used the following files:
![An orange car]()
![A red car]()
![A gray car]()
![An archeological drawing of a flint arrow head]()
![An archeological drawing of a flint axe head]()

When searching for "A car", it finds the three cars. But if I search for "A red car", it ranks the orange
car above the red one. Only by searching "A red musclecar" can I get the red car to be the first result.
I was unsuccessful in ever getting the gray car as the first result.

Also, searching for "An archeological drawing of a flint (arrow/axe) head" returned the arrow head first in
both cases. This doesn't bode well for my intended application of finding archeological pictures with specific
features, if it can't even distinguish an arrow head from an axe head.

# Why did that not work?
I suspect simply finding the closest vector is too rough of a search metric. I suspect that the orange car is
somehow "more" a car than the red one, so any prompt using the word "car" will rank the orange car above
the red one. Perhaps we can gradually filter, by first finding "cars", then searching those for "red". More 
research is definitely required. For domain specific applications liek searching archeological pictures, 
some finetuing will likely improve the results, but I don't have access to the required compute. If someone
at the danish national museum of the danish national library would like to collaborate on this, please let me
know.

By the way, I'm looking for a job. If you think this sounded interesting, I'm very open to talk. You can find
me on [LinkedIn at www.linkedin.com/in/andreas-bøgh-p-37226587](www.linkedin.com/in/andreas-bøgh-p-37226587).
