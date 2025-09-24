# Case Study: An Automated Pipeline for Analyzing Corporate ESG Reports

This document outlines a modular pipeline built to automate the extraction, classification, and analysis of claims from corporate Environmental, Social, and Governance (ESG) reports.

---
## 1. Problem Statement

Analyzing corporate ESG reports is essential for investors, regulators, and the public, but the process presents significant challenges:
* **Unstructured Data**: Most reports are published as lengthy, dense PDFs, making data extraction difficult and time-consuming.
* **Scalability Issues**: Manually reading and evaluating reports from multiple companies is not scalable, leading to inconsistent analysis.
* **Lack of Quantifiable Metrics**: It's difficult to objectively compare claims, which often vary in quality, specificity, and tone.

The objective of this project is to create an end-to-end pipeline that ingests these PDF reports and produces a **structured, quantitative dataset of ESG claims**. This structured data can then be used for various downstream tasks, such as tracking corporate commitments, comparing sustainability strategies, or, as an optional final step, calculating a metric like a "greenwashing risk score."

---
## 2. Assumptions
The solution is built upon two key assumptions:
* **Guidance from SASB Standards**: The analysis uses a predefined list of ESG topics (e.g., "Greenhouse Gas Emissions," "Data Security") that are aligned with established frameworks like the **Sustainability Accounting Standards Board (SASB)**. This ensures the extracted claims are relevant to materially important issues.
* **Quantifiable Claim Quality**: The quality and potential risk of a claim can be measured using a combination of linguistic features, such as the specificity of the language, the sentiment expressed, and the presence of non-committal "hedging" phrases.

---
## 3. Solution Overview
The solution is a multi-stage pipeline that processes raw PDF reports and outputs a structured DataFrame of analyzed claims.



1.  **Task 1: Ingestion and Claim Extraction**
2.  **Task 2: ESG Topic Classification**
3.  **Task 3: Feature Engineering (Sentiment & Specificity Analysis)**
4.  **Task 4: Optional Risk Modeling (Greenwashing Score)**

---
## 4. Step-Wise Implementation Details

### **Step 1: Ingestion and Claim Extraction**
This step converts unstructured PDF data into a clean, sentence-level dataset of potential ESG claims.

* **Implementation Details**:
    * The `fitz` library is used to parse PDFs, and `spaCy` is used for accurate sentence segmentation.
    * A rule-based function, `is_claim`, filters for sentences that are likely to be substantive claims. The rules check for minimum length, the presence of ESG-related keywords, and the inclusion of action verbs or numbers.

* **Tweakable Parameters**:
    * `MIN_PARAGRAPH_LENGTH`: The minimum character length for a text block to be processed (default: `50`).
    * `MIN_WORD_COUNT`: The minimum word count for a sentence to be considered a claim (default: `8`).
    * `ESG_KEYWORDS`: The list of keywords used to identify relevant claims.
    * `ACTION_VERBS`: The list of verbs used to identify actionable claims.

* **Other Alternatives**:
    * For scanned documents or complex layouts, an **Optical Character Recognition (OCR)** engine like Tesseract could be used for text extraction.
    * A **supervised binary classification model** (e.g., fine-tuned BERT) could be trained to identify claims with higher accuracy than the rule-based approach, provided a labeled dataset is available.

### **Step 2: ESG Topic Classification**
Each extracted claim is categorized into a standard ESG topic.

* **Implementation Details**:
    * A **zero-shot classification pipeline** from Hugging Face (`facebook/bart-large-mnli`) is used to categorize claims without requiring topic-specific training data.

* **Tweakable Parameters**:
    * `esg_topics`: The master list of candidate labels for classification.
    * `confidence_threshold`: The minimum model confidence score required to assign a topic (default: `0.7`). Claims below this are marked "Unclassified."

* **Other Alternatives**:
    * A **fine-tuned multi-class classification model** would likely yield higher accuracy if a large, topic-labeled dataset of claims were available for training.
    * Unsupervised methods like **BERTopic** could discover topics automatically, which would then need to be manually mapped to the standard ESG categories.

### **Step 3: Feature Engineering**
Quantitative features are generated for each claim to measure its quality and tone.

* **Implementation Details**:
    * **Sentiment Analysis**: The `analyze_sentiment` function uses `ProsusAI/finbert`, a model fine-tuned on corporate language, to assign a sentiment score.
    * **Specificity Score**: The `calculate_specificity_score` function uses a rule-based system to award points for the presence of numbers, years, percentages, and specific technical metrics (e.g., 'tco2e').
    * **Hedging Score**: The `calculate_hedging_score` function returns `1` if non-committal phrases (e.g., "aim to," "strive to") are present, and `0` otherwise.

* **Tweakable Parameters**:
    * `specific_metrics`: The list of keywords that indicate high specificity.
    * `HEDGING_PHRASES`: The list of phrases that indicate a lack of firm commitment.

* **Other Alternatives**:
    * A more advanced specificity score could be generated using **Named Entity Recognition (NER)** to identify and count a wider range of metrics, dates, and other quantifiable data.

### **Step 4: Optional Risk Modeling (Greenwashing Score)**
As an example application, the engineered features are combined into a single greenwashing risk score.

* **Implementation Details**:
    * The `calculate_greenwashing_risk` function computes a final score using a weighted average that heavily penalizes low specificity while also accounting for positive sentiment and hedging language.

* **Tweakable Parameters**:
    * `w_spec`: The weight for the inverse specificity score (default: `0.5`).
    * `w_sent`: The weight for the positive sentiment score (default: `0.25`).
    * `w_hedge`: The weight for the hedging score (default: `0.25`).

* **Other Alternatives**:
    * Instead of a fixed-weight formula, a **regression model** (e.g., Linear Regression, Gradient Boosting) could be trained on an expert-labeled dataset to learn the optimal feature weights and predict a more nuanced risk score.