
# Sentiment Analysis Project (SAL) Documentation

## Project Overview

**Project Name**: Sentiment Analysis for Best Firms to Work For (SAL)  
**Purpose**: To analyze employee feedback from the "Best Firms to Work For" survey, delivering actionable insights into employee sentiment, strengths, weaknesses, and category-specific feedback for participating companies.  
**Objective**: Provide a robust pipeline to classify sentiment, summarize responses, identify key themes, and present results through an intuitive front-end interface.  
**Scope**: Processes survey responses from employees, using fine-tuned language models to categorize feedback, generate summaries, and highlight strengths and weaknesses for each company (referred to as a "collector").  

## System Architecture

The SAL project is a Django-based application integrated with fine-tuned GPT-4o-mini models from OpenAI for natural language processing. Key components include:

- **Data Sources**: Historical survey responses supplemented with synthetic data to ensure a comprehensive dataset.
- **Backend**: Django scripts (`/workspace/insights-net/backend/bftwf/management/commands`) for data processing, model inference, and database management.
- **Models**: Fine-tuned models for sentiment classification, category-specific summarization, overall summarization, and strengths/weaknesses identification.
- **Front-End**: A user interface to display analysis results, including sentiment breakdowns, summaries, and key themes.
- **Storage**: Datasets and results stored in a shared file system (link provided in project details).

## Key Steps in the Process

The SAL project follows a structured pipeline to transform raw survey responses into meaningful insights. Below are the critical steps, explained without delving into function-level details.

### 1. Dataset Creation
- **Objective**: Build a robust dataset for training and analysis.
- **Process**:
  - Collected historical employee responses from previous years’ "Best Firms to Work For" surveys.
  - Used ground-truth key summaries from a specific company to label responses as Positive, Negative, or Outliers.
  - Generated additional synthetic responses using GPT models to increase dataset size and diversity, ensuring balanced representation across sentiment categories.
  - Created separate datasets for:
    - Sentiment classification (Positive, Negative, Outliers).
    - Strengths and weaknesses identification.
    - Category-specific summaries (Culture, Compensation, Benefits, Recruiting and Retention, Professional Development, Performance Management and Recognition).
    - Overall summaries aggregating all feedback.
  - Stored datasets in a shared file system for accessibility (link provided in project details).

### 2. Model Fine-Tuning
- **Objective**: Develop specialized models to handle different analysis tasks.
- **Process**:
  - Fine-tuned a GPT-4o-mini model for sentiment classification to accurately label responses as Positive, Negative, or Outliers.
  - Created individual models for summarizing feedback in each of the six survey categories, tailored to capture category-specific nuances.
  - Fine-tuned a model for generating overall summaries that combine insights across categories.
  - Developed separate models for identifying top strengths and weaknesses from feedback.
  - Configured secure API keys and model IDs for each task to ensure reliable access during processing.

### 3. Sentiment Classification
- **Objective**: Assign sentiment labels to each employee response.
- **Process**:
  - Processed all text responses for a given collector (company or survey instance).
  - Applied the fine-tuned classification model to categorize each response as Positive, Negative, or Outliers based on its content and context (e.g., the survey question).
  - Skipped responses already classified to avoid redundant processing.
  - Stored classification results in the database for use in subsequent steps.
  - Used progress tracking to monitor completion for each collector.

### 4. Overlap Detection
- **Objective**: Identify responses that touch on multiple survey categories.
- **Process**:
  - Analyzed each response to detect mentions of multiple categories (e.g., a response discussing both Culture and Benefits).
  - Used a predefined set of keywords for each category to flag overlaps (e.g., “team spirit” for Culture, “pay raise” for Compensation).
  - Recorded overlaps in the database, linking responses to relevant categories.
  - Counted and reported the number of multi-category responses to highlight interconnected themes.
  - Ensured progress was tracked to manage large datasets efficiently.

### 5. Category Summarization
- **Objective**: Generate concise summaries for each survey category.
- **Process**:
  - Grouped responses by category for each collector, including any overlapping responses identified in the previous step.
  - Combined positive, negative, and overlap data into a single input for summarization.
  - Used category-specific fine-tuned models to create summaries, with length adjusted based on response volume:
    - Short summaries (2-3 sentences) for up to 5 responses.
    - Medium summaries (4-5 sentences) for up to 20 responses.
    - Detailed summaries (6-8 sentences) for more than 20 responses.
  - Stored summaries in the database, along with counts of positive and negative responses for each category.
  - Tracked progress to ensure all categories were processed.

### 6. Overall Summarization
- **Objective**: Provide a comprehensive summary of feedback across all categories.
- **Process**:
  - Aggregated category summaries for each collector, including positive and negative response counts.
  - Used a fine-tuned model to generate an overall summary, tailored to the total number of responses:
    - Concise (2-3 sentences) for up to 10 responses.
    - Medium (4-6 sentences) for up to 50 responses.
    - Detailed (7-10 sentences) for more than 50 responses.
  - Ensured negative feedback was described without using the word “concerns” for consistency.
  - Saved the summary and sentiment counts in the database for front-end display.
  - Monitored progress to confirm completion.

### 7. Strengths and Weaknesses Identification
- **Objective**: Highlight the top three positive and negative themes in feedback.
- **Process**:
  - Collected all positive and negative responses for a collector, based on earlier classifications.
  - Used separate fine-tuned models to analyze positive responses for strengths and negative responses for weaknesses.
  - Generated exactly three bullet points for each, focusing on the most frequent themes (e.g., “Strong team culture” for strengths, “Limited career growth” for weaknesses).
  - Stored results in the database, with fallback messages (e.g., “No positive feedback found”) if no relevant responses existed.
  - Tracked progress to ensure efficient processing.

### 8. Front-End Display
- **Objective**: Present analysis results in a user-friendly format.
- **Process**:
  - Built a front-end interface to display:
    - Sentiment distributions (positive vs. negative counts) for each collector.
    - Category-specific summaries with key themes and overlap insights.
    - Overall summary providing a high-level view of employee feedback.
    - Top three strengths and weaknesses in bullet-point format.
  - Enabled filtering by collector ID or name for targeted analysis.
  - Ensured data was pulled from the database to reflect the latest pipeline results.

## Technical Implementation

- **Platform**: Django backend with scripts located in `/workspace/insights-net/backend/bftwf/management/commands`.
- **Dependencies**: Python libraries for progress tracking, text processing, and API integration, plus Django models for data management.
- **Execution**: Run via a Django management command:
  ```bash
  python manage.py collector_pipeline [--collector-id=<id>] [--collector-name=<name>]
  ```
  - Supports processing specific collectors or all collectors if no arguments are provided.
- **Progress Tracking**: Used visual progress bars to monitor each step, ensuring transparency during long-running tasks.
- **Error Handling**: Included mechanisms to handle API failures or missing data, logging errors without halting the pipeline.

## Dataset Details

- **Historical Data**: Responses from past "Best Firms to Work For" surveys, labeled using ground-truth summaries from a specific company.
- **Synthetic Data**: Additional entries created by GPT to enhance dataset size and balance.
- **Categories Covered**:
  - Sentiment (Positive, Negative, Outliers).
  - Strengths and weaknesses.
  - Six survey categories (Culture, Compensation, Benefits, Recruiting and Retention, Professional Development, Performance Management and Recognition).
  - Overall summaries.
- **Storage**: Datasets stored in a shared file system (link provided in project details).

## Security Considerations

- **API Keys**: Securely configured for each task (classification, summarization, strengths/weaknesses) to prevent unauthorized access.
- **Data Access**: Restricted to authorized users via the shared file system.
- **Database Integrity**: Ensured data consistency by updating or creating records only when necessary.

## Limitations and Future Improvements

- **Dataset Quality**: Synthetic data may not fully capture real-world nuances; validate against new survey data periodically.
- **Model Accuracy**: Models may need retraining as employee feedback evolves.
- **Processing Speed**: Large datasets could slow the pipeline; explore parallel processing for scalability.
- **Overlap Detection**: Keyword-based approach may miss complex themes; consider advanced NLP methods.
- **Front-End**: Add charts or interactive elements to enhance visualization of results.

## Maintenance Plan

- **Model Retraining**: Update models annually or with significant new data.
- **Code Updates**: Review quarterly for compatibility with Django and API changes.
- **Data Checks**: Audit datasets regularly for accuracy.
- **User Input**: Gather feedback to improve front-end usability and pipeline outputs.

