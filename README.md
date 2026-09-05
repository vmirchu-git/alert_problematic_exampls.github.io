<p align="center">
  <img src="assets/accuracy_causal_chart.png" alt="Classification accuracy before, during, and after the alert gap" width="100%">
</p>

<p align="right"><a href="README.ru.md">Русская версия →</a></p>

# Automatic Alert for Incorrect Auto-Labeling in Chatbot Intents

![Python](https://img.shields.io/badge/Python-pandas-14131a?style=flat-square&labelColor=14131a&color=7a1f2b)
![SQL](https://img.shields.io/badge/SQL-ETL-14131a?style=flat-square&labelColor=14131a&color=7a1f2b)
![BI](https://img.shields.io/badge/BI-reporting-14131a?style=flat-square&labelColor=14131a&color=7a1f2b)
![status](https://img.shields.io/badge/status-production%20since%202024-14131a?style=flat-square&labelColor=14131a&color=c9a227)

> **Note on source code.** This repository documents methodology, design decisions, and results only. The implementation is proprietary to the employer where this project was built and is not published here.

A simple, explainable ranking algorithm that surfaces the 15 chatbot intents with the most critical auto-labeling errors every two weeks — and, unusually for a project like this, comes with direct before/after evidence that it actually mattered.

## The evidence, first

The chart above isn't a mocked-up illustration — it's what actually happened when the team stopped using this alert for about seven months and later reinstated it. While the alert was active, classification accuracy climbed steadily (≈91.5% → ≈93.5%). Once it stopped being used, accuracy drifted down over the following months to as low as ≈87.6% — a relative decline of roughly 6%. Within weeks of reinstating the alert, accuracy climbed back above 98% — a relative recovery of over 12% from the low point. This wasn't a designed causal experiment with a control group, but the timing lines up closely enough to treat it as strong, if informal, evidence that the alert was doing real work, not just producing reports nobody needed.

**Downstream effect on the classifier.** The classifier model itself is trained on labeled examples that pass through this full-text layer, so cleaner labeling upstream should, in principle, also lift the model's own accuracy over time. That's what shows up in practice too — accuracy climbed steadily from ≈68.6% to a peak of ≈75.8% over several months. (The metric later declined for an unrelated reason — model overfitting — so that portion is left out here to keep the two effects from being conflated.)

<p align="center">
  <img src="assets/model_accuracy_chart.png" alt="Downstream classifier model accuracy trend" width="100%">
</p>

## Problem

Auto-labeling errors in chatbot intents were frequent enough to noticeably reduce intent accuracy, introduce noise into training datasets, and erode trust in automation results generally. Manual detection of these errors was slow and incomplete, which meant systematic quality improvement kept stalling.

## Approach

**Data pipeline.** A SQL script extracts labeling error data; Python aggregates, filters, and ranks it by frequency and significance. Filtering rules were deliberately simple and explainable: at least 2 errors on the same gold example, exclusion of technical or obsolete categories, and priority given to errors that actually affect model performance rather than edge cases.

**Alert system.** A report is generated automatically every two weeks, surfacing the top 15 problem intents ranked by correction priority, and feeding into the team's BI tooling for visual review. The illustrative example below shows the shape of that output — accuracy per example color-coded by severity, paired with a suggested corrective action:

<p align="center">
  <img src="assets/priority_table.svg" alt="Illustrative example of the ranked priority table" width="100%">
</p>

**Why accuracy at this stage matters so much.** One of the inputs this alert monitors is the full-text matching layer — an early, low-cost stage in intent classification that's meant to catch messages cheaply before heavier, more expensive methods are needed downstream. Because everything after it depends on getting these matches right, even small labeling errors here compound significantly by the time they reach later stages.

**Cross-team integration.** Filtering parameters were agreed on jointly with each chatbot product team rather than set unilaterally, then the solution was scaled across all chatbot directions and handed over to the NLP team for ongoing ownership.

## Results

- More than a 20× reduction in errors per labeled example.
- In stable production use for 1.5+ years (since at least 2024) — long enough to observe the natural before/after comparison above.
- Became a standard part of the broader model quality control process, not a one-off report.

## Business impact

- Improved the quality of training data by catching labeling errors before they compounded downstream.
- Reduced the time needed to find and fix critical errors, replacing an incomplete manual process.
- Increased the stability of the underlying business logic and query understanding — with the accuracy chart above as direct evidence of what happens when that stability is left unmonitored.
- Delivered a scalable, fully explainable algorithm — simple enough that everyone involved understood exactly why an intent was flagged, with a measurable link to model accuracy.

## Tech stack

Python · pandas · SQL · ETL · BI reporting tools

---

<sub>Individual project completed as part of a Data Analytics role. Described here for portfolio purposes; production code is not publicly available.</sub>
