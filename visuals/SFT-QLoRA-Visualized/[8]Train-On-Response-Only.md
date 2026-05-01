```mermaid
flowchart TB
    A["SFTTrainer created"] --> B["Import helper"]
    B --> B1["from unsloth.chat_templates import train_on_responses_only"]

    B1 --> C["trainer = train_on_responses_only(...)"]

    C --> D["instruction_part = <|im_start|>user\\n"]
    C --> E["response_part = <|im_start|>assistant\\n<think>"]

    D --> D1["Tells Unsloth where user/instruction spans begin"]
    E --> E1["Tells Unsloth where assistant response span begins"]

    C --> F["Trainer dataset labels are modified"]

    F --> G["Input IDs"]
    G --> G1["Full conversation still visible to model"]
    G1 --> G2["User prompt + assistant answer + template tokens"]

    F --> H["Labels"]
    H --> H1["Tokens outside assistant response are set to -100"]
    H1 --> H2["-100 means ignore in loss calculation"]

    H --> I["Loss is calculated only on assistant-side tokens"]
    I --> J["Model learns assistant behavior"]
    J --> J1["reasoning format"]
    J --> J2["answer style"]
    J --> J3["task solving patterns"]

    I --> K["Model does NOT waste training on copying user prompts"]
    K --> K1["Better instruction-following preservation"]

    C --> L["Important dependency"]
    L --> L1["response_part must match actual Qwen template output"]
    L1 --> L2["Previous sample should show:<br/><|im_start|>assistant<br/><think>"]
```