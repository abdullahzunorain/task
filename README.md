
# Phishing Email Detection using ML + XAI(LLM)

## Project Overview

This project implements a **hybrid AI system for phishing email detection**, combining a classical **machine learning (ML) model** with a controlled **large language model (LLM) escalation** for borderline or high-impact cases. The goal is to classify incoming emails into `Safe`, `Suspicious`, or `Phishing`, provide interpretable signals, and optionally generate non-technical explanations and guidance for end-users.

Key objectives:

* **Automated detection** of phishing attempts with a risk score (0–100).
* **Explainable AI**: Identify top contributing TF-IDF features and additional signals.
* **Controlled LLM escalation**: Leverage `llama-3.1-8b-instant` for borderline or high-impact emails without over-relying on generative AI.
* **Practical usability**: Provide actionable guidance for non-technical users.

This design ensures that **strong ML signals drive primary classification**, while the LLM is used strategically to handle uncertainty, complex context, or high-value targets.

---

## System Architecture

The system follows a **three-layered architecture**:

1. **Machine Learning Layer**

   * **Model**: `RandomForestClassifier` trained on a TF-IDF representation of email content.
   * **Preprocessing**: Text normalization (lowercasing, whitespace removal, regex-based filtering) to match training conditions.
   * **Explainability**:

     * Extracts **top contributing TF-IDF features** per email.
     * Flags additional signals, such as urgency, links, credential-related terms, and high-impact keywords (finance, HR, executive roles).

2. **LLM Escalation Layer**

   * Triggered for **borderline risk scores (45–70)** or **high-impact contexts**.
   * **LLM**: Groq-hosted `llama-3.1-8b-instant`.
   * Output is strictly **JSON-structured**, including:

     * LLM classification (`Safe`, `Suspicious`, `Phishing`)
     * Confidence score
     * Alignment with ML predictions
     * Bullet-pointed reasons and user guidance
     * Non-technical explanation (concise, <60 words)
   * Includes **robust parsing and fallback mechanisms** in case of API errors or unexpected LLM output.

3. **Decision & Guidance Layer**

   * Combines ML and LLM outputs using **controlled logic**:

     * High ML confidence (>70) overrides LLM.
     * Low ML confidence (<45) generally stays Safe unless LLM strongly disagrees in high-impact contexts.
     * Borderline/Suspicious emails may adopt LLM refinement unless alignment indicates strong disagreement.
   * Generates **end-user explanation and guidance** automatically, reducing cognitive load and improving compliance.

---

## Implementation Details

### Core Utilities

* **Text cleaning**: Regex-based filtering for alphanumeric, whitespace, and select symbols (`:/.`).
* **Probability → Risk Score**: Converts ML probability output to an integer scale (0–100).
* **Risk Labeling**: Threshold-based labeling (`Safe` <45, `Suspicious` 45–70, `Phishing` ≥70).

### ML Explainability

* Calculates **feature contributions** by multiplying TF-IDF values by Random Forest feature importances.
* Detects **additional signals** (urgency terms, links, credentials, high-impact keywords).
* Provides **top 5–10 features** per email for auditability.

### LLM Integration

* **Prompt Engineering**: System prompt instructs the LLM to consider ML results, follow triage rules, and output JSON only.
* **Escalation Control**: Only borderline/high-impact emails are passed to LLM to minimize API usage.
* **Fallbacks**: Default ML-only guidance is used if LLM fails or returns invalid JSON.

### Evaluation

* Sampled **50 test emails** from the dataset to evaluate combined system.
* Metrics obtained:

  * **Accuracy**: 0.68
  * **Precision**: 0.667
  * **Recall**: 0.667
  * **F1-score**: 0.667
* LLM was used in **80% of samples**, showing the system's reliance on ML to reduce unnecessary API calls.

---

## Justification of Approach

1. **Hybrid Design**: Combining ML and LLM allows the system to retain **predictable, explainable behavior** via ML while leveraging **contextual reasoning** for borderline/high-value cases.
2. **Explainability**: Extracting top TF-IDF features and additional signals ensures that decisions can be **audited and interpreted**.
3. **Controlled Escalation**: Avoids over-reliance on generative models, reducing **false positives** and **operational cost**.
4. **End-User Focused**: Structured explanations and guidance help reduce human error in handling phishing attempts.

---

## Failure Modes

1. **ML Limitations**

   * Random Forest may misclassify **rare or novel phishing patterns**.
   * TF-IDF cannot capture **semantic context or social engineering nuances**.

2. **LLM Risks**

   * LLM may **misinterpret context** or provide misleading explanations.
   * JSON parsing errors or API downtime may fallback to ML-only results.

3. **High-Impact Sensitivity**

   * High-value targets may still be **missed if neither ML nor LLM detects context**, requiring continual keyword updates and retraining.

4. **Data Bias**

   * Training dataset may overrepresent certain phishing patterns, limiting generalization to new campaigns.

---

## Future Directions

1. **Semantic Embeddings**

   * Replace or augment TF-IDF with **transformer-based embeddings** (e.g., BERT, SBERT) for better detection of context and paraphrasing.

2. **Active Learning**

   * Integrate a **human-in-the-loop feedback mechanism** to continuously improve the model on borderline cases.

3. **Threat Intelligence Integration**

   * Leverage **external phishing feeds and URL reputation** for real-time detection of new threats.

4. **User Personalization**

   * Adjust scoring thresholds and guidance based on **user role or department** (e.g., Finance vs. Engineering).

5. **Monitoring & Logging**

   * Track **LLM escalations, false positives/negatives, and user feedback** to continuously refine model and system behavior.

---

## Conclusion

This project demonstrates a **robust, explainable, and user-centric phishing email detection system**. By combining classical ML models with a **strategically used LLM**, it balances **accuracy, interpretability, and operational efficiency**. The system not only flags high-risk emails but also provides actionable guidance, making it suitable for deployment in enterprise environments where both security and usability are critical.

---

