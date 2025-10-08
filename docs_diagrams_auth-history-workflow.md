# Authentication and History/Feedback Workflow

```mermaid
flowchart LR
  A[Open App] --> B{Authenticated?}
  B -- No --> C[Login (JWT) or Google OAuth 2.0]
  C --> D[Token issued\n(Session established)]
  B -- Yes --> E[Chat Screen]
  D --> E

  %% Chat actions from the main screen
  E --> F[Send Message]
  E --> G[Open History]
  E --> H[Submit Feedback]

  %% History retrieval
  G --> I[GET /chat/history]
  I --> J[(MongoDB)]

  %% Feedback submission
  H --> K[POST /feedback]
  K --> J

  %% Orchestration for chat message
  F --> L[POST /chat/message]
  L --> M[Auth check (JWT)]
  M --> N[Chat Orchestrator (Node/Express)]
  N --> O[Rasa NLU\n(intents, entities, confidence)]
  O --> P{Confident intent\n+ KB mapping?}

  P -- Yes --> Q[Fetch canonical answer]
  Q --> J
  Q --> R[Return response to UI]
  R --> S[Persist message + response]
  S --> J

  P -- No --> T[Gemini API (guarded augmentation)]
  T --> R
  R --> U[Persist message + response\n(augmented=true)]
  U --> J
```

Notes:
- Login supports JWT sessions and Google OAuth 2.0.
- History and feedback are persisted in MongoDB.
- Chat orchestration calls Rasa first; Gemini is used only on low-confidence or unmapped intents, with strict guardrails.