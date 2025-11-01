# Knowledge Bases via Minds SDK

With the Knowledgebase SDK, we wanted to it dead simple to build Large Scale Semantic Search on top MindsDB, as such the default configuration is designed for the simplest wat to: load, search, and get answers from large amounts of unstructured data in just a few lines of code. 


> **NOTE:** We'll dive into `expert-mode` soon—for those who prefer coding with their hoodie up, tinkering with every last detail- But first, let's enjoy the simplicity of the default `hassle-zero` mode.



```python
from minds_sdk import Client

# Example for seamntic search over SEC Filings 
# Load a pre-existing knowledge base 'sec_filings'
kb = Client(base_url=.., api_key=..).kb('sec_filings')

# Search
results = kb.search("Quarterly reports for NVIDIA during H2 2024")

# Analyze results
answer = results.analyze("What changed in revenue?")

# Unbound analysis over the entire KB
answer = kb.analyze("Quarterly revenue for NVIDIA over the past 5 years")

# Search with literal metadata filters
results = kb.search("NVIDIA during H2 2024", report_type="Quarterly")
```

The goal with this part of the SDK is simple: **ask a question and get the answer your need**—`fast`. As such; the default for `.search(<plain language query>)` method auto-magically determines hybrid metadata filtering and semantic search over unstructured data to return the most relevant results. If, instead of a list results you want a direct answer from either your search results or the entire knowledge base, use `.analyze(<plain language question>)`. That's it folks! No agents bs, no fuss—just answers.



## How to create a knowledge base from scratch

Creating knowledge bases is as simple as a few lines of code too.
For example, this is how to created the KB from PDFs containing SEC filings that we searched previously:

```python
from minds_sdk import Client
from pathlib import Path
import datetime
from pydantic import BaseModel

class FilingSchema(BaseModel):
    report_type: str
    company: str
    filing_date: datetime.date
    most_relavant_data_json: dict  
    summary: str

# Simply insert anything you want into the KB
kb = Client(<api_key>).kb.create('sec_filings', FilingSchema)
for pdf_file in Path("quarterly_filings_folder").glob("*.pdf"):
    kb.insert(pdf_file, report_type='quarterly')  
```



When inserting into a Knowledge Base, unless you say otherwise, MindsDB Server works on your behalf to handle all the heavy lifting most people gladly skip:
- Extracts info from files (PDFs, etc.) 
- Tames messy content to fit into the schema you provide, merging it with whatever metadata you specify
- Indexes your text attributes for lightning-fast semantic search

> Every `insert` request is managed asynchronously, making it very fast to send large amounts of data into a Knowledge Base.


The `insert` method is designed for maximum flexibility and ease-of-use.

- The *first unnamed argument* is assumed to be the main content you want to insert into the knowledge base. This can be:
  - Raw text (`str`)
  - A file pointer (e.g., opened PDF, image, etc.)
  - A valid HTTP(S) URL (`HttpUrl` string)
  - Or a Pydantic object matching your schema.

- **Named arguments** let you set any attribute directly (`key=<value>`) and skip auto-fill for that specific attribute (`<value>` can be `None`). 

    - If you don't specify an `id`, MindsDB will automatically generate one by taking an MD5 hash of the content. 
    - Globally disable autofill using `_auto_fill=False`.



> **Tip:** For best auto-extraction accuracy, document your schema attributes in the Pydantic class using descriptive field docs.

Example: Explicit no auto-fill
```python
kb.insert(
    file_pointer, 
    file_name="filing.pdf", 
    company = 'NVIDIA',
    _auto_fill=False  # skip autofill for all other missing fields
)
```
In summary, just call `insert` with your content and whatever schema fields you have. MindsDB handles the rest—validation, auto-filling, hashing, and smart LLM extraction—unless you tell it not to.
