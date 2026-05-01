```mermaid
flowchart TB
    A["Trainer configured"] --> B["Response-only masking applied"]
    B --> C["Label sanity check passed"]

    C --> D["trainer_stats = trainer.train()"]

    D --> E["Training loop begins"]

    E --> F["Load batch from dataset"]
    F --> G["Model forward pass"]
    G --> G1["Input goes through frozen 4-bit base model"]
    G --> G2["LoRA side-path modifies target modules"]

    G2 --> H["Model predicts next tokens"]

    H --> I["Compute loss"]
    I --> I1["Loss only on unmasked labels"]
    I1 --> I2["User/template tokens ignored if label = -100"]

    I --> J["Backward pass"]
    J --> J1["Gradients flow into LoRA A/B weights"]
    J --> J2["Frozen base weights do not update"]

    J1 --> K["Optimizer step"]
    K --> K1["adamw_8bit updates LoRA parameters"]

    K --> L["Logging"]
    L --> L1["Loss"]
    L --> L2["Learning rate"]
    L --> L3["Runtime/progress"]

    K --> M{"Checkpoint step reached?"}
    M -->|Yes| N["Save checkpoint to output_dir"]
    M -->|No| O["Continue training"]

    N --> O
    O --> P{"Training complete?"}
    P -->|No| F
    P -->|Yes| Q["Return TrainOutput object"]

    Q --> R["Stored as trainer_stats"]
    R --> S["View metrics with:<br/>trainer_stats.metrics"]

    S --> T["Possible metrics:<br/>train_loss<br/>train_runtime<br/>samples/sec<br/>steps/sec"]
```