---
base_model: Qwen/Qwen3.5-9B
library_name: peft
pipeline_tag: text-generation
tags:
- base_model:adapter:Qwen/Qwen3.5-9B
- lora
- transformers
- enterprise-agent
- multi-domain
- tool-calling
- accounts-receivable
- itsm
- erp
---

# AutoFlow Pilot Qwen-3.5-9B (V5 Multi-Domain)

**AutoFlow Pilot V5 Multi-Domain** is an enterprise-grade Parameter-Efficient Fine-Tuning (PEFT LoRA) adapter trained on top of `Qwen/Qwen3.5-9B`. Building upon the deterministic financial policy foundation of V4, V5 expands the agent's capabilities into **multi-domain enterprise operations**—orchestrating complex workflows across ERP financial ledgers (**SAP S/4HANA**), Customer Relationship Management (**Salesforce CRM**), and IT & Customer Service Management (**ServiceNow**).

## Model Details

- **Model Identifier:** `autoflow-pilot-qwen35-v5-multidomain`
- **Base Model:** [`Qwen/Qwen3.5-9B`](https://huggingface.co/Qwen/Qwen3.5-9B)
- **Adapter Type:** PEFT LoRA (Low-Rank Adaptation)
- **Fine-Tuning Architecture:**
  - **LoRA Rank ($r$):** 8
  - **LoRA Alpha ($\alpha$):** 16
  - **LoRA Dropout:** 0.0
  - **Target Modules (12 projections across all attention & MLP blocks):**
    `q_proj`, `k_proj`, `v_proj`, `o_proj`, `gate_proj`, `up_proj`, `down_proj`, `out_proj`, `in_proj_a`, `in_proj_b`, `in_proj_z`, `in_proj_qkv`
  - **Trainable LoRA Tensors:** 496 tensors
  - **Bias:** `none`
  - **Task Type:** `CAUSAL_LM`
- **Inference Precision:** Native `bfloat16` or 4-bit `NF4` with BitsAndBytes
- **Adapter Model Checksum:** SHA256: `a8d2945515179b8b3e8ef0fdeb46521e6f898bc9b973f2f838ea50b3486fe2ba`
- **Parent V4 Checksum:** SHA256: `3bcb11bfe42a364ddcfe33187ceed2f3e0a827e9a7c76137f9d92d3a30a7e873`
- **Distribution Bundle:** `autoflow-v5-multidomain-bundle.zip` (SHA256: `39a2292e86264bd97b121d18c5a1ddf8b7e3ef4463a6eb3bbd4947a92c818900`)

---

## Capabilities & Multi-Domain Extensions

1. **Cross-Platform Enterprise Orchestration**:
   - **Financial Ledgers (SAP S/4HANA)**: Audit trails, unsettled receivables queries (`A_OperationalAcctgDocItemCube`), invoice currency normalization, payment reconciliation.
   - **CRM Customer Accounts (Salesforce)**: Contact discovery, account manager lookups, credit hold governance, and tailored communication scheduling.
   - **Service Management & Escalations (ServiceNow)**: Creation and lifecycle routing of `HOLD_ESCALATION` incident tickets and `AR_FOLLOWUP_TASK` tasks.
   - **Local Immutable State**: Persistent SQLite logging for SOC-2 and corporate audit compliance.

2. **Deterministic Policy Rules**:
   - Strict hierarchical enforcement:
     1. `on_hold == True` $\rightarrow$ `review_hold`
     2. `outstanding_minor == 0` $\rightarrow$ `skip_paid`
     3. `due_date >= as_of` $\rightarrow$ `skip_not_due`
     4. `approval_required == True` $\rightarrow$ `approval_required`
     5. Otherwise $\rightarrow$ `task_and_draft`
   - Explicit confirmation lock: `claims_business_action_executed` is strictly bound to `false` until verified through backend systems.

3. **Multi-Domain Tool Calling & Error Recovery**:
   - Multi-turn function calling with JSON schema validation.
   - Resilient recovery handling for transient errors (HTTP 429, 503, connection timeouts).
   - Separation of data fields from operational prompts to prevent indirect prompt injection.

---

## How to Get Started

### Quick Inference Example

```python
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer, BitsAndBytesConfig
from peft import PeftModel

base_model_id = "Qwen/Qwen3.5-9B"
adapter_id = "./autoflow-pilot-qwen35-v5-multidomain"

# Load Tokenizer
tokenizer = AutoTokenizer.from_pretrained(adapter_id, trust_remote_code=True)

# Load Base Model (4-bit quantized or bfloat16)
quant_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_compute_dtype=torch.bfloat16,
    bnb_4bit_use_double_quant=True,
    bnb_4bit_quant_type="nf4"
)

base_model = AutoModelForCausalLM.from_pretrained(
    base_model_id,
    quantization_config=quant_config,
    device_map="auto",
    torch_dtype=torch.bfloat16,
    trust_remote_code=True
)

# Attach LoRA Adapter
model = PeftModel.from_pretrained(base_model, adapter_id)
model.eval()

# Run Inference
messages = [
    {
        "role": "system",
        "content": "You are AutoFlow AI, an enterprise workflow pilot. Evaluate incoming financial records and generate safe, schema-valid action decisions."
    },
    {
        "role": "user",
        "content": '{"invoice_id": "INV-2026-1049", "customer_id": "CUST-10492", "outstanding_minor": 1425000, "on_hold": false, "due_date": "2026-09-15", "as_of": "2026-10-31"}'
    }
]

prompt = tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=True, enable_thinking=False)
inputs = tokenizer(prompt, return_tensors="pt").to(model.device)

with torch.no_grad():
    outputs = model.generate(
        **inputs,
        max_new_tokens=128,
        do_sample=False,
        pad_token_id=tokenizer.eos_token_id
    )

print(tokenizer.decode(outputs[0][inputs.input_ids.shape[1]:], skip_special_tokens=True).strip())
```

---

## Framework Compatibility

- **PEFT**: `0.18.1` / `0.21.2`
- **Transformers**: `>= 4.40.0`
- **PyTorch**: `>= 2.1.0`
- **BitsAndBytes**: `>= 0.43.0`
- **Safetensors**: `>= 0.4.0`
