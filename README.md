# A GraphRAG Framework for Natural Language Question Answering over Knowledge Graphs
Experimental evaluation of a proposed GraphRAG system for querying Knowledge Graphs with LLMs. The study benchmarks GPT-3.5-turbo, GPT-4o, GPT-5.5, Mistral Small across 34 query types (18 SELECT, 16 ASK), including 17 reasoning-based queries, with each type comprising 25 questions evaluated over 4 prompt levels—resulting in 13,600 experimental configurations.
Graph RAG: Microsoft https://github.com/microsoft/graphrag

## Prerequisites & Setup

Before running this project, you need to obtain API keys and configure your environment:

### Required API Keys

1. **OpenAI API Key**
   - Get your key from: https://platform.openai.com/api-keys
   - Required for GPT-3.5-turbo, GPT-4o, and GPT-5.5 models

2. **Mistral API Key**
   - Get your key from: https://console.mistral.ai/api-keys/
   - Required for Mistral Small-Latest model

### Installation & Running

#### 1. Install Dependencies
```bash
pip install graphrag langchain_openai openai mistralai llama-index
```

#### 2. Set Environment Variables
Before running any notebook or script, set your API keys as environment variables:

**On Windows (Command Prompt):**
```cmd
set OPENAI_API_KEY=your_openai_key_here
set MISTRAL_API_KEY=your_mistral_key_here
```

**On Windows (PowerShell):**
```powershell
$env:OPENAI_API_KEY="your_openai_key_here"
$env:MISTRAL_API_KEY="your_mistral_key_here"
```

**On macOS/Linux (Bash/Zsh):**
```bash
export OPENAI_API_KEY=your_openai_key_here
export MISTRAL_API_KEY=your_mistral_key_here
```

#### 3. Running the Code

**On Google Colab:**
1. Upload or open the notebook: `GraphRAG_Framework_for_Natural_Language_Question_Answering_over_Knowledge_Graphs.ipynb`
2. Set environment variables in the first cell before running:
   ```python
   import os
   os.environ["OPENAI_API_KEY"] = "your_openai_key_here"
   os.environ["MISTRAL_API_KEY"] = "your_mistral_key_here"
   ```
3. Run all cells sequentially from top to bottom

**On Local Machine:**
```bash
jupyter notebook
```
Then open `initial.ipynb` or the main notebook file.

**Note:** The code will read the `OPENAI_API_KEY` and `MISTRAL_API_KEY` environment variables at runtime through `os.environ["OPENAI_API_KEY"]` and similar calls in the notebook.

### Environment Variables in Code

The code uses these environment variables internally:
```python
import os
api_key = os.environ["OPENAI_API_KEY"]
mistral_key = os.environ["MISTRAL_API_KEY"]
```

Ensure both variables are set before running the notebooks or Python scripts.

---

## Dataset: UMLS Knowledge Graph Construction

The evaluation employs a **new provided UMLS-based knowledge graph** containing both **TBox (terminological)** and **ABox (assertional)** components, essential for assessing GraphRAG's ability to handle both graph structure and deductive reasoning.

### Data Source & Processing

The UMLS dataset is constructed using official UMLS RRF (Rich Release Format) files:
- **MRREL.RRF** – Relations between concepts (predicates)
- **MRCONSO.RRF** – Concept-to-term mappings (CUI → human-readable terms)
- **MRSTY.RRF** – Semantic types for each concept
- **MRDEF.RRF** – Natural language definitions for entities

UMLS concepts (CUIs) are mapped to human-readable terms, producing natural language triples. Semantic type information reconstructs the ontological structure and taxonomy.

### Filtering & Entity Descriptions

To maintain dataset quality and entity completeness:
- **Structure-preserving filtering** – Retains entity completeness and relational diversity across all predicates
- **Entity-centric stratification** – Ensures triples involving the same entity across multiple predicates are retained
- **Definition-based descriptions** – Entity descriptions are derived from MRDEF.RRF concept definitions (avoiding long connection chains that could complicate GraphRAG context attachment)

The final dataset is converted to RDF format and imported into GraphDB, forming a comprehensive KG suitable for evaluating diverse query types across reasoning patterns.

---

## Query Taxonomy
Structural Queries – focusing on explicit ABox/TBox graph exploration (e.g., instance retrieval, class and property hierarchy navigation)
Reasoning-Oriented Queries – requiring ontology-aware inference such as subclass propagation, domain/range constraints, subproperty inheritance, and combined ABox–TBox reasoning
For both structural and reasoning-oriented categories, we include SELECT-based retrieval and ASK-based validation, enabling a comprehensive assessment of our proposed GraphRAG system

## Standard SELECT Query Taxonomy

| Query ID | Query Name | Description | SPARQL Pattern |
|----------|------------|------------|----------------|
| 1 | Outgoing Predicate Instantiation | Retrieving all objects linked from a given subject via a specified predicate | `SELECT ?object WHERE { @node @predicate ?object.}` |
| 2 | Incoming Predicate Instantiation | Retrieving all subjects connected to a specified object via a given predicate | `SELECT ?subject WHERE { ?subject @predicate @node. }` |
| 3 | Undirected Predicate Instantiation | Retrieving all triples where a given predicate connects a specified entity as subject or object | `SELECT ?ent WHERE {{ @node @relationship ?ent.} UNION { ?ent @relationship @node. }}` |
| 4 | Direct Entity-to-Entity Relation Retrieval | Retrieving predicates that directly connect two specified entities | `SELECT ?predicate WHERE {{ @node1 ?predicate @node2. } UNION { @node2 ?predicate @node1. }}` |
| 5 | Subject-Centric Statement Retrieval | Retrieving all triples where a specified entity appears as subject | `SELECT ?predicate ?target WHERE { @node ?predicate ?target. }` |
| 6 | Predicate-Centric Connectivity Retrieval | Retrieving all entity pairs connected by a specified predicate, regardless of direction | `SELECT ?source ?target WHERE {{ ?source @predicate ?target.} UNION { ?target @predicate @source. }}` |
| 7 | Full Entity Description | Retrieving all triples where the entity appears as subject or object | `DESCRIBE @x` |


## Standard ASK Query Taxonomy

| Query ID | Query Name | Description | SPARQL Pattern |
|----------|------------|------------|----------------|
| 8 | Fully Grounded Triple Verification | Checking whether a fully specified triple exists | `ASK WHERE { @subject @predicate @object.}` |
| 9 | Subject-Predicate Relation Existence | Checking whether a given subject participates in at least one statement with a specified predicate | `ASK WHERE { @subject @predicate ?object.}` |
| 10 | Predicate-Object Relation Existence | Checking whether at least one subject is connected to a specified object via a given predicate | `ASK WHERE { ?subject @predicate @object.}` |
| 11 | Undirected Predicate Neighborhood | Checking whether a specified entity participates in a given predicate either as subject or as object | `ASK WHERE {{ @object @predicate ?subject.} UNION { ?subject @predicate @object.}}` |
| 12 | Any Predicate Between Two Entities | Checking whether two specified entities are directly connected by any predicate | `ASK WHERE { @subject ?predicate @object.}` |
| 13 | Undirected Entity Connectivity | Checking whether two specified entities are connected by any predicate in either direction | `ASK WHERE {{ @subject ?predicate @object.} UNION { @object ?predicate @subject. }}` |
| 14 | Subject Participation in Any Statement | Checking whether a specified entity appears as subject in any statement | `ASK WHERE { @subject ?predicate ?object.}` |
| 15 | Object Participation in Any Statement | Checking whether a specified entity appears as object in any statement | `ASK WHERE { ?subject ?predicate @object.}` |
| 16 | Predicate Usage Existence | Checking whether a specified predicate is used in at least one statement | `ASK WHERE { ?subject @predicate ?object.}` |
| 17 | Entity Global Mention Detection | Checking whether an entity appears anywhere in the graph as subject, predicate, or object | `ASK WHERE { ?sub ?prop ?val. FILTER ( @subject = ?sub || @subject = ?prop || @subject = ?val )}` |

## Reasoning SELECT Query Taxonomy

| ID | Name | Description | SPARQL Pattern |
|----|------|------------|----------------|
| 18 | Type Assertion (Forward) | Reading the (possibly inferred) type of a known entity | `SELECT ?object WHERE { @subject rdf:type ?object.}` |
| 19 | Type Inversion (Backward) | Reading all entities that instantiate a given class | `SELECT ?subject WHERE { ?subject rdf:type @object.}` |
| 20 | Subclass Discovery (Downward) | Reading the more specific classes of a known class | `SELECT ?subject WHERE { ?subject rdfs:subClassOf @object.}` |
| 21 | Superclass Discovery (Upward) | Reading the more general classes of a known class | `SELECT ?object WHERE { @subject rdfs:subClassOf ?object.}` |
| 22 | Property Generalization (Upward) | Reading the more general properties of a known property | `SELECT ?predicate WHERE { @predicate rdfs:subPropertyOf ?predicate.}` |
| 23 | Property Specialization (Downward) | Reading the more specific properties of a known property | `SELECT ?predicate WHERE { ?predicate rdfs:subPropertyOf @predicate.}` |
| 24 | Typed Relation Filtering | Reading pairs of entities connected by a relationship, constrained by inferred types | `SELECT ?subject ?object WHERE { ?subject @predicate ?object. ?subject rdf:type @subjectType. ?object rdf:type @objectType. }` |
| 25 | Object Type Inference (Range) | Reading the type of an object implied by predicate range | `SELECT ?object WHERE { @subject @predicate @object. @object rdf:type ?object. }` |
| 26 | Object Class Abstraction (Range) | Reading the most general class implied via predicate range | `SELECT ?object WHERE { @subject @predicate @object. @predicate rdfs:range ?object.}` |
| 27 | Subject Type Inference (Domain) | Reading the type of a subject implied by predicate domain | `SELECT ?object WHERE { @subject @predicate @object. @subject rdf:type ?object.}` |
| 28 | Subject Class Abstraction (Domain) | Reading the most general class implied via predicate domain | `SELECT ?object WHERE { @subject @predicate @object. @predicate rdfs:domain ?object.}` |


## Reasoning ASK Query Taxonomy

| Query ID | Query Name | Description | SPARQL Pattern |
|----------|------------|------------|----------------|
| 29 | Range Compatibility | Checking subject compatibility with range | `ASK WHERE { @predicate rdfs:range ?object. @subject rdfs:subClassOf ?object.}` |
| 30 | Domain Compatibility | Checking subject compatibility with domain | `ASK WHERE { @predicate rdfs:domain ?object. @subject rdfs:subClassOf ?object.}` |
| 31 | Instance Type Assertion | Checking instance relation | `ASK WHERE { @subject rdf:type @object.}` |
| 32 | Subclass Assertion | Checking subclass relation | `ASK WHERE { @subject rdfs:subClassOf @object.}` |
| 33 | Typed Existential Relation | Checking typed relation | `ASK WHERE { @subject @predicate ?object. ?object rdf:type @object.}` |
| 34 | Subproperty Assertion | Checking subproperty relation | `ASK WHERE { @subject rdfs:subPropertyOf @object.}` |

LLMs utilized: GPT-3.5-Turbo,  GPT-4o, GPT-5.5, Mistrall-Small-Latest



