```mermaid
flowchart TB
    A["Loaded Qwen3.5-27B base model"] --> B["FastLanguageModel.get_peft_model(...)"]

    B --> C["Attach LoRA adapters"]
    C --> D["Base model weights stay frozen"]
    C --> E["Small LoRA weights become trainable"]

    B --> F["r = 64"]
    F --> F1["LoRA rank<br/>Controls adapter capacity"]
    F1 --> F2["Higher rank:<br/>more capacity<br/>more VRAM<br/>larger adapter"]
    F1 --> F3["Lower rank:<br/>lighter<br/>faster<br/>less capacity"]

    B --> G["target_modules"]
    G --> H["Attention layers"]
    H --> H1["q_proj"]
    H --> H2["k_proj"]
    H --> H3["v_proj"]
    H --> H4["o_proj"]

    G --> I["MLP / feed-forward layers"]
    I --> I1["gate_proj"]
    I --> I2["up_proj"]
    I --> I3["down_proj"]

    H1 --> J["LoRA inserted into selected linear modules"]
    H2 --> J
    H3 --> J
    H4 --> J
    I1 --> J
    I2 --> J
    I3 --> J

    J --> K["Original layer:<br/>y = W x"]
    K --> L["LoRA layer:<br/>y = W x + scale * B A x"]
    L --> M["W is frozen<br/>A and B are trainable"]

    B --> N["lora_alpha = 64"]
    N --> N1["Scales LoRA update strength"]

    B --> O["lora_dropout = 0"]
    O --> O1["No dropout<br/>Fast/optimized Unsloth setting"]

    B --> P["bias = none"]
    P --> P1["Do not train bias terms"]

    B --> Q["use_gradient_checkpointing = unsloth"]
    Q --> Q1["Saves VRAM"]
    Q1 --> Q2["Recomputes some activations during backward pass"]
    Q2 --> Q3["Useful for long context + large model"]

    B --> R["random_state = 3407"]
    R --> R1["Reproducibility"]

    B --> S["use_rslora = False<br/>loftq_config = None"]
    S --> S1["Advanced LoRA variants disabled for baseline"]

    C --> T["Result:<br/>4-bit frozen base model<br/>+ trainable LoRA adapters"]
```