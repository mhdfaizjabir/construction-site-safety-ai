# 🏗️ Data-Driven Construction Safety System

[![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://www.python.org/)
[![XGBoost](https://img.shields.io/badge/XGBoost-1.7+-orange.svg)](https://xgboost.readthedocs.io/)
[![Sentence Transformers](https://img.shields.io/badge/SentenceTransformers-2.2+-green.svg)](https://www.sbert.net/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## 📋 Overview

This project transforms construction site safety from **reactive** to **proactive** using Natural Language Processing and Machine Learning. Instead of waiting for accidents to happen, our system analyzes work descriptions to predict hazards and provide evidence-based safety recommendations.

**The Problem:** Construction sites have the highest rates of workplace accidents globally, yet most safety protocols only act AFTER an incident occurs.

**Our Solution:** A data-driven system that:
- 🔮 **Predicts** potential hazards from plain-language work descriptions
- 📚 **Retrieves** similar past accidents as evidence
- 🛡️ **Recommends** targeted safety measures

---

## 🎯 Key Features

### 1. **Hazard Classification**
- Uses `all-mpnet-base-v2` Sentence Transformer to convert text to 768-dim vectors
- XGBoost classifier achieves **83% accuracy** across 8 hazard categories
- Particularly strong on "Falls" (F1 = 0.95) and "Electrical" (F1 = 0.86)

### 2. **Semantic Evidence Retrieval**
- Finds past accidents semantically similar to the current work description
- **MRR of 0.92** – relevant results appear in top 1-2 positions
- **MAP of 0.89** – excellent ranking quality

### 3. **Rule-Based Safety Recommendations**
- OSHA-aligned safety guidelines for each hazard category
- Easily extensible to data-driven recommendations

---

## 📊 Performance Metrics

| Hazard Category | Precision | Recall | F1-Score |
|----------------|-----------|--------|----------|
| Falls | 0.95 | 0.95 | 0.95 |
| Electrical | 0.79 | 0.94 | 0.86 |
| Struck-By | 0.76 | 0.80 | 0.78 |
| Caught/Equipment | 0.69 | 0.68 | 0.68 |
| Chemical/Exposure | 0.71 | 0.71 | 0.71 |
| Fire/Explosion | 0.68 | 0.52 | 0.59 |
| Medical/Illness | 0.53 | 0.56 | 0.54 |
| Other/Animal | 0.25 | 0.33 | 0.29 |

**Overall Accuracy: 83%**

### Evidence Retrieval Performance
| Metric | Score | Interpretation |
|--------|-------|----------------|
| P@1 | 0.8333 | First result is relevant 83% of the time |
| P@3 | 0.8889 | Top 3 results are highly relevant |
| MRR | 0.9167 | First relevant result appears near the top |
| MAP | 0.8958 | Excellent overall ranking quality |
| nDCG@10 | 0.9447 | Near-ideal ranking utility |

---

## 🏗️ System Architecture

```
┌─────────────────┐
│  User Input     │  "Installing steel trusses on wet roof"
│  Work Description│
└────────┬────────┘
         ↓
┌─────────────────┐
│  Text Cleaning  │  Lowercase, remove punctuation
│  & Preprocessing│
└────────┬────────┘
         ↓
┌─────────────────┐
│  Sentence       │  all-mpnet-base-v2
│  Transformer    │  → 768-dim embedding
└────────┬────────┘
         ↓
    ┌────┴────┐
    ↓         ↓
┌──────────┐ ┌──────────────────┐
│ XGBoost  │ │ Cosine Similarity│
│Classifier│ │ to 10k incidents │
└────┬─────┘ └────────┬─────────┘
     ↓                ↓
┌──────────┐ ┌──────────────────┐
│ Hazard   │ │ Top-K Similar    │
│ Category │ │ Past Accidents   │
└────┬─────┘ └────────┬─────────┘
     ↓                ↓
┌──────────┐ ┌──────────────────┐
│ Safety   │ │ Evidence with    │
│ Rules    │ │ narratives, keywords, severity
└──────────┘ └──────────────────┘
```

---

## 📁 Repository Structure

```
📦 construction-safety-ml/
├── 📓 data_mining_codes.ipynb          # Main EDA and XGBoost classification
├── 📓 evidence_retrieval.ipynb          # Semantic search implementation
├── 📄 requirements.txt                   # Python dependencies
├── 📄 README.md                          # You are here
└── 📄 LICENSE                            # MIT License
```

### Notebook Breakdown

#### `data_mining_codes.ipynb`
- Loads and cleans OSHA dataset (53,550 → 10,000 construction incidents)
- Exploratory Data Analysis with visualizations
- Text preprocessing and embedding generation
- XGBoost classifier training and evaluation
- Model persistence (saves `.pkl` files)

#### `evidence_retrieval.ipynb`
- Implements `WorkSafetyEvidence` class
- Semantic search using cosine similarity
- Evaluation with information retrieval metrics (P@K, MRR, MAP, nDCG)
- Interactive query examples

---

## 🚀 Getting Started

### Prerequisites
- Python 3.8+
- pip package manager
- 8GB+ RAM recommended

### Installation

1. **Clone the repository**
```bash
git clone https://github.com/yourusername/construction-safety-ml.git
cd construction-safety-ml
```

2. **Install dependencies**
```bash
pip install -r requirements.txt
```

3. **Launch Jupyter**
```bash
jupyter notebook
```

### Data Requirements

The notebooks expect an Excel file named `Injury Severity.xlsx` in the root directory. This file contains OSHA Severe Injury Reports with the following key columns:
- `abstract` - Incident description
- `event_type` - OSHA event classification
- `event_keyword` - Keywords associated with the incident
- `degree_of_inj_x` - Severity level
- Weather data columns (`temp`, `pressure`, `humidity`, etc.)

> **Note:** Due to size limitations, the full dataset is not included in this repository. You can:
> - Download from [OSHA Severe Injury Reports](https://www.osha.gov/severe-injury)
> - Contact the authors for a sample
> - Use your own OSHA data with similar structure

---

## 💻 Usage Examples

### Classify a Work Description
```python
# In data_mining_codes.ipynb
query = "Workers installing roof trusses at height"
embedding = model.encode([query])
prediction = clf.predict(embedding)
hazard_type = le.inverse_transform(prediction)[0]
print(f"Predicted hazard: {hazard_type}")
```

### Find Similar Accidents
```python
# In evidence_retrieval.ipynb
system = WorkSafetyEvidence()
system.load_system()  # Load pre-computed embeddings

results = system.find_similar_accidents(
    "Crew welding near flammable materials",
    top_k=5
)

for result in results:
    print(f"Similarity: {result['similarity']}")
    print(f"Category: {result['event_category']}")
    print(f"Narrative: {result['past_accident'][:100]}...")
```

---

## 🧪 Example Queries

**Query:** *"Installing steel roof trusses at height with cranes"*

**Top Result:**
```
Similarity: 0.667
Category: Falls
Accident Type: 5
Severity: 2
Keywords: BRACING, COLLAPSE, FRACTURE, INSTALLING, TRUSS
Narrative: At 1:30 p.m. on October 7, 2019, a crew of five carpenters 
were installing a roof truss system when the trusses collapsed...
```

**Query:** *"Electrician working on live panel"*

**Top Result:**

```
Similarity: 0.493
Category: Electrical
Accident Type: 13
Severity: 1
Keywords: ELECTRIC SHOCK, ELECTROCUTED, INSTALLING, POWER LINE
Narrative: At 1:50 p.m. on March 18, 2020, Employee #1 was installing 
gutters when he made contact with overhead power lines...
```

---

## 🔬 Methodology Deep Dive

### 1. **Data Preparation**
- Filtered to construction incidents only
- Removed duplicates and handled missing values
- Merged rare event types into broader categories
- Final dataset: ~10,000 incidents

### 2. **Text Processing**
- Minimal preprocessing to preserve context
- Lowercasing and punctuation removal only
- Combined abstract and keywords for richer semantics

### 3. **Embedding Generation**
- Model: `all-mpnet-base-v2` (Microsoft)
- Output: 768-dimensional vectors
- Captures semantic relationships, synonyms, and context

### 4. **Classification**
- Algorithm: XGBoost
- Features: Text embeddings + weather data
- Strategy: Stratified train/test split (80/20)
- Class weights for imbalance handling

### 5. **Evidence Retrieval**
- Cosine similarity between query and incident embeddings
- Returns top-K most similar incidents
- Evaluated with standard IR metrics

---

## 📈 Future Improvements

- [ ] **Multilingual support** for non-English work sites
- [ ] **Photo-based hazard detection** using computer vision
- [ ] **Real-time IoT integration** for dynamic risk assessment
- [ ] **Hybrid recommendation system** (rule-based + Apriori algorithm)
- [ ] **Active learning** to improve with user feedback
- [ ] **API endpoint** for easy integration with existing safety software

---

## 👥 Team

This project was developed by students at the **University of Doha For Science and Technology (CCIT)**:

| Name | 
|------|
| Mohammad Faiz Jabir |
| Abein Gijo |
| Mohd Soad Bin Bashar 
| Keshob Kumar Sarkar|

---

## 📚 References

1. Kim, H., & Lee, D. (2021). Predictive safety analytics using NLP on unstructured incident reports. *Safety Science*.
2. U.S. Department of Labor, OSHA. (2023). Severe Injury Reports.
3. Devlin, J., et al. (2019). BERT: Pre-training of Deep Bidirectional Transformers. *NAACL-HLT*.
4. Chen, T., & Guestrin, C. (2016). XGBoost: A scalable tree boosting system. *ACM SIGKDD*.
5. Vaswani, A., et al. (2017). Attention is All You Need. *NeurIPS*.
6. Ou, Z., et al. (2025). Building safer sites: A large-scale multi-level dataset for Construction Safety Benchmark. *CIKM*.

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

---

*Built with for safer construction sites everywhere*
