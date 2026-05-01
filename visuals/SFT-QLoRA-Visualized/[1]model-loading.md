```mermaid
flowchart TB
    A["Start Colab Runtime<br/>GPU runtime selected"] --> B["Import libraries<br/>from unsloth import FastLanguageModel<br/>import torch"]

    B --> C["Reference list:<br/>fourbit_models = [...]"]
    C --> C_NOTE["Important:<br/>This list is only examples<br/>It does NOT load all models"]

    B --> D["FastLanguageModel.from_pretrained(...)"]

    D --> E["model_name = unsloth/Qwen3.5-27B"]
    D --> F["max_seq_length = 8192 / 16384 / 32768"]
    D --> G["load_in_4bit = True"]
    D --> H["load_in_8bit = False"]
    D --> I["full_finetuning = False"]

    E --> E1["Downloads/loads the selected base model<br/>from Hugging Face / Unsloth repo"]

    F --> F1["Controls maximum context length<br/>Higher = more VRAM + slower training<br/>Lower = safer for first run"]

    G --> G1["Loads base model weights in 4-bit<br/>Major VRAM savings"]
    G1 --> G2["4-bit base + LoRA later<br/>= QLoRA-style setup"]

    H --> H1["Do not load 8-bit weights<br/>8-bit uses more VRAM than 4-bit"]

    I --> I1["Do not train all base weights<br/>Base model will stay frozen"]
    I1 --> I2["Later we train LoRA adapters only"]

    D --> J["Returns two objects"]
    J --> K["model<br/>The neural network"]
    J --> L["tokenizer<br/>Text ↔ token IDs<br/>Also handles chat template later"]

    K --> M["Ready for LoRA adapter insertion"]
    L --> N["Ready for Qwen chat-template setup"]
```