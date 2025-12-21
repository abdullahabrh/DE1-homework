# Sentiment Analysis Pipeline for Indonesian Tourism Reviews  
**Data Engineering 1 – Assignment 3**

---

## 1. The Problem
This project builds a small, end-to-end data pipeline to analyze customer sentiment and extract structured feedback from tourism reviews written in Indonesian. The motivation is practical: tourism platforms and local authorities often receive large volumes of unstructured, multilingual feedback, which is difficult to summarize and act upon.

**Our goals were to:**
- **Translate** Indonesian-language reviews into English.
- **Sentiment** using AWS Comprehend services and compare results with existing labels.
- **Quantify** reviews by generating a numeric 1–5 score.
- **Extract** a short “Chief Complaint” to highlight the main issue, if any.

> **Note:**  
> Although the original assignment suggested scraping TripAdvisor directly, scraping was unreliable and time-consuming in practice. To stay focused on data engineering and AWS services, we used a pre-collected dataset instead.

---

## 2. The Dataset
We used the **Dataset for Sentiment Analysis of Tourist Attraction Reviews in Indonesian Language**.

- **Source:** TripAdvisor (Indonesian site).  
- **Size:** 200 reviews.  
- **Original Labels:** 100 Positive, 100 Negative (human-annotated).  
- **Columns:** `No`, `Review`, `Sentiment`.

The dataset is relatively small and balanced, which makes it suitable for experimentation, but it also limits the generalizability of the results.

---

## 3. Methodology & AWS Services
We implemented a Python pipeline using `boto3` and three AWS serverless services.

### A. AWS Translate
All reviews were translated from Indonesian to English using **AWS Translate**. While translations were generally understandable, the quality was inconsistent. Informal language, local expressions, and place-specific terms were sometimes translated awkwardly, which likely affected downstream sentiment analysis.

### B. AWS Comprehend
We applied **AWS Comprehend** to the translated text to detect sentiment. Unlike the dataset’s binary labels, Comprehend outputs four categories: **Positive, Negative, Neutral, Mixed**. This provided a broader and often more cautious interpretation of sentiment.

### C. AWS Bedrock (Amazon Nova Micro)
Using **Amazon Bedrock** with the `nova-micro-v1` model, we performed two additional tasks:
1. **Review score generation:** Assigning a numeric score from 1 to 5.
2. **Chief complaint extraction:** Identifying the single most important complaint, or explicitly stating when no clear complaint exists.

These steps move beyond classification and aim to produce outputs closer to what a business might actually use.

---

## 4. Findings & Results

### Sentiment Distribution
AWS Comprehend classified a substantial number of reviews as **Neutral** (65) or **Mixed** (24), even when the original label was Positive or Negative. This suggests that many reviews contain both praise and criticism, which binary labels fail to capture.

### Review Scores
Bedrock-generated scores had a mean of **2.39**. Reviews with small complaints were often pushed toward lower scores. This may be useful for quality control, but it also risks overstating negativity.

### Chief Complaint Extraction
For a positive city tour review, Bedrock correctly returned:
- **Result:** *“No complaint (the review is praise or contains no complaint).”*
We did this fr only one review as it got very long and time consuming. For the larger dataset this model might not be perfect.

---

## 5. Cost Estimation
Estimated AWS costs for processing 200 reviews are shown below:

| Service | Usage | Estimated Cost (USD) |
| :--- | :--- | :--- |
| **AWS Translate** | ~30,000 characters | $0.45 |
| **AWS Comprehend** | Sentiment detection (200 reviews) | $0.20 |
| **AWS Bedrock** | ~50,000 input/output tokens (Nova Micro) | $0.05 |
| **Total** |  | **~$0.70** |

These costs are low at this scale, but they would grow quickly for larger datasets or more complex Bedrock prompts.

---

## 6. Conclusion
This project demonstrates that AWS managed services can be combined into a simple yet powerful sentiment analysis pipeline. **AWS Comprehend** offered a more nuanced view of sentiment than binary labels, while **Amazon Bedrock** was useful for generating human-readable summaries such as review scores and chief complaints.

That said, the pipeline is far from perfect. Translation quality clearly affected downstream results, the dataset was small, and some model outputs were overly generic or pessimistic. Overall, the project should be seen as a realistic prototype rather than a production-ready solution, highlighting both the potential and the limitations of using cloud-based AI services for multilingual text analysis.
