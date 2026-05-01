```mermaid
flowchart TB
    A["After train_on_responses_only"] --> B["Run label sanity check"]

    B --> C["trainer.train_dataset[100]['labels']"]
    C --> C1["Gets labels for sample 100"]

    C1 --> D["Labels contain token IDs and -100 values"]
    D --> D1["Real token ID = train on this token"]
    D --> D2["-100 = ignore this token during loss"]

    D --> E["Problem:<br/>tokenizer cannot decode -100"]
    E --> F["Replace -100 with tokenizer.pad_token_id"]

    F --> G["tokenizer.decode([...])"]
    G --> G1["Converts labels back into readable text"]

    G1 --> H["replace(tokenizer.pad_token, ' ')"]
    H --> H1["Hides ignored tokens visually"]

    H1 --> I["Inspect output"]

    I --> J{"What should you see?"}
    J -->|Good| K["Mostly assistant response text<br/><think>...</think><br/>final answer"]
    J -->|Bad| L["Full user prompt visible<br/>or empty/broken assistant labels"]

    K --> M["Masking likely worked"]
    L --> N["Check response_part / instruction_part<br/>and printed dataset template"]

    M --> O["Safe to start training"]
```