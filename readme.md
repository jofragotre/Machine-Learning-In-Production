This repo serves as notes / code tracking for the Machine Learning in Production coursera course by DeepLearning.AI

# Machine learning project lifecycle

## Scoping

  - Define project

## Data

  - Define data and establish baseline
  - Label and organize data

## Modeling

  - Select and train model
  - Perform error analysis

Iterate over Data and Modeling until ready to deploy.

## Deployment

  - Deploy in production
  - Monitor and mantain the system

Problems with live data performance or concept/data drift might affect performance and revising the data or modeling aproach might be necessary.

# Deployment

## Problems

  - Data drift - gradual or sudden changes to data.

  - Concept drift - desired mapping changes (i.e. fraud detection after covid-19).

  - Software engineering issues:

      - Realtime or batched
      - Cloud vs Edge/Browser
      - Compute resources for deployment
      - Latency, Throughput
      - Logging
      - Security and Privacy

First deployment -\> maintenance\!

## Deployment patterns

  - New product/capability
  - Automate/assist manual task
  - Replace previous system:
      - Gradual Ramp-up with monitoring
      - Rollback

**Shadow mode** : use in paralell with human to validate
**Canary deployment** : ram up traffic slowly
**Blue green deployment** : *router* from old prediction service to new one (easy to roll back)

### Degrees of automation:

Spectrum:

  - Human only
  - Shadow mode
  - A.I. assistance
  - Partial automation : small confidence -\> human
  - Full automation

## Monitoring

**Dashboard**
e.g. Server load, fraction of non-null outputs, fraction of missing input values.
Brainstorm what could go wrong and implement statistics/metrics to detect the specific problems. Better have too many metrics.

**Example metrics**

  - Software metrics: memory cpu, latency, etc
  - Input metrics: distribution metrics (e.g., average image brightness, avg. text length, distribution of categorical features)
  - Output metrics: output changes over time (e.g., distribution of predicted classes), user satisfaction (if measurable)

**Deployment is iterative as well:**
Deploy/Monitor -\> Get Traffic -\> Analyse performance -\> repeat

  - Set thresholds for alarms\!
  - Adapt metrics and threshold over time.

Manual or Automatic retraining\!

## Pipeline Monitoring

Module changes may affect future modules: cascading effect (example face detection -\> face recognition).

  - Monitor metrics for each component/module\!

# Modeling

AI = code + data

Challenges:

  - Do well on train set
  - Do well on dev/test set
  - Do well on business metrics/project goals

**Why low error is not enough.**

  - Test set might not be representative of final production: X part of data might be more important to user that other part. E.g. informational queries vs navigational queries on search engines: *Disproportional importance* -\> Use weighted metrics or targeted error analysis.

  - Bias. Example ethnicity in FR. Evaluate performance on key data slices.

  - Rare classes. e.g. (99% acc -\> test set is 99% one class). Use appropriate metrics like Precision, Recall, F1-score, or AUC PR.

### Establish a baseline

Example: Human Level performance on unstructured data: image, audio, text.

  - Human level performance (HLP) - gives estimate of Bayes Optimal Error.
  - Literature search of SOTA/Open Source
  - Quick/Dirty implementation
  - Older system performance

### Tips for getting started

Getting started:

  - Literature search to see what's possible.
  - Open source if available.
  - Usually: Reasonable algorithm with good data \>\> amazing algorithm with not so good data (Data-centric AI philosophy).

Deployment constraints: if baseline already established and the goal is to build and deply. If not, establish a baseline first even if it is above constraints (e.g. too slow), then optimize.

**Usefull sanity checks**

  - Try to overfit on small training set. Find bugs much easier. Check if model can learn *something*.

## Error analysis

Understand failed prediction examples to help where model is failing. Specific sample categories like noisy images, low contrast, specific user types, etc. using **tags**.

Usefull insights:

  - What fraction of total error has a tag.
  - What fraction of examples with a specific tag are misclassified.
  - Create an error analysis spreadsheet/tool.

### Where to focus improvements:

  - **Ceiling Analysis:** How much room for improvement per category (potential impact on overall error)?
  - **Frequency:** How often does this category appear in the data?
  - **Ease:** How easy is it to make improvements for this category (e.g., collecting more data vs. fundamental research)?
  - **Importance:** How important is this category to the business/product goal (even if rare)?

Use the above factors for prioritization.

Common improvement strategies derived from error analysis:

  - Collect more data for specific categories.
  - Use data augmentation targeted at specific weaknesses.
  - Improve data quality/label accuracy for difficult examples.
  - Feature engineering.
  - Adjust model architecture or hyperparameters.

### Skewed datasets

Raw accuracy is not a useful metric on skewed data.

**Confusion matrix**

|           | Predicted Negative | Predicted Positive |
| :-------- | :----------------- | :----------------- |
| **Actual Negative** | TN                 | FP                 |
| **Actual Positive** | FN                 | TP                 |

Precision = $\\frac{TP}{TP+FP}$ (Of those predicted positive, how many actually are?)
Recall = $\\frac{TP}{TP+FN}$ (Of all actual positives, how many were found?)

F1 Score = $2 \\times \\frac{Precision \\times Recall}{Precision + Recall}$ (Harmonic mean, balances Precision and Recall)

Choose metric based on business need (is False Positive or False Negative worse?). Also consider Precision-Recall Curve (PR Curve) and Area Under PR Curve (AUC PR).

### Performance auditing

**Framework:**

  - Brainstorm potential issues and failure modes (biases, fairness concerns, performance on rare classes, robustness to specific inputs, security vulnerabilities).
  - Establish metrics to assess performance on each of these issues (e.g., accuracy per subgroup, checks for specific adversarial inputs).
  - Get input from business/product owner on critical slices or fairness requirements.
  - Perform checks periodically, not just at launch.

## Data iteration

**Augmentation**
Create **realistic** examples that the algorithm currently performs poorly on, but are not so hard that it is unreasonable for the algorithm to eventually do well. Target specific errors found during error analysis. Avoid creating unrealistic artifacts.

**Adding data:**
If Unstructured data (images, audio, text):

  - Using a low bias model (large model).
  - The mapping X-\>Y is relatively clear (humans can do it well).
    Then adding more data rarely hurts performance and often helps significantly.

Structured data:
Adding more examples *might* help, but adding relevant *features* (feature engineering) is often more impactful.

### Experiment tracking - best practices

Track:

  - Algorithm/code version (e.g., Git commit hash).
  - Dataset used (version, source, preprocessing steps).
  - Hyperparameters.
  - Environment details (libraries, hardware).
  - Results (metrics, evaluation plots, example outputs, qualitative analysis).
  - Notes/Observations.

Use tools like MLflow, Weights & Biases, Comet ML, or simple spreadsheets/logs.

## Data:

**Data-Centric AI Development:**
Shift focus from solely optimizing code/model to systematically improving data quality. Often, improving data yields better results than improving the model architecture, especially once a reasonable model is chosen.

**Data Definition and Requirements:**

  - **Sources:** Identify where data comes from (logs, user uploads, databases, third-party APIs, sensors).
  - **Features:** What information is available? What might be needed?
  - **Labels:** What is the target variable (y)? How is it defined? Are labels available or do they need to be created?
  - **Volume:** How much data is needed? How much is available?
  - **Format:** Understand the structure (structured tables, images, text, etc.).

**Data Collection/Acquisition:**

  - Strategies for gathering data (logging, purchasing, scraping, generating).
  - Considerations for data privacy and regulations (GDPR, CCPA, etc.) *during* collection.
  - Potential biases introduced during collection (e.g., sampling bias).

**Data Cleaning and Preprocessing:**

  - **Handling Missing Values:** Strategies like deletion (rows/columns), mean/median/mode imputation, or more complex model-based imputation. Assess impact of missingness.
  - **Outlier Detection/Handling:** Identify extreme values (e.g., using standard deviations, IQR). Decide whether to cap, remove, or transform them based on domain knowledge.
  - **Data Formatting:** Ensure consistency (e.g., date formats, categorical encodings).
  - **Normalization/Standardization:** Scaling numerical features (e.g., min-max scaling, z-score standardization) often required by models.
  - **Feature Engineering:** Creating new features from existing ones based on domain knowledge (can also be part of modeling).

**Labeling Data (for Supervised Learning):**

  - **Labeling Instructions:** Create clear, unambiguous guidelines for human labelers. Include edge cases.
  - **Labeling Workforce:** Decide between in-house experts, crowdsourcing platforms (e.g., Amazon Mechanical Turk, Appen), or specialized labeling services.
  - **Quality Control:** Implement mechanisms to ensure label consistency and accuracy:
      - **Consensus:** Multiple labelers label the same item; use agreement to measure quality or determine final label.
      - **Gold Standard Sets:** Use examples with known ground truth to test labelers.
      - **Auditing:** Review labeled examples periodically.
  - **Iterative Labeling:** Refine instructions and processes based on labeler feedback and model performance.

**Data Splitting:**

  - **Train/Dev(Validation)/Test Sets:**
      - **Train:** Used to train the model parameters.
      - **Dev (Validation):** Used for hyperparameter tuning, model selection, and error analysis during development. *Crucial: Must come from the same distribution as the Test set and reflect production data.*
      - **Test:** Used for final, unbiased evaluation of the chosen model's performance *after* all development is complete.
  - **Distribution:** Ensure Dev and Test sets reflect the actual distribution of data the model will see in production. If the production distribution is expected to change, the Dev/Test sets might need to reflect that future state.
  - **Size:** Allocate enough data to Dev/Test sets for statistically meaningful evaluation, especially for rare classes or slices.
  - **Leakage:** Prevent data from the Dev/Test sets from influencing the training process (directly or indirectly via feature engineering choices based on Dev/Test performance).
  - **Splitting Methods:** Random split (common), time-based split (for temporal data), group-based split (e.g., ensure all data from one user is in the same split).

**Data Storage and Versioning:**

  - Store data reliably (e.g., cloud storage like S3/GCS/Azure Blob, databases, data lakes).
  - **Versioning:** Track changes to datasets over time. Tools like DVC (Data Version Control) can help manage large datasets alongside code versions. Essential for reproducibility.

**Data Provenance/Lineage:**

  - Track where data originated and how it was transformed (preprocessing steps, feature engineering, labeling process).
  - Important for debugging, auditing, understanding model behavior, and ensuring reproducibility.

**(End of new/completed notes)**

## Scoping

**Process:**
- Identify business problem, not A.I. problem;
- Brainstorm A.I. solutions;
- Assess feasability and value of possible solutions;
- Determine milestones;
- Budget for resources;

**Feasability:**
- Use external benchmark.