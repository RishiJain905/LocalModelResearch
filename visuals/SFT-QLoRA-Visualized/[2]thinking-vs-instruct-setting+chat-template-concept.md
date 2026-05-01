```mermaid
flowchart TB
    A["Qwen Model Usage Modes"] --> B["Instruct Mode"]
    A --> C["Thinking Mode"]

    B --> B1["Direct instruction-following"]
    B --> B2["Often shorter/direct answers"]
    B --> B3["Typical sampling:<br/>temperature ~0.7<br/>top_p ~0.80<br/>top_k 20"]

    C --> C1["Reasoning-oriented output"]
    C --> C2["May use explicit think tags"]
    C --> C3["Output shape:<br/><think>reasoning</think><br/>final answer"]
    C --> C4["Typical sampling:<br/>temperature 0.6/0.7/1<br/>top_p ~0.95<br/>top_k 20"]

    C4 --> D["Sampling settings affect inference/testing<br/>NOT the actual training data"]
    B3 --> D

    D --> E["Training quality depends more on:<br/>dataset quality<br/>chat template<br/>LoRA config<br/>learning rate<br/>batch size<br/>training steps"]

    E --> F["Template First Rule"]
    F --> G["Pick model first"]
    G --> H["Use that model's native chat template"]

    H --> I["For Qwen thinking:<br/><|im_start|>user<br/>...<br/><|im_end|><br/><|im_start|>assistant<br/><think>...</think><br/>answer<br/><|im_end|>"]

    I --> J["Why it matters"]
    J --> K["Correct role boundaries"]
    J --> L["Correct assistant-start marker"]
    J --> M["Correct think-tag format"]
    J --> N["Stable SFT loss and better behavior"]

    N --> O["Later code uses:<br/>get_chat_template(..., chat_template='qwen3-thinking')"]
```