```mermaid
flowchart TB

    %% =========================================================
    %% HIGH LEVEL OVERVIEW
    %% =========================================================

    A["1. Choose Base Model<br/>Example: unsloth/Qwen3.5-27B"] --> B["2. Load Base Model into Memory<br/>FastLanguageModel.from_pretrained(...)"]
    B --> C["3. Model Architecture is Now Available<br/>Transformer blocks + internal layer names exist"]
    C --> D["4. Attach LoRA Adapters<br/>FastLanguageModel.get_peft_model(...)"]

    %% =========================================================
    %% WHY BASE MODEL MUST BE LOADED FIRST
    %% =========================================================

    subgraph WHY["Why the Base Model Must Be Loaded First"]
        W1["LoRA is NOT a standalone model"]
        W2["LoRA needs a real transformer architecture to attach to"]
        W3["PEFT/Unsloth searches the loaded model for target module names"]
        W4["Examples of target module names:<br/>q_proj, k_proj, v_proj, o_proj,<br/>gate_proj, up_proj, down_proj"]
        W1 --> W2 --> W3 --> W4
    end

    C --> WHY

    %% =========================================================
    %% TARGET MODULES / TRANSFORMER BLOCK
    %% =========================================================

    subgraph BLOCK["Inside the Loaded Transformer Block"]
        B1["Transformer Block"]
        B2["Self Attention"]
        B3["MLP / Feed Forward"]

        B1 --> B2
        B1 --> B3

        B2 --> Q["q_proj<br/>LoRA inserted here"]
        B2 --> K["k_proj<br/>LoRA inserted here"]
        B2 --> V["v_proj<br/>LoRA inserted here"]
        B2 --> O["o_proj<br/>LoRA inserted here"]

        B3 --> G["gate_proj<br/>LoRA inserted here"]
        B3 --> U["up_proj<br/>LoRA inserted here"]
        B3 --> DN["down_proj<br/>LoRA inserted here"]
    end

    D --> BLOCK

    %% =========================================================
    %% HOW LORA WORKS INSIDE ONE LINEAR LAYER
    %% =========================================================

    subgraph LORA_MATH["How LoRA Changes a Target Linear Layer"]
        M1["Original layer output<br/>y = W x"]
        M2["W = original base model weight"]
        M3["In LoRA, W stays frozen"]
        M4["LoRA adds small trainable matrices A and B"]
        M5["New effective output<br/>y = W x + scale * (B A x)"]
        M6["So the model output changes<br/>even though W never changes"]

        M1 --> M2 --> M3 --> M4 --> M5 --> M6
    end

    Q --> LORA_MATH
    K --> LORA_MATH
    V --> LORA_MATH
    O --> LORA_MATH
    G --> LORA_MATH
    U --> LORA_MATH
    DN --> LORA_MATH

    %% =========================================================
    %% FROZEN VS TRAINABLE
    %% =========================================================

    subgraph TRAINABLE["What is Frozen vs What is Trainable"]
        F1["Frozen:<br/>Original Qwen base weights W"]
        F2["Trainable:<br/>LoRA A and B matrices"]
        F3["Result:<br/>Most of the giant model stays unchanged"]
        F4["Only small adapter weights are updated"]
        F1 --> F3
        F2 --> F4
    end

    LORA_MATH --> TRAINABLE

    %% =========================================================
    %% TRAINING FLOW
    %% =========================================================

    subgraph TRAINING["Training Flow During LoRA / QLoRA SFT"]
        T1["Input text enters the model"]
        T2["Forward pass runs through frozen base model"]
        T3["At target modules, LoRA side-path modifies activations"]
        T4["Model produces predictions"]
        T5["Loss is computed"]
        T6["Gradients flow only into LoRA A/B weights"]
        T7["Optimizer updates only LoRA weights"]
        T8["Base model weights remain frozen the whole time"]

        T1 --> T2 --> T3 --> T4 --> T5 --> T6 --> T7 --> T8
    end

    TRAINABLE --> TRAINING

    %% =========================================================
    %% QLORA CONTEXT
    %% =========================================================

    subgraph QLORA["Where QLoRA Fits In"]
        Q1["Base model loaded in 4-bit"]
        Q2["This reduces VRAM usage"]
        Q3["LoRA adapters are attached on top"]
        Q4["So training setup becomes:<br/>4-bit base model + trainable LoRA adapters"]
        Q5["This is why it is commonly called QLoRA"]

        Q1 --> Q2 --> Q3 --> Q4 --> Q5
    end

    B --> QLORA
    D --> QLORA

    %% =========================================================
    %% WHY ADAPTER-ONLY SAVING WORKS
    %% =========================================================

    subgraph SAVE["Saving After Training"]
        S1["Option A: Save Adapter Only<br/>model.save_pretrained('qwen_lora')"]
        S2["Saved contents:<br/>only the learned LoRA weights"]
        S3["This is much smaller than a full 27B model"]
        S4["Useful for experiments and storage efficiency"]

        S5["Option B: Merge Model"]
        S6["Base model + LoRA adapter are combined into one full model"]
        S7["This creates a much larger standalone model"]
        S8["Useful for full deployment / direct Transformers loading"]

        S9["Option C: Export GGUF"]
        S10["Merged model can be exported to GGUF"]
        S11["Useful for llama.cpp / LM Studio / local runtimes"]

        S1 --> S2 --> S3 --> S4
        S5 --> S6 --> S7 --> S8
        S8 --> S9 --> S10 --> S11
    end

    TRAINING --> SAVE

    %% =========================================================
    %% HOW RELOADING WORKS
    %% =========================================================

    subgraph RELOAD["How Adapter Reloading Works Later"]
        R1["Load the SAME compatible base model again"]
        R2["Load the saved LoRA adapter"]
        R3["Attach adapter back onto the base model"]
        R4["Now inference uses:<br/>base model + trained LoRA updates"]
        R5["Important:<br/>Adapter must match the same architecture/module layout"]

        R1 --> R2 --> R3 --> R4 --> R5
    end

    S1 --> RELOAD

    %% =========================================================
    %% FINAL SUMMARY
    %% =========================================================

    subgraph SUMMARY["Mental Model Summary"]
        Z1["Base model = giant frozen brain"]
        Z2["Target modules = exact places LoRA is inserted"]
        Z3["LoRA = small trainable side weights A and B"]
        Z4["Training = update only LoRA, not the full base model"]
        Z5["Adapter save = save only your learned changes"]
        Z6["Merged save = combine base + adapter into one standalone model"]

        Z1 --> Z2 --> Z3 --> Z4 --> Z5 --> Z6
    end

    RELOAD --> SUMMARY
```