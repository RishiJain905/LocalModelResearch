```mermaid
flowchart TB
    A["Raw datasets loaded"] --> B["Problem:<br/>Each dataset has different schema"]

    B --> C["Goal:<br/>Convert all into common conversations format"]
    C --> C1["Final shape:<br/>[{role: user, content: ...},<br/>{role: assistant, content: <think>...</think> answer}]"]

    C --> D["_strip(x)"]
    D --> D1["Safely handles None"]
    D --> D2["Removes leading/trailing whitespace"]

    C --> E["THINK_BLOCK_RE"]
    E --> E1["Regex finds:<br/><think>...</think>"]
    E1 --> E2["DOTALL allows multi-line reasoning"]

    C --> F["normalize_assistant_to_think_solution(text)"]
    F --> F1{"Assistant text empty?"}
    F1 -->|Yes| F2["Return:<br/><think></think>"]
    F1 -->|No| F3{"Already has think block?"}
    F3 -->|Yes| F4["Keep think block<br/>put final answer after newline"]
    F3 -->|No| F5["Add empty think block:<br/><think></think><br/>answer"]

    C --> G["build_assistant_with_reasoning(content, reasoning)"]
    G --> G1{"Content already has think tags?"}
    G1 -->|Yes| G2["Normalize existing think block"]
    G1 -->|No| G3{"Separate reasoning field exists?"}
    G3 -->|Yes| G4["Build:<br/><think>reasoning</think><br/>content"]
    G3 -->|No| G5["Use normalize_assistant_to_think_solution(content)"]

    C --> H["parse_message_item(m)"]
    H --> H1{"m is dict?"}
    H1 -->|Yes| H2["Return dict"]
    H1 -->|No| H3{"m is string?"}
    H3 -->|Yes| H4["Try json.loads(m)"]
    H4 --> H5{"Valid dict?"}
    H5 -->|Yes| H6["Return parsed dict"]
    H5 -->|No| H7["Return None"]
    H3 -->|No| H7

    C --> I["format_ds1"]
    I --> I1["Input columns:<br/>problem / thinking / solution"]
    I1 --> I2["user = problem"]
    I2 --> I3["assistant = <think>thinking</think><br/>solution"]
    I3 --> I4["Output conversations"]

    C --> J["format_ds2"]
    J --> J1["Input:<br/>conversation list"]
    J1 --> J2["human → user"]
    J1 --> J3["gpt → assistant"]
    J3 --> J4["Normalize assistant think format"]
    J2 --> J5["Keep only valid convos ending in assistant"]
    J4 --> J5

    C --> K["format_ds3"]
    K --> K1["Input:<br/>messages list"]
    K1 --> K2["Parse dict or JSON-string messages"]
    K2 --> K3["Remove system messages"]
    K3 --> K4["Require final message = assistant"]
    K4 --> K5["For assistant:<br/>combine reasoning + content"]
    K5 --> K6["Keep user/assistant only"]
    K6 --> K7["Output conversations"]

    I4 --> L["ds1.map(format_ds1, remove original columns)"]
    J5 --> M["ds2.map(format_ds2, remove original columns)"]
    K7 --> N["ds3.map(format_ds3, remove original columns)"]

    L --> O["All datasets now share same schema:<br/>conversations"]
    M --> O
    N --> O
```