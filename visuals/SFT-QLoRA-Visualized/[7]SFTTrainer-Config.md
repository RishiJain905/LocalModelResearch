```mermaid
flowchart TB
    A["Final dataset ready"] --> B["Import TRL"]
    B --> B1["from trl import SFTTrainer, SFTConfig"]

    B1 --> C["Create SFTTrainer"]

    C --> D["model = model"]
    D --> D1["Qwen base model<br/>with LoRA adapters attached"]

    C --> E["tokenizer = tokenizer"]
    E --> E1["Qwen thinking chat-template tokenizer"]

    C --> F["train_dataset = dataset"]
    F --> F1["Uses final text column"]

    C --> G["eval_dataset = None"]
    G --> G1["No validation split in this guide"]

    C --> H["SFTConfig"]

    H --> I["dataset_text_field = text"]
    I --> I1["Trainer trains on dataset['text']"]

    H --> J["per_device_train_batch_size = 6"]
    J --> J1["Mini-batch size per GPU"]

    H --> K["gradient_accumulation_steps = 6"]
    K --> K1["Accumulate gradients over multiple mini-batches"]

    J --> L["Effective batch size"]
    K --> L
    L --> L1["6 × 6 = 36"]

    H --> M["warmup_ratio = 0.03"]
    M --> M1["First 3% of training gradually ramps LR"]

    H --> N["num_train_epochs = 1"]
    N --> N1["Train over full dataset once"]

    H --> O["Optional max_steps"]
    O --> O1["Useful for debug runs<br/>Example: max_steps = 50"]

    H --> P["learning_rate = 2e-4"]
    P --> P1["Common LoRA/QLoRA SFT LR"]
    P1 --> P2["For longer serious runs,<br/>lower LR may be safer"]

    H --> Q["logging_steps = 1"]
    Q --> Q1["Log every training step"]

    H --> R["optim = adamw_8bit"]
    R --> R1["Memory-efficient optimizer"]

    H --> S["weight_decay = 0.001"]
    S --> S1["Small regularization"]

    H --> T["lr_scheduler_type = linear"]
    T --> T1["LR decreases linearly after warmup"]

    H --> U["seed = 3407"]
    U --> U1["Reproducibility"]

    H --> V["save_steps = 200"]
    V --> V1["Save checkpoint every 200 steps"]

    H --> W["save_total_limit = 1"]
    W --> W1["Keep only latest checkpoint"]

    H --> X["report_to = wandb"]
    X --> X1["Logs to Weights & Biases<br/>if configured"]
    X --> X2["Use report_to = none<br/>if not using WandB"]

    H --> Y["output_dir = drive_output_path"]
    Y --> Y1["Saves checkpoints to Google Drive path"]

    C --> Z["Trainer object is prepared<br/>but training has not started yet"]
```