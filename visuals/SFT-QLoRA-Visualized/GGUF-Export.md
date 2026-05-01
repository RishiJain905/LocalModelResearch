```mermaid
flowchart TB
    A["Fine-tuned model after training"] --> B["Need local llama.cpp / LM Studio compatible model?"]

    B -->|Yes| C["Export GGUF"]
    B -->|No| D["Adapter-only or merged HF may be enough"]

    C --> E["Authenticate with HF_TOKEN"]
    E --> F["repo_id = username/Qwopus3.5-27B-GGUF"]

    F --> G["model.push_to_hub_gguf(...)"]

    G --> H["quantization_method list"]
    H --> H1["q4_k_m"]
    H --> H2["q8_0"]
    H --> H3["bf16"]

    H1 --> I["q4_k_m"]
    I --> I1["Smaller GGUF"]
    I --> I2["Good VRAM efficiency"]
    I --> I3["Common llama.cpp choice"]

    H2 --> J["q8_0"]
    J --> J1["Larger than q4"]
    J --> J2["Higher quality"]
    J --> J3["Useful if VRAM/storage allows"]

    H3 --> K["bf16"]
    K --> K1["Very large"]
    K --> K2["Near full precision"]
    K --> K3["Usually not needed for first local test"]

    G --> L["Upload GGUF files to Hugging Face Hub"]

    L --> M["Use later with"]
    M --> M1["llama.cpp"]
    M --> M2["LM Studio"]
    M --> M3["OpenWebUI with GGUF backend"]
    M --> M4["custom local inference stacks"]

    C --> N["Practical recommendation"]
    N --> N1["First export q4_k_m only"]
    N --> N2["Then q8_0 if quality comparison needed"]
    N --> N3["Skip bf16 unless you have a strong reason"]
```