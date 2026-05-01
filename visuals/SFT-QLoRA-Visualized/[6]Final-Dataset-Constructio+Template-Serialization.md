```mermaid
flowchart TB
    A["Normalized datasets:<br/>ds1, ds2, ds3"] --> B["Remove empty conversations"]

    B --> B1["ds1.filter(conversations not empty)"]
    B --> B2["ds2.filter(conversations not empty)"]
    B --> B3["ds3.filter(conversations not empty)"]

    B1 --> C["concatenate_datasets([ds1, ds2, ds3])"]
    B2 --> C
    B3 --> C

    C --> D["Shuffle combined dataset<br/>using RANDOM_SEED"]
    D --> E["Define formatting_prompts_func"]

    E --> F["For each conversation"]
    F --> G["tokenizer.apply_chat_template(...)"]

    G --> H["tokenize = False"]
    H --> H1["Return formatted text string<br/>not token IDs yet"]

    G --> I["add_generation_prompt = False"]
    I --> I1["Correct for training<br/>assistant answer already included"]

    G --> J["Produces Qwen-formatted training text"]
    J --> J1["<|im_start|>user<br/>...<br/><|im_end|><br/><|im_start|>assistant<br/><think>...</think><br/>answer<br/><|im_end|>"]

    J --> K["dataset = combined_dataset.map(formatting_prompts_func)"]
    K --> L["Dataset now has text column"]

    L --> M["num_proc = mp.cpu_count()"]
    M --> N["_text_tok = tokenizer.tokenizer if available<br/>else tokenizer"]

    N --> O["filter_long_sequences_batched"]
    O --> O1["Tokenize text with:<br/>truncation=False<br/>padding=False<br/>add_special_tokens=False"]
    O1 --> O2["Count token length"]
    O2 --> O3{"length <= MAX_CONTEXT_WINDOW?"}
    O3 -->|Yes| O4["Keep sample"]
    O3 -->|No| O5["Remove sample"]

    O4 --> P["dataset.filter(..., num_proc=num_proc)"]

    P --> Q["check_assistant_format"]
    Q --> Q1["For every assistant message"]
    Q1 --> Q2["Must contain <think>"]
    Q1 --> Q3["Must contain </think>"]
    Q1 --> Q4["Must contain </think> followed by newline"]

    Q --> R["Create temporary check dataset<br/>with _ok column"]
    R --> S["bad = total - sum(_ok)"]

    S --> T{"bad > 0?"}
    T -->|Yes| U["Filter out bad-format samples"]
    T -->|No| V["Keep dataset as-is"]

    U --> W["Final training dataset ready"]
    V --> W

    W --> X["print(dataset[0]['text'][:8000])"]
    X --> X1["Human sanity check:<br/>verify role markers + think tags"]
```