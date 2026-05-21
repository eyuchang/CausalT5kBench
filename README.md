# CausalT5K: Diagnosing and Informing Refusal for Trustworthy Causal Reasoning of Skepticism, Sycophancy, Detection-Correction, and Rung Collapse

```
How to test LLM for what they know and what they don't know?

--- CausalT5K Authors
```
<p align="center">
  ⬇️ <a href="https://github.com/genglongling/CausalT5kBench">Github</a>  
  📃 <a href="https://arxiv.org/pdf/2602.08939">Paper</a>  
  🌐 <a href="https://example.com/project">Project Page</a>
</p>

### Newest Update
3. [2026/2/8] Benchmark: How to test LLM for what they know and what they don't know? 📃 <a href="https://arxiv.org/pdf/2602.08939">CausalT5KBench Paper</a> 
2. [2026/1/28] Benchmark: 📃 <a href="https://arxiv.org/pdf/2601.08258">T3Bench Paper (ACL 2026)</a> 
1. [2026/1/28] Post-training: How to teach LLM to specify what they know and what they don't know? 📃 <a href="https://arxiv.org/abs/2601.23133">RAudit Paper (in review)</a>  

## Overview

CausalT5K is a comprehensive benchmark dataset designed to evaluate and diagnose critical failure modes in large language models' causal reasoning capabilities. This dataset focuses on four key diagnostic problems:

1. **P1-The Skepticism Trap**: The Skepticism Trap occurs when a model systematically prioritizes safety over correctness, rejecting a large fraction of valid causal claims even when sufficient evidence is available. In this regime, high safety scores coexist with severely degraded utility.

2. **P2-Inverse Scaling of Sycophancy**: Inverse Scaling of Sycophancy refers to the phenomenon where more capable or higher-capacity models become more, not less, susceptible to endorsing incorrect causal claims under social or rhetorical pressure.

3. **P3-The Detection-Correction Gap**: The Detection-Correction Gap arises when a model correctly identifies the presence of a causal trap or insufficiency but fails to act on that recognition by revising its final answer.

4. **P4-Rung Collapse**: Rung Collapse occurs when a model answers higher-order causal queries (diagnostic or counterfactual) using only lower-rung associative evidence, effectively ignoring the epistemic requirements of the task.

## Dataset Structure

The dataset contains over 6,000 causal reasoning cases organized across multiple dimensions:

### Dataset Summary by Domain

The dataset is organized into 10 main domains (D1-D10) with the following Pearl level distribution. Unique Total refers to the number of unique case ids. Valid Total refers to the number where a final score of each dataset example ≥ 9.0.

| Domain | L1 | L2 | L3 | Unique Total | Valid Total |
|--------|----|----|----|--------------|-------------|
| Daily Life (D1) | 131 | 558 | 221 | 910 | 771 |
| History (D2) | 51 | 309 | 134 | 494 | 437 |
| Markets & Finance (D3) | 88 | 349 | 183 | 620 | 422 |
| Medicine & Health (D4) | 180 | 1,000 | 341 | 1,521 | 1,233 |
| Economics (D5) | 80 | 310 | 117 | 507 | 498 |
| Environment & Climate (D6) | 14 | 251 | 26 | 291 | 239 |
| Law & Ethics (D7) | 113 | 345 | 125 | 583 | 532 |
| AI & Technology (D8) | 60 | 368 | 185 | 613 | 611 |
| Sports & Performance (D9) | 38 | 120 | 48 | 206 | 204 |
| Social Science (D10) | 50 | 257 | 99 | 406 | 200 |
| **Total** | **805** | **3,867** | **1,479** | **6,151** | **5,147** |

### Pearl Hierarchy Levels
- **L1 (Association)**: Observational data and correlations
- **L2 (Intervention)**: Causal effects under interventions
- **L3 (Counterfactual)**: Counterfactual reasoning about what would have happened

### Label Definitions
The dataset uses three label types across all Pearl levels (L1, L2, L3):

- **YES**: Cases where the claim matches the scenario, and no trap is identified (only L1 has a sheep type).
- **NO**: Cases where the claim does not support the scenario, and there must be one and only one trap type.
- **AMBIGUOUS**: Cases where the claim can support a scenario, or the claim cannot support a scenario, or there are multiple trap types.

### Trap Type Rules
Trap types are only present for:
- **NO cases** at all Pearl levels (L1, L2, L3): Each NO case must have exactly one trap type.
- **L1 YES cases**: L1 YES cases may have a trap type (sheep type).

All other cases (L2 YES, L3 YES, and all AMBIGUOUS cases at any level) should have `trap` set to `null` or an empty object.

### Causal Traps
The dataset includes various causal reasoning pitfalls:
- **Confounding** (C): Omitted variables, common causes
- **Selection Bias** (W1): Availability, media narrative, survivorship bias
- **Mediation** (M): Intermediate variables
- **Ecological Fallacy** (W5): Aggregate to individual inference errors
- **Regression to Mean** (W4): Statistical artifacts
- And many more...

### Dataset Organization

- **`validated_dataset(round=1)/`**: Contains validated datasets from the first validation round
- **`validated_dataset(round=2)/`**: Contains datasets from the second validation round with additional revisions and second validator scores
- **`final_dataset/`**: Contains the final curated dataset organized by domain (D1-D10), with separate JSON files for each Pearl level (L1, L2, L3)
- **`unvalidated_dataset/`**: Contains unvalidated submissions organized by contributor groups
- **`experiment/`**: Contains experimental results and analysis for each domain (D1-D10), including model predictions, evaluation metrics, and domain-specific writeups
- **`Full_Experiments.tex`**: LaTeX appendix document with complete experimental results, domain-specific tables, and cross-domain aggregation

### Data Format

Each case in the dataset includes:

```json
{
  "id": "unique_case_id",
  "case_id": "case_identifier",
  "bucket": "bucket_name",
  "pearl_level": "L1|L2|L3",
  "domain": "domain_name",
  "subdomain": "subdomain_name",
  "trap": {
    "type": "trap_code",
    "type_name": "Trap Name",
    "subtype": "subtype_code",
    "subtype_name": "Subtype Name"
  },  // null for L2/L3 YES and all AMBIGUOUS cases; required for NO cases and optional for L1 YES cases
  "difficulty": "Easy|Medium|Hard",
  "scenario": "Description of the causal scenario",
  "claim": "Causal claim to evaluate",
  "label": "YES|NO|AMBIGUOUS",
  "is_ambiguous": false,
  "variables": {
    "X": "Variable X description",
    "Y": "Variable Y description",
    "Z": ["Confounding variables"]
  },
  "causal_structure": "Description of causal relationships",
  "key_insight": "Key insight about the causal trap",
  "hidden_timestamp": "Temporal information if relevant",
  "conditional_answers": {
    "answer_if_condition_1": "...",
    "answer_if_condition_2": "..."
  },
  "gold_rationale": "Correct explanation (includes wise refusal content when applicable)",
  "initial_author": "Author name",
  "validator": "Validator name",
  "final_score": 8.5
}
```

## Key Features

### 1. Sycophancy Detection
Cases are designed to test whether models will:
- Agree with user claims that are causally invalid
- Maintain correct causal reasoning despite user framing
- Distinguish between correlation and causation

### 2. Rung Collapse Diagnosis
The dataset systematically tests:
- Whether models can distinguish L1 (association) from L2 (intervention) reasoning
- Whether models collapse counterfactual (L3) reasoning to lower levels
- Appropriate use of causal language at each Pearl level

### 3. Informed Refusal Evaluation
Cases include scenarios where:
- Causal claims cannot be reliably evaluated from available information
- Models should refuse rather than guess
- Appropriate refusal responses are provided as ground truth

## Usage

### Loading the Dataset

The validated datasets are available in JSON format in the `validated_dataset(round=1)/` and `validated_dataset(round=2)/` directories. The final curated dataset is organized in the `final_dataset/` directory with subdirectories for each domain (D1-D10), containing separate JSON files for each Pearl level (L1, L2, L3).

### Evaluation Metrics

When evaluating models on CausalT5K, consider:

1. **Accuracy**: Overall correctness on YES/NO labels
2. **Sycophancy Rate**: Agreement with invalid user claims
3. **Rung Accuracy**: Performance breakdown by Pearl level (L1/L2/L3)
4. **Refusal Quality**: Whether models appropriately refuse when they should
5. **Trap-Specific Performance**: Accuracy across different causal trap types

### Full Experiments Appendix

The `Full_Experiments.tex` file provides comprehensive experimental results across all domains. This appendix includes:

**Models Evaluated:**
- Two closed-source models: Llama-3-8b, Llama-3-70b
- Two open-source models: Deepseek-R1, GPT-4o-2024-08

**Structure:**
- **Domain-Specific Results (D1-D10)**: For each domain, Tables 1-7 (corresponding to Tables A.1-A.7 in the main paper) present:
  - Table 1 (A.1): Rung-wise Performance across L1/L2/L3
  - Table 2 (A.2): Wise Refusal Score by Trap Family
  - Table 3 (A.3): Model Comparison (Utility, Safety, WRS)
  - Table 4 (A.4): Prompt Ablation (Direct, CoT, Structured)
  - Table 5 (A.5): Inter-Annotator Agreement
  - Table 6 (A.6): Error Analysis by Rung and Error Type
  - Table 7 (A.7): Domain Summary Statistics
  - Domain-specific analysis and insights

- **Cross-Domain Aggregation (Tables 9-15)**: Aggregated results across all domains:
  - Table 9: Dataset Composition by Domain
  - Table 10: Rung-wise Results by Domain
  - Table 11: Wise Refusal Score by Domain
  - Table 12: Model Comparison by Domain
  - Table 13: Prompt Ablation by Domain
  - Table 14: Inter-Annotator Agreement by Domain
  - Table 15: Qualitative Error Cases

The appendix provides detailed definitions and explanations for all tables, making it a comprehensive reference for understanding model performance across the CausalT5K benchmark.


## Citation

If you use CausalT5K in your research, please cite:

```bibtex
@article{causalt5k2026,
  title={CausalT5K: Diagnosing Sycophancy, Rung Collapse, and Informed Refusal for Trustworthy Causal Reasoning},
  author={Geng, Longling and Ouyang, Andy and Wu, Theodore and Cooper, Rachael and Zeng, Yuqiao and Hayes, Matthew John and Barretto, Daphne and Vijay, Sameer and Ancone, Gia and Rai, Ankit and Wolfman, Matthew and Flanagan, Patrick and Chang, Edward Y},
  journal={under review},
  year={2026}
}
```

## Dataset Statistics

- **Total Cases**: 5,000+
- **Pearl Level Distribution**: 
  - L1 (Association): ~10%
  - L2 (Intervention): ~60%
  - L3 (Counterfactual): ~30%
- **Domains**: Sports, Healthcare, Economics, Environment, Technology, and more
- **Validation**: Multi-round validation process with expert review

## Contributing

This dataset was created through a collaborative effort. Contributors are organized by groups (A-J), and each submission has been validated for quality and correctness.

## License

[Specify license here]

## Contact

For questions or issues related to the dataset, please [specify contact information].

## Acknowledgments

We thank the following contributors for their work on CausalT5K development.

### Contributors Table

| Name | Major | Level | Generation Domains | Generation Cases Number | Generation Cases ID | Validation Domains | Validation Cases Number | Validation Cases ID |
|------|-------|-------|-------------------|------------------------|---------------------|-------------------|----------------------|-------------------|
| Andy Ouyang | Computer Science | MS | Law & Ethics | 165 | T3-BucketLarge-C-7.1-NC1-T3-BucketLarge-C-7171 | Law & Ethics | 171 | T3-BucketLarge-C-7001-T3-BucketLarge-C-7171 |
| Theodore Wu | Non-Degree (AI) | N/A | Markets | 288 | G.2-G.10 | Markets | 34 | T3-BucketLarge-G.1-T3-BucketLarge-G.9 |
| Rachael Yaran Cooper | Computer Science (AI Track) | MS | Daily Life | 52 | T3-BucketLarge-E-129-T3-BucketLarge-E-180 | Daily Life | 189 | T3-BucketLarge-E-1.100-T3-BucketLarge-E-186 |
| Yuqiao Zeng | Electrical Engineering | MS | Arts, Business, Computer Science, D9, Education, Finance, Health, Public Policy, Science, Sports | 177 | T3-BucketD-0001-T3-BucketLarge-D-9.316 | D9, Sports | 217 | T3-BucketLarge-D-9.100-T3-BucketLarge-D-9.316 |
| Matthew John Hayes | Computer Science (AI Track) | MS | Law & Ethics | 412 | T3-BucketLarge-C-7.mhgen.A.1-T3-BucketLarge-C-7085 | Law & Ethics | 406 | T3-BucketLarge-C-7.1-NC1-T3-BucketLarge-C-7.mhgen.W9.4 |
| Daphne Barretto | Computer Science (AI Track) | MS | Medicine, D10 (Social Science) | 250 | T3-BucketA-0001-T3-BucketJ-81 | D10 (Social Science), Medicine | 170 | T3-BucketJ-01-T3-BucketLarge-A-4.9-P1-2 |
| Sameer Vijay | Business | MS | History | 260 | T3-BucketF-0001-T3-F.99 | D2, History | 200 | T3-BucketLarge-F-181-T3-F.99 |
| Gia Ancone | Computer Science (AI Concentration) | UG | Medicine, Social Science | 407 | T3-BucketLarge-A-4.1-P1-1-T3-BucketLarge-J-A2.1.210 | Medicine, Social Science | 525 | T3-BucketA-0001-T3-BucketLarge-J-A2.1.9 |
| Ankit Rai | Informatics | PhD | Medicine | 158 | T3-BucketLarge-A-4.1-P2-1-T3-BucketLarge-A-A2.1.99 | Medicine | 323 | T3-BucketLarge-A-4.1-P1-1-T3-BucketLarge-A-4.9-P3-2-R4 |
| Matthew Wolfman | Business | MS | D9, Sports | 200 | T3-BucketLarge-D-9.100-T3-BucketLarge-D9-9.542 | D9 | 200 | T3-BucketLarge-D9-9.100-T3-BucketLarge-D9-9.542 |
| Alanood Alrassan | Energy Science and Engineering | MS | AI & Tech | 80 | N/A | D8 - AI Safety & Alignment | 400 | T3-BucketI-L1-001-T3-BucketI-L3-120 |
| Alessandro Balzi | MS in Business + BS & MS in Computer Science | MS | AI & Tech, D8 - AI Safety & Alignment | 384 | T3-BucketLarge-I-L1-006-T3-BucketLarge-I-L3-060 | AI & Tech, D8 - AI Safety & Alignment | 200 | T3-BucketLarge-I-L1-001-T3-BucketLarge-I-L3-060 |
| April Yang | Electrical & Computer Engineering | MS | History | 80 | T3-F.1-T3-F.9 | History | 203 | T3-F1-0001-T3-F1-L2-T15-029 |
| Arya Marwaha | Management Science & Engineering | MS | AI & Tech, Daily Life | 118 | T3-A2-001-T3-BucketLarge-I-L3-003 | Daily Life | 102 | T3-A2-001-T3-A2-102 |
| Atanu Mukherjee | Chemical Engineering | MS | Economics, Business | 315 | T3-BucketLarge-B-5.116-T3-BucketLarge-B-D5-L3-topup-24 | Various | 200 | T3-BucketLarge-B-5.116-T3-BucketLarge-B-D5-L3-topup-24 |
| Chenyang Dai | Computer Science (AI Track) | MS | Daily Life | 337 | T3-BucketLarge-E-1.001-T3-BucketLarge-E-2.125 | Daily Life | 212 | T3-BucketLarge-E-1.001-T3-BucketLarge-E-1.212 |
| Chinmay Pimpalkhare | Computational and Mathematical Engineering | MS | Daily Life | 256 | T3-BucketLarge-E-1.100-T3-BucketLarge-E-1.99 | Daily Life | 230 | T3-BucketLarge-E-1.1-T3-BucketLarge-E-1.99 |
| Chris Pearce | Computer Science | MS | Economics | 219 | T3-BucketLarge-B-1.10-T3-BucketLarge-B-5.479 | Economics | 218 | T3-BucketLarge-B-1.10-T3-BucketLarge-B-6.85 |
| Deveen Manitha Harischandra | Management Science & Engineering | MS | Markets | 79 | T3-BucketLarge-G-genL1-0001-T3-BucketLarge-G.9 | Markets | 160 | T3-BucketLarge-G-genL1-0001-T3-BucketLarge-G-lgren007-0115 |
| Fernando Torres Navarrete | Business | MS | D8 - AI Safety & Alignment | 400 | T3-BucketI-L1-001-T3-BucketI-L3-120 | - | 0 | N/A |
| Jordan Zhang | Bioengineering | MS | Medicine | 558 | T3-BucketA-0001-T3-BucketLarge-A-4.9-P3-2-R3 | Medicine | 19 | T3-BucketA-0019-T3-BucketA-0227 |
| Juli Huang | Computer Science + AI | UG | Markets | 79 | G.10-G.10 | Markets | 220 | G.2-G.10 |
| Kelvin Christian | Non-Degree | N/A | Social Science | 230 | T3-BucketLarge-J-0046-T3-BucketLarge-J-0275 | - | 0 | N/A |
| Leiguang Ren | Non-Degree (AI) | N/A | Markets | 115 | T3-BucketLarge-G-lgren007-0001-T3-BucketLarge-G-lgren007-0115 | Markets | 360 | T3-BucketLargeG-1-T3BucketLargeG-360 |
| Manolo Alvarez | Electrical Engineering | MS | D9, Sports | 172 | T3-BucketLarge-D-022f735f-T3-BucketLarge-D9-9.179 | Arts, Business, Education, Finance, Health | 200 | T3-BucketD-0047-T3-BucketLarge-D-f93e44bf |
| Mason Hu | Computational and Mathematical Engineering | MS | Economics | 192 | T1-BucketSmall-A-5.372-T3-BucketLarge-B-8.00 | Economics | 194 | T1-BucketSmall-A-5.372-T3-BucketLarge-B-5.371 |
| Mingyang Wang | Non-Degree | N/A | History | 30 | T3-BucketF-0151-T3-BucketF-0180 | History | 170 | T3-BucketF-0001-T3-BucketF-0180 |
| Mudit Baid | Computer Science | UG | Medicine | 180 | T3-BucketLarge-A-4.1-P3-1-T3-BucketLarge-A-4.9-P3-2-R4 | Medicine | 360 | T3-BucketLarge-A-4.1-P1-1-T3-BucketLarge-A-4.9-P3-2 |
| Rebecca Joseph | Mathematics | UG | Medicine | 79 | T3-BucketA-0231-T3-BucketLarge-A-new.3-P3-3 | Medicine | 243 | T3-BucketLarge-A-4.1-P1-1-T3-BucketLarge-A-new.3-P3-3 |
| Ryan He | Computer Science | MS | Daily Life | 291 | T3-BucketLarge-E-1.1-T3-BucketLarge-E-186 | Daily Life | 177 | T3-BucketLarge-E-129-T3-BucketLarge-E-2.125 |
| Samantha Afra van Rijs | Electrical Engineering | PhD | Agriculture, Arts, Business, Criminal Justice, Economics, Education, Finance, Health, Public Policy, Science | 251 | T3-BucketD-0041-T3-BucketLarge-D-0183 | Agriculture, Arts, Business, Computer Science, Criminal Justice, Education, Finance, Health, Public Policy, Science | 183 | T3-BucketD-0001-T3-BucketLarge-D-0183 |
| Sreya Vangara | Mechanical Engineering | PhD | D10 (Social Science), Social Science | 879 | T3-BucketJ-01-T3-BucketLarge-J-A2.1.9 | Social Science | 230 | T3-BucketLarge-J-0046-T3-BucketLarge-J-0275 |
| Veljko Skarich | Non-Degree (AI) | N/A | Environment | 259 | T3-BucketH-0006-T3-BucketH-999 | Environment | 254 | T3-BucketH-1001-T3-BucketH-999 |
| Vivek Sathe | MS in Analytics & AI | MS | Economics | 188 | T3-BucketLarge-B-5.061-T3-BucketLarge-B-5.345 | Economics | 302 | T3-BucketLarge-B-5.061-T3-BucketLarge-B-8.00 |
| Ray Du | Management Science & Engineering | MS | D9, Sports | 80 | T3-BucketD-0047-T3-BucketLarge-D-f93e44bf | AI & Tech, D8 - AI Safety & Alignment | 170 | T3-BucketI-L1-001-T3-BucketI-L3-120 |

### Dataset Summary by Domain (Round 1 vs Round 2 Validation)

The following table summarizes the dataset quality scores organized by domain, comparing round=1 and round=2 validation:

| Domain | Total Case Numbers | Case ID Range | Initial Author | First Validator | Rule-based Score (human validation round=1) | Score from Other (human validation round=1) | LLM Score (human validation round=1) | Final Score (human validation round=1) | Second Validator | Rule-based Score (human validation round=2) | Score from Other (human validation round=2) | LLM Score (human validation round=2) | Final Score (human validation round=2) |
|--------|-------------------|---------------|----------------|-----------------|-------------------------------------------|--------------------------------------------|--------------------------------------|----------------------------------------|------------------|-------------------------------------------|--------------------------------------------|--------------------------------------|----------------------------------------|
| Daily Life (D1) | 910 | T3-A2-001-T3-BucketLarge-E-2.125 | Arya Marwaha, Chenyang Dai, Chinmay Pimpalkhare, Rachael Yaran Cooper, Ryan He | Arya Marwaha, Chenyang Dai, Chinmay Pimpalkhare, Rachael Yaran Cooper, Ryan He | 4.00 | 0.68 | 4.30 | 3.49 | Longling Geng | 3.60 | 0.96 | 5.20 | 4.53 |
| History (D2) | 370 | T3-BucketF-0001-T3-F.99 | April Yang, Mingyang Wang, Sameer Vijay | Mingyang Wang, Sameer Vijay | 3.00 | 0.84 | 3.50 | 3.23 | Longling Geng | 3.00 | 0.91 | 4.80 | 4.11 |
| Markets & Finance (D3) | 635 | G.2-G.10 | Samantha Afra van Rijs, Yuqiao Zeng, Deveen Manitha Harischandra, Theodore Wu, Juli Huang, Leiguang Ren | Deveen Manitha Harischandra, Manolo Alvarez, Samantha Afra van Rijs, Theodore Wu, Juli Huang, Leiguang Ren | 4.50 | 0.77 | 5.25 | 4.03 | Longling Geng | 4.50 | 0.90 | 5.00 | 4.66 |
| Medicine & Health (D4) | 1522 | T3-BucketA-0001-T3-BucketLarge-J-A2.1.210 | Ankit Rai, Daphne Barretto, Gia Ancone, Jordan Zhang, Mudit Baid, Rebecca Joseph | Ankit Rai, Daphne Barretto, Gia Ancone, Jordan Zhang, Mudit Baid, Rebecca Joseph | 4.00 | 0.91 | 4.42 | 3.90 | Longling Geng | 4.00 | 0.94 | 5.10 | 4.59 |
| Economics (D5) | 507 | T1-BucketSmall-A-5.372-T3-BucketLarge-B-8.00 | Chris Pearce, Mason Hu, Samantha Afra van Rijs, Vivek Sathe, Atanu Mukherjee | Chris Pearce, Mason Hu, Samantha Afra van Rijs, Vivek Sathe, Atanu Mukherjee | 3.75 | 0.74 | 4.00 | 3.44 | Longling Geng | 3.75 | 0.97 | 5.30 | 4.63 |
| Environment & Climate (D6) | 292 | T3-BucketD-0097-T3-BucketH-999 | Samantha Afra van Rijs, Veljko Skarich | Samantha Afra van Rijs, Veljko Skarich | 3.00 | 0.41 | 3.00 | 2.40 | Longling Geng | 3.00 | 0.87 | 4.50 | 3.95 |
| Law & Ethics (D7) | 584 | T3-BucketD-0086-T3-BucketLarge-C-7171 | Andy Ouyang, Matthew John Hayes, Samantha Afra van Rijs | Andy Ouyang, Matthew John Hayes, Samantha Afra van Rijs | 4.67 | 0.91 | 5.50 | 4.39 | Longling Geng | 4.67 | 0.95 | 5.20 | 4.88 |
| AI & Technology (D8) | 615 | T3-BucketD-0022-T3-BucketLarge-I-L3-060 | Alessandro Balzi, Arya Marwaha, Fernando Torres Navarrete, Alanood Alrassan | Alanood Alrassan, Alessandro Balzi, Samantha Afra van Rijs, Ray Du | 4.00 | 0.85 | 4.25 | 3.76 | Longling Geng | 3.50 | 0.94 | 5.00 | 4.40 |
| Sports & Performance (D9) | 545 | T3-BucketD-0001-T3-BucketLarge-D9-9.542 | Manolo Alvarez, Matthew Wolfman, Samantha Afra van Rijs, Yuqiao Zeng, Ray Du | Manolo Alvarez, Matthew Wolfman, Samantha Afra van Rijs, Yuqiao Zeng | 4.25 | 0.84 | 4.88 | 3.97 | Longling Geng | 4.25 | 0.96 | 5.10 | 4.71 |
| Social Science (D10) | 492 | T3-BucketD-0011-T3-BucketLarge-J-A2.1.9 | Daphne Barretto, Gia Ancone, Kelvin Christian, Sreya Vangara | Daphne Barretto, Gia Ancone, Manolo Alvarez, Sreya Vangara | 4.00 | 0.75 | 4.50 | 3.66 | Longling Geng | 4.00 | 0.89 | 4.60 | 4.35 |

**Score Definitions:**
- **Rule-based Score (round=1)**: Average `final_score` from `assignment2.csv` for initial authors in this domain
- **Rule-based Score (round=2)**: Average `final_score` from `grade_assignment2.py` output for initial authors in this domain (uses same grading script as round=1, so scores should be comparable)
- **Score from Other (round=1)**: Average `validatee_score` from validators who validated cases in this domain (0-1 scale, normalized to 0-5 in final calculation) 
- **Score from Other (round=2)**: Average `final_score_2` from round=2 datasets (0-1 scale, normalized from 0-10) - represents scores given by the second validator (Longling Geng)
- **LLM Score (round=1)**: Average `final_score` from `assignment2_with_llm.csv` (Evaluated using Auto Agent, 0-7 scale normalized to 0-5 in final calculation)
- **LLM Score (round=2)**: Evaluated using Auto Agent based on case quality metrics (0-5 scale, directly used in final calculation)
- **Final Score**: Average of available scores (all normalized to 0-5 scale). For round=2, only Rule-based Score and Score from Other are used in the calculation.
