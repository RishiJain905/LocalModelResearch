```mermaid
flowchart TB
    A["Start data pipeline"] --> B["Import dataset tools"]
    B --> B1["load_dataset"]
    B --> B2["concatenate_datasets"]
    B --> B3["Dataset"]
    B --> B4["get_chat_template"]
    B --> B5["re / json / multiprocessing / pandas"]

    A --> C["Set global config"]
    C --> C1["RANDOM_SEED = 12181531"]
    C --> C2["MAX_CONTEXT_WINDOW = 8192"]

    C1 --> C1A["Makes shuffle/sample reproducible"]
    C2 --> C2A["Later removes samples longer than 8192 tokens"]

    A --> D["Set sample counts"]
    D --> D1["ds1: 3900 requested"]
    D --> D2["ds2: 700 requested"]
    D --> D3["ds3: 9633 requested"]

    D1 --> D1A["Actual used = min(requested, len(dataset))"]
    D1A --> D1B["So if ds1 has fewer rows,<br/>it just uses all available rows"]

    A --> E["Set Qwen thinking template"]
    E --> E1["tokenizer = get_chat_template(tokenizer, chat_template='qwen3-thinking')"]
    E1 --> E2["Later apply_chat_template uses Qwen thinking format"]

    A --> F["Define ds3 fallback loader"]
    F --> F1["load_ds3_via_pandas_parquet()"]
    F1 --> F2["Reads HF parquet directly with pandas"]
    F2 --> F3["Converts pandas DataFrame → HF Dataset"]

    A --> G["Define load_and_sample(...)"]
    G --> G1["Try load_dataset(dataset_name, split='train')"]
    G1 --> G2{"Normal load works?"}

    G2 -->|Yes| H["Dataset loaded"]
    G2 -->|No, specific ds3 JSON issue| I["Use pandas parquet fallback"]
    G2 -->|Other error| J["Raise error"]

    H --> K["If sample_count provided"]
    I --> K

    K --> L["sample_count = min(sample_count, len(ds))"]
    L --> M["Shuffle with RANDOM_SEED"]
    M --> N["Select first sample_count rows after shuffle"]

    N --> O["Load ds1"]
    O --> O1["nohurry/Opus-4.6-Reasoning-3000x-filtered<br/>schema: problem/thinking/solution"]

    N --> P["Load ds2"]
    P --> P1["Jackrong/Qwen3.5-reasoning-700x<br/>schema: multi-turn conversation"]

    N --> Q["Load ds3"]
    Q --> Q1["Roman1111111/claude-opus-4.6-10000x<br/>schema: messages with possible reasoning fields"]

    O1 --> R["Raw datasets are loaded<br/>but still have different schemas"]
    P1 --> R
    Q1 --> R
```