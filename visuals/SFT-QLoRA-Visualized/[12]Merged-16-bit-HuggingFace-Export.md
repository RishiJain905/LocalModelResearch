```mermaid
flowchart TB
    A["Fine-tuned model exists as:<br/>base model + LoRA adapter"] --> B["Prepare Hugging Face authentication"]

    B --> C["from huggingface_hub import whoami"]
    B --> D["from google.colab import userdata"]

    D --> E["hf_token = userdata.get('HF_TOKEN')"]
    E --> F{"HF_TOKEN exists?"}
    F -->|No| G["Raise error:<br/>HF_TOKEN not found"]
    F -->|Yes| H["Authenticate with Hugging Face"]

    H --> I["username = whoami(token=hf_token)['name']"]
    I --> J["repo_id = username/Qwopus3.5-27B"]

    J --> K["model.push_to_hub_merged(...)"]

    K --> L["save_method = merged_16bit"]
    L --> M["Merge process"]
    M --> M1["Take frozen base model"]
    M --> M2["Apply trained LoRA updates"]
    M --> M3["Create one standalone 16-bit model"]

    M3 --> N["Upload full merged model to HF Hub"]

    N --> O["Pros"]
    O --> O1["Self-contained model"]
    O --> O2["Easier Transformers loading"]
    O --> O3["No separate adapter needed at inference"]

    N --> P["Cons"]
    P --> P1["Very large"]
    P --> P2["Slower upload"]
    P --> P3["More disk/RAM required"]
    P --> P4["Overkill for quick experiments"]

    P --> Q["Best used when:<br/>adapter has been tested and is worth publishing"]
```