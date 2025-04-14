# Sentiment Analysis for Best Firms to Work For (SAL) - README

## Overview

**Project Name**: Sentiment Analysis for Best Firms to Work For (SAL)  
**Purpose**: Analyze employee feedback from the "Best Firms to Work For" survey to provide insights into sentiment, strengths, weaknesses, and category-specific feedback for participating companies.  
**Objective**: Deliver actionable insights through a pipeline that classifies sentiment, summarizes responses, identifies key themes, and presents results via a user-friendly front-end.  
**Scope**: Processes survey responses using fine-tuned language models to categorize feedback, generate summaries, and highlight strengths/weaknesses for each collector, defined as a unique combination of a company and the survey year (e.g., Company X, 2023).

## System Architecture

SAL is a Django-based application integrated with fine-tuned GPT-4o-mini models from OpenAI. Key components:

- **Data Sources**: Historical survey responses plus synthetic data for robust analysis.  
- **Backend**: Django scripts (`/workspace/insights-net/backend/bftwf/management/commands`) for processing, inference, and database management.  
- **Models**: Fine-tuned models for sentiment classification, category summarization, overall summarization, and strengths/weaknesses identification.  
- **Front-End**: Interface displaying sentiment breakdowns, summaries, and key themes.  
- **Storage**: Datasets stored in the codebase under `/workspace/insights-net/backend/bftwf/datasets`.

## Pipeline Structure

The SAL pipeline is divided into two main phases: **Training** and **Inference**. Below are the critical steps for each phase.

### Training Phase

The training phase focuses on preparing data and models to ensure accurate analysis.

#### 1. Dataset Creation
- **Goal**: Build a comprehensive dataset for model training.  
- **Details**:  
  - Collected historical "Best Firms to Work For" responses, labeled as Positive, Negative, or Outliers using ground-truth summaries from a specific company.  
  - Generated synthetic responses via GPT to increase dataset size and balance across categories.  
  - Created datasets for:  
    - Sentiment classification (Positive, Negative, Outliers).  
    - Strengths and weaknesses identification.  
    - Six survey categories: Culture, Compensation, Benefits, Recruiting and Retention, Professional Development, Performance Management and Recognition.  
    - Overall summaries aggregating feedback.  
  - Stored datasets in `/workspace/insights-net/backend/bftwf/datasets`.

#### 2. Model Fine-Tuning
- **Goal**: Develop specialized models for analysis tasks.  
- **Details**:  
  - Fine-tuned a GPT-4o-mini model for sentiment classification to label responses as Positive, Negative, or Outliers.  
  - Created individual models for summarizing each of the six survey categories, tailored to their unique themes.  
  - Fine-tuned a model for generating overall summaries that combine category insights.  
  - Developed separate models for identifying top strengths and weaknesses.  
  - Configured secure API keys and model IDs for reliable access.

### Inference Phase

The inference phase applies the trained models to analyze new survey data and present results.

#### 3. Sentiment Classification
- **Goal**: Label responses by sentiment.  
- **Details**:  
  - Processed responses for each collector (company and survey year), categorizing them as Positive, Negative, or Outliers.  
  - Skipped previously classified responses to optimize performance.  
  - Stored results in the database for use in later steps.  
  - Tracked progress for efficiency.

#### 4. Overlap Detection
- **Goal**: Identify responses spanning multiple categories.  
- **Details**:  
  - Flagged responses mentioning multiple categories using predefined keywords (e.g., “team spirit” for Culture, “pay raise” for Compensation).  
  - Recorded overlaps in the database to link responses to relevant categories.  
  - Reported overlap counts to highlight interconnected themes.  
  - Monitored progress to manage large datasets.

#### 5. Category Summarization
- **Goal**: Summarize feedback for each category.  
- **Details**:  
  - Grouped responses by category, incorporating overlaps from other categories.  
  - Generated summaries with length based on response count:  
    - 2-3 sentences for ≤5 responses.  
    - 4-5 sentences for ≤20 responses.  
    - 6-8 sentences for >20 responses.  
  - Used category-specific models for tailored summaries.  
  - Stored summaries and sentiment counts (positive/negative) in the database.  
  - Tracked progress to ensure completion.

#### 6. Overall Summarization
- **Goal**: Provide a comprehensive feedback summary.  
- **Details**:  
  - Combined category summaries and sentiment counts for each collector.  
  - Generated overall summaries with length based on response count:  
    - 2-3 sentences for ≤10 responses.  
    - 4-6 sentences for ≤50 responses.  
    - 7-10 sentences for >50 responses.  
  - Saved results in the database for front-end display.  
  - Monitored progress to confirm processing.

#### 7. Strengths and Weaknesses Identification
- **Goal**: Highlight top three positive and negative themes.  
- **Details**:  
  - Analyzed positive responses for strengths and negative responses for weaknesses.  
  - Produced three bullet points per category using dedicated models, focusing on frequent themes.  
  - Stored results in the database, with fallback messages (e.g., “No positive feedback found”) if no data existed.  
  - Tracked progress for efficient processing.

#### 8. Front-End Display
- **Goal**: Present results clearly to users.  
- **Details**:  
  - Built an interface displaying:  
    - Sentiment distributions (positive vs. negative counts).  
    - Category summaries with key themes and overlap insights.  
    - Overall summary for a high-level view.  
    - Top three strengths and weaknesses in bullet-point format.  
  - Supported filtering by collector ID to focus on specific company-year combinations.  
  - Pulled data directly from the database to reflect the latest results.

## Technical Details

- **Platform**: Django backend located at `/workspace/insights-net/backend/bftwf/management/commands`.  
- **Dependencies**: Python libraries for text processing, progress tracking, and API integration; Django models for data management.  
- **Run Command**:  
  ```bash
  python manage.py collector_pipeline [--collector-id=<id>]
  ```
- **Progress Tracking**: Visual progress bars for each inference step to monitor completion.  
- **Error Handling**: Robust handling of API failures and missing data to ensure pipeline stability.

## Dataset

- **Historical Data**: Past "Best Firms to Work For" survey responses, labeled using ground-truth summaries from a specific company.  
- **Synthetic Data**: GPT-generated entries to enhance dataset size and balance.  
- **Categories**: Covers sentiment classification, strengths/weaknesses, six survey categories, and overall summaries.  
- **Storage**: Stored in `/workspace/insights-net/backend/bftwf/datasets`.

