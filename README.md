Markdown
# Global Financial Data Quality Gate & Pre-Consolidation Framework

## Executive Summary
In multinational financial environments, data governance failures during monthly group consolidations frequently lead to closing bottlenecks, manually adjusted journals, and broken audit trails. 

This repository demonstrates an automated **Data Quality Gate Engine** built in SQL Server. It simulates an enterprise ETL pipeline that intercepts disparate ledger submissions from three global subsidiaries (Denmark, Kenya, United States), enforces master data normalization rules on conflicting supplier/intercompany profiles, and executes a trial balance validation sweep prior to group ledger entry.

---

## The Data Governance Problem Simulated
Three separate regional operations submitted ledger journals involving a single global software vendor (**Goshen Group**). The raw submissions suffered from fundamental data integrity issues:
1. **Fragmented Master Data:** - Denmark ERP tracked the entity as `Goshen Ltd`.
   - Kenya ERP tracked the entity as `Goshen Systems`.
   - United States ERP tracked the entity as `Goshen International`.
2. **Internal Accounting Failure:** The United States entity submitted an unbalanced ledger journal (Debits: $6,200 vs. Credits: $6,500), violating foundational dual-entry controls.
3. **Data Truncation Error:** The Kenya subsidiary's raw system extract contained a missing trailing zero on its credit entry ($950,000 vs. $9,500,000 true debit balance).

---

## Solution Architecture & SQL Engineering

### 1. Unified Staging & Master Data Mapping Layer
The data pipeline implements a `LEFT JOIN` strategy optimized with string cleaning (`TRIM`, `LOWER`) and fallback logic (`COALESCE`) against a centralized Group Master Mapping lookup table. This dynamically unifies all local variants under a single, auditable record: **Goshen Group Corporate**.

### 2. Automated Type-Casting & Data Repairs
To safeguard against formatting inconsistencies common to flat-file formats, string cleaning functions (`REPLACE`) eliminate spaces and thousands separators. A conditional `CASE` statement implements a programmatic audit patch to seamlessly repair Kenya’s truncated credit amount back to true numeric equilibrium.

### 3. Pre-Consolidation Guardrails (The Audit Gate)
A programmatic system sweep assesses absolute variances between total debits and credits across all reporting sectors before accounting information interacts with the consolidation core.

---

## Production Impact & Validation Results
Upon pipeline execution, the staging framework cleanly achieved the following outcomes:
- **Master Data Resolution:** Converted 100% of fragmented naming anomalies across three source databases into the golden group standard record.
- **Typo Interception:** Automated type casting and custom case parameters brought Kenya's mismatched multi-currency record to a perfect zero balance.
- **Exception Quarantine:** Instantly isolated the **US Subsidiary** with a highlighted **-300.00 net variance imbalance**, preventing broken data structures from corrupting consolidated group financial statements.

**Technology Stack:** SQL Server (T-SQL), Microsoft Excel / Flat-File Data Extracts, VS Code.
