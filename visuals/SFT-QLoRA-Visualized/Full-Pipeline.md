```mermaid
flowchart TB
    A["Start"] --> B["Set up Colab GPU runtime"]
    B --> C["Install / import libraries"]
    C --> D["Load Qwen3.5-27B with Unsloth"]
    D --> D1["4-bit loading"]
    D --> D2["max_seq_length set"]
    D --> D3["full_finetuning = False"]

    D1 --> E["Attach LoRA adapters"]
    D2 --> E
    D3 --> E

    E --> E1["target_modules:<br/>q_proj, k_proj, v_proj, o_proj,<br/>gate_proj, up_proj, down_proj"]
    E1 --> E2["Base weights frozen"]
    E2 --> E3["LoRA weights trainable"]

    E3 --> F["Load datasets from Hugging Face"]
    F --> F1["ds1"]
    F --> F2["ds2"]
    F --> F3["ds3"]

    F1 --> G["Normalize schemas"]
    F2 --> G
    F3 --> G

    G --> G1["All datasets become conversations"]
    G1 --> G2["Assistant format:<br/><think>...</think><br/>answer"]

    G2 --> H["Apply Qwen thinking chat template"]
    H --> H1["Create dataset['text']"]

    H1 --> I["Filter long samples"]
    I --> I1["Keep samples <= MAX_CONTEXT_WINDOW"]

    I1 --> J["Format QA"]
    J --> J1["Verify think tags"]

    J1 --> K["Create SFTTrainer"]
    K --> K1["Set batch size"]
    K --> K2["Set grad accumulation"]
    K --> K3["Set learning rate"]
    K --> K4["Set optimizer/checkpointing/logging"]

    K4 --> L["Apply train_on_responses_only"]
    L --> L1["Mask user/template labels to -100"]
    L1 --> L2["Train only assistant responses"]

    L2 --> M["Label sanity check"]
    M --> M1["Decode labels to verify masking"]

    M1 --> N["trainer_stats = trainer.train()"]
    N --> N1["Update only LoRA weights"]
    N1 --> N2["Base model remains frozen"]

    N2 --> O["Save / deploy"]
    O --> O1["Adapter-only save"]
    O --> O2["Merged 16-bit HF export"]
    O --> O3["GGUF export"]

    O1 --> P["Best for experiments"]
    O2 --> Q["Best for standalone HF model"]
    O3 --> R["Best for llama.cpp/local runtime"]
```