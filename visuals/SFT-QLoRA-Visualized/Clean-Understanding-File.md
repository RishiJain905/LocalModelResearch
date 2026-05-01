```mermaid
flowchart LR
    A["Base Qwen3.5-27B<br/>frozen 4-bit model"] --> B["LoRA adapters inserted<br/>into target modules"]
    B --> C["SFT dataset<br/>Qwen template + think tags"]
    C --> D["Response-only training<br/>ignore user tokens"]
    D --> E["Train only LoRA weights"]
    E --> F["Save adapter<br/>small mod file"]
    E --> G["Merge model<br/>large standalone model"]
    E --> H["Export GGUF<br/>local inference"]
```