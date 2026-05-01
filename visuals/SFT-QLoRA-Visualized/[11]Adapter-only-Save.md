```mermaid
flowchart TB
    A["Training complete"] --> B["Save adapter-only checkpoint"]

    B --> C["model.save_pretrained('qwen_lora')"]
    B --> D["tokenizer.save_pretrained('qwen_lora')"]

    C --> E["Saves LoRA adapter weights"]
    D --> F["Saves tokenizer/config files"]

    E --> G["Important:<br/>This is NOT the full 27B model"]
    G --> H["It saves only the learned LoRA changes"]

    H --> I["Why this works"]
    I --> I1["Base model was frozen"]
    I --> I2["Only LoRA A/B matrices changed"]
    I --> I3["Adapter contains those trained A/B weights"]

    I3 --> J["To use later"]
    J --> J1["Load same compatible base model"]
    J --> J2["Load qwen_lora adapter"]
    J --> J3["Attach adapter onto base model"]
    J --> J4["Inference uses base + adapter"]

    H --> K["Benefits"]
    K --> K1["Much smaller than merged 27B model"]
    K --> K2["Fast to save"]
    K --> K3["Good for experiments"]
    K --> K4["Can store many fine-tune variants"]

    B --> L["Recommended Colab tweak"]
    L --> L1["Save to Google Drive:<br/>/content/drive/MyDrive/qwen_lora"]
    L1 --> L2["Prevents loss when Colab runtime disconnects"]
```