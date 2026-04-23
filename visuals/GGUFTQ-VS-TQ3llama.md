flowchart TD
    A[You have a model file] --> B{What kind of model format is it?}

    B --> C[Standard GGUF weight format]
    B --> D[TurboQuant weight format]

    C --> C1[Examples:<br>Q8_0<br>Q4_K<br>Q5_K_M<br>F16]
    D --> D1[Examples:<br>TQ3_1S<br>TQ3_4S]

    C --> E[Use Backend 1]
    E --> E1[llama.cpp-turboquant-hip]
    E --> E2[Purpose:<br>Run normal GGUF models]
    E --> E3[Bonus:<br>Use TurboQuant KV cache at runtime]
    E --> E4[Cache options:<br>turbo2 / turbo3 / turbo4]

    D --> F[Use Backend 2]
    F --> F1[llama.cpp-hip-turboquant-tq3]
    F --> F2[Purpose:<br>Run TQ3 weight-format models]
    F --> F3[Supports TQ3_1S / TQ3_4S weights]
    F --> F4[KV cache world:<br>TQ3_0]

    E --> G[Example]
    G --> G1[Qwopus3.5-9B-v3.Q8_0.gguf]
    G --> G2[Works on Backend 1]
    G --> G3[Can use turbo4 KV cache]
    G --> G4[Weights stay Q8_0]

    F --> H[Example]
    H --> H1[Qwen3.6-27B-TQ3_4S.gguf]
    H --> H2[Needs Backend 2]
    H --> H3[Because weights themselves are TQ3_4S]

    I[Key idea] --> I1[Model weight format and KV cache format are different]
    I --> I2[Backend 1:<br>normal weights + Turbo KV cache]
    I --> I3[Backend 2:<br>TurboQuant weights]

    J[Simple rule] --> J1[If model name says Q8_0 / Q4_K / Q5_K_M -> Backend 1]
    J --> J2[If model name says TQ3_1S / TQ3_4S -> Backend 2]