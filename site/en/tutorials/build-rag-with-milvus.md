---
id: build-rag-with-milvus.md
summary: build rag with milvus
title: Build RAG with Milvus
---

# Build RAG with Milvus

<a href="https://colab.research.google.com/github/milvus-io/bootcamp/blob/master/bootcamp/tutorials/quickstart/build_RAG_with_milvus.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/></a>

In this tutorial, we will show you how to build a RAG(Retrieval-Augmented Generation) pipeline with Milvus.

The RAG system combines a retrieval system with a generative model to generate new text based on a given prompt. The system first retrieves relevant documents from a corpus using a vector similarity search engine like Milvus, and then uses a generative model to generate new text based on the retrieved documents.


## Preparation
### Dependencies and Environment


```python
$ pip install --upgrade pymilvus openai requests tqdm
```

<div class="alert note">

If you are using Google Colab, to enable dependencies just installed, you may need to **restart the runtime**.

</div>

We will use OpenAI as the LLM in this example. You should prepare the [api key](https://platform.openai.com/docs/quickstart) `OPENAI_API_KEY` as an environment variable.


```python
import os

os.environ["OPENAI_API_KEY"] = "sk-***********"
```

### Prepare the data

We use the [Milvus development guide](https://github.com/milvus-io/milvus/blob/master/DEVELOPMENT.md) to be as the private knowledge in our RAG, which is a good data source for a simple RAG pipeline.

Download it and save it as a local text file.


```python
import json
import urllib.request

url = "https://raw.githubusercontent.com/milvus-io/milvus/master/DEVELOPMENT.md"
file_path = "./Milvus_DEVELOPMENT.md"

if not os.path.exists(file_path):
    urllib.request.urlretrieve(url, file_path)
```

We simply use "# " to separate the content in the file, which can roughly separate the content of each main part of the markdown file.


```python
with open(file_path, "r") as file:
    file_text = file.read()

text_lines = file_text.split("# ")
```

### Prepare the Embedding Model

We initialize the OpenAI client to prepare the embedding model.


```python
from openai import OpenAI

openai_client = OpenAI()
```

Define a function to generate text embeddings using OpenAI client. We use the [text-embedding-3-small](https://platform.openai.com/docs/guides/embeddings) model as an example.


```python
def emb_text(text):
    return (
        openai_client.embeddings.create(input=text, model="text-embedding-3-small")
        .data[0]
        .embedding
    )
```

Generate a test embedding and print its dimension and first few elements.


```python
test_embedding = emb_text("This is a test")
embedding_dim = len(test_embedding)
print(embedding_dim)
print(test_embedding[:10])
```

    1536
    [0.009907577186822891, -0.0055520725436508656, 0.006800490897148848, -0.0380667969584465, -0.018235687166452408, -0.04122573509812355, -0.007634099572896957, 0.03221159428358078, 0.0189057644456625, 9.491520904703066e-05]


## Load data into Milvus

### Create the Collection


```python
from pymilvus import MilvusClient

milvus_client = MilvusClient("./milvus_demo.db")

collection_name = "my_rag_collection"
```

Check if the collection already exists and drop it if it does.


```python
if milvus_client.has_collection(collection_name):
    milvus_client.drop_collection(collection_name)
```

Create a new collection with specified parameters. 

If we don't specify any field information, Milvus will automatically create a default `id` field for primary key, and a `vector` field to store the vector data. A reserved JSON field is used to store non-schema-defined fields and their values.


```python
milvus_client.create_collection(
    collection_name=collection_name,
    dimension=embedding_dim,
    metric_type="IP",  # Inner product distance
    consistency_level="Strong",  # Strong consistency level
)
```

### Insert data
Iterate through the text lines, create embeddings, and then insert the data into Milvus.

Here is a new field `text`, which is a non-defined field in the collection schema. It will be automatically added to the reserved JSON dynamic field, which can be treated as a normal field at a high level.


```python
from tqdm import tqdm

data = []

for i, line in enumerate(tqdm(text_lines, desc="Creating embeddings")):
    data.append({"id": i, "vector": emb_text(line), "text": line})

milvus_client.insert(collection_name=collection_name, data=data)
```

    Creating embeddings: 100%|█| 47/47 [00:16<00:00,  





    {'insert_count': 47,
     'ids': [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46],
     'cost': 0}



## Build RAG

### Retrieve data for a query

Let's define a query question about the content of the development guide documentation.


```python
question = "what is the hardware requirements specification if I want to build Milvus and run from source code?"
```

Search for the question in the collection and retrieve the semantic top-3 matches.


```python
search_res = milvus_client.search(
    collection_name=collection_name,
    data=[
        emb_text(question)
    ],  # Use the `emb_text` function to convert the question to an embedding vector
    limit=3,  # Return top 3 results
    search_params={"metric_type": "IP", "params": {}},  # Inner product distance
    output_fields=["text"],  # Return the text field
)
```

Let's take a look at the search results of the query



```python
retrieved_lines_with_distances = [
    (res["entity"]["text"], res["distance"]) for res in search_res[0]
]
print(json.dumps(retrieved_lines_with_distances, indent=4))
```

    [
        [
            "Hardware Requirements\n\nThe following specification (either physical or virtual machine resources) is recommended for Milvus to build and run from source code.\n\n```\n- 8GB of RAM\n- 50GB of free disk space\n```\n\n##",
            0.7718855142593384
        ],
        [
            "Building Milvus on a local OS/shell environment\n\nThe details below outline the hardware and software requirements for building on Linux and MacOS.\n\n##",
            0.7328144907951355
        ],
        [
            "Software Requirements\n\nAll Linux distributions are available for Milvus development. However a majority of our contributor worked with Ubuntu or CentOS systems, with a small portion of Mac (both x86_64 and Apple Silicon) contributors. If you would like Milvus to build and run on other distributions, you are more than welcome to file an issue and contribute!\n\nHere's a list of verified OS types where Milvus can successfully build and run:\n\n- Debian/Ubuntu\n- Amazon Linux\n- MacOS (x86_64)\n- MacOS (Apple Silicon)\n\n##",
            0.6443224549293518
        ]
    ]


### Use LLM to get a RAG response

Convert the retrieved documents into a string format.


```python
context = "\n".join(
    [line_with_distance[0] for line_with_distance in retrieved_lines_with_distances]
)
```

Define system and user prompts for the Lanage Model. This prompt is assembled with the retrieved documents from Milvus.


```python
SYSTEM_PROMPT = """
Human: You are an AI assistant. You are able to find answers to the questions from the contextual passage snippets provided.
"""
USER_PROMPT = f"""
Use the following pieces of information enclosed in <context> tags to provide an answer to the question enclosed in <question> tags.
<context>
{context}
</context>
<question>
{question}
</question>
"""
```

Use OpenAI ChatGPT to generate a response based on the prompts.


```python
response = openai_client.chat.completions.create(
    model="gpt-3.5-turbo",
    messages=[
        {"role": "system", "content": SYSTEM_PROMPT},
        {"role": "user", "content": USER_PROMPT},
    ],
)
print(response.choices[0].message.content)
```

    The hardware requirements specification for building Milvus and running it from the source code are as follows:
    
    - 8GB of RAM
    - 50GB of free disk space

