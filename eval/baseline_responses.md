# Baseline Evaluation - Raw Model Responses

## Overview

This document contains the verbatim, unedited raw responses from the base Llama 3.2 model when prompted to extract structured JSON from 20 held-out business documents (Phase C baseline).

**Important**: Responses are copied exactly as returned by the model, including any markdown, prose, or formatting deviations. No editing.

---

## Document 1

**Source Document:**
```
[Raw invoice or PO text from evaluation set]
```

**Prompt Used:**
```
[Your extraction prompt here]
```

**Raw Model Response:**
```
[Verbatim response from model - may include markdown, prose, etc.]
```

**Parsing Attempt (python3 -c "import json; json.loads(...)"):**
- Valid JSON? YES / NO
- Error (if any): [error message]

---

## Document 2

**Source Document:**
```
[Raw document text]
```

**Prompt Used:**
```
[Your extraction prompt here]
```

**Raw Model Response:**
```
[Verbatim response]
```

**Parsing Attempt:**
- Valid JSON? YES / NO
- Error (if any): [error message]

---

## Document 3 through Document 20

[Continue pattern for all 20 evaluation documents...]

---

## Summary of Raw Response Characteristics

- **Total Responses**: 20
- **Valid JSON Responses**: [X]
- **Invalid JSON Responses**: [Y]
- **Responses with Code Fences**: [N]
- **Responses with Prose Preamble**: [N]
- **Average Response Length**: [chars]

---

**Baseline Responses Collected**: [DATE]
