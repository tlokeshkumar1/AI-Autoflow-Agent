---
base_model: Qwen/Qwen3.5-9B
library_name: peft
pipeline_tag: text-generation
tags:
- base_model:adapter:Qwen/Qwen3.5-9B
- lora
- transformers
- finance
- accounts-receivable
- agent
---

# AutoFlow Pilot Qwen-3.5-9B (V4)

**AutoFlow Pilot V4** is a LoRA adapter fine-tuned on top of `Qwen/Qwen3.5-9B` specifically designed for autonomous enterprise Accounts Receivable (AR) management, policy enforcement, invoice reconciliation, and cross-platform orchestration across ERP (SAP S/4HANA), CRM (Salesforce), and ITSM (ServiceNow).

## Model Details

- **Model Name:** AutoFlow Pilot Qwen-3.5-9B (V4)
- **Base Model:** [Qwen/Qwen3.5-9B](https://huggingface.co/Qwen/Qwen3.5-9B)
- **Adapter Type:** PEFT LoRA
- **LoRA Hyperparameters:**
  - `r`: 8
  - `lora_alpha`: 16
  - `lora_dropout`: 0.0
  - `target_modules`: `q_proj`, `k_proj`, `v_proj`, `o_proj`, `gate_proj`, `up_proj`, `down_proj`, `out_proj`, `in_proj_a`, `in_proj_b`, `in_proj_z`, `in_proj_qkv`
  - `bias`: `none`
  - `task_type`: `CAUSAL_LM`
- **Precision:** Native `bfloat16`
- **Adapter Weights SHA256:** `3bcb11bfe42a364ddcfe33187ceed2f3e0a827e9a7c76137f9d92d3a30a7e873`
- **Parent V3 Adapter SHA256:** `e12764c4929333f49e5ae4b58c4fb39e349a8497ac329f86f6236ea2963c4d0d`

## Intended Use

- **Direct Use:**
  - Automated evaluation of accounts receivable against corporate billing policy.
  - Deterministic decision mapping (`review_hold`, `skip_paid`, `skip_not_due`, `approval_required`, `task_and_draft`).
  - Safe generation of customized collection follow-up communications and action plans.
  - Multi-turn tool calling and structured protocol summaries.
- **Out of Scope:**
  - Unsupervised execution of monetary transfers or ledger write-offs without human approval.
  - Using untrusted invoice fields as executable system instructions.

## Policy Enforcement Logic

The adapter is trained to follow strict hierarchical evaluation:
1. `on_hold == True` $\rightarrow$ `review_hold`
2. `outstanding_minor == 0` $\rightarrow$ `skip_paid`
3. `due_date >= as_of` $\rightarrow$ `skip_not_due`
4. `approval_required == True` $\rightarrow$ `approval_required`
5. Otherwise $\rightarrow$ `task_and_draft`

## Verification & Benchmark Summary

| Benchmark Suite | Result | Status |
| :--- | :---: | :---: |
| SAP Open-Item Audit Self-Test | PASSED | Verified |
| Due-Date Backend Guard Self-Test | PASSED | Verified |
| Salesforce & ServiceNow 16-Case Suite | 16/16 PASSED | Verified |
| Regression Case `V3-TEST-D-0001-e4a7d3d26a30` | 200/200 PASSED | Verified |
| Regression Case `V3-VALIDATION-D-0009-8c3a7629cbde` | 200/200 PASSED | Verified |
| Multi-Turn Tool Calling (`bfloat16`) | 4/4 PASSED | Verified |
| Final Workflow Summary Protocol (`bfloat16`) | 4/4 PASSED | Verified |

## How to Get Started

```python
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer
from peft import PeftModel

base_model_name = "Qwen/Qwen3.5-9B"
adapter_path = "path/to/autoflow-pilot-qwen35-v4"

tokenizer = AutoTokenizer.from_pretrained(adapter_path, trust_remote_code=True)
base_model = AutoModelForCausalLM.from_pretrained(
    base_model_name,
    torch_dtype=torch.bfloat16,
    device_map="auto",
    trust_remote_code=True
)
model = PeftModel.from_pretrained(base_model, adapter_path)
model.eval()
```

### Framework Versions
- PEFT 0.18.1 / 0.21.2
- Transformers 4.40+
- PyTorch 2.1+
