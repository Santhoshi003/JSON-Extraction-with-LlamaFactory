# LoRA Fine-Tuning Configuration - Phase D

**Model**: Meta Llama 3.2 3B-Instruct  
**Method**: LoRA (Low-Rank Adaptation)  
**Framework**: LlamaFactory  
**Training Dataset**: 80 curated examples (data/curated_train.jsonl)  
**Evaluation Set**: 20 held-out examples (used for baseline in Phase C)  

---

## Hyperparameter Configuration

### LoRA Architecture

| Parameter | Value | Justification |
|-----------|-------|---------------|
| **LoRA Rank** | 16 | Small dataset (80 examples) requires low rank to avoid overfitting. Rank 16 = ~0.8M trainable params (vs 3.2B original) |
| **LoRA Alpha** | 32 | Standard 2×rank scaling factor. Provides stable gradient flow through LoRA update matrices |
| **Target Modules** | q_proj, v_proj, up_proj, down_proj | Attention and feed-forward weights. Sufficient for format learning; skips embedding layers |
| **LoRA Dropout** | 0.1 | Light regularization (10%). Prevents overfitting on small dataset while maintaining learning capacity |

### Training Hyperparameters

| Parameter | Value | Justification |
|-----------|-------|---------------|
| **Epochs** | 3 | Small dataset (80 examples) = 240 gradient updates per epoch. 3 epochs allows convergence without severe overfitting (~700 total steps) |
| **Batch Size** | 8 | Fits on consumer GPU (8GB VRAM). Larger batches (16+) reduce gradient noise but increase memory |
| **Learning Rate** | 2e-4 | Conservative LR for stable LoRA training. Too high (5e-4) causes instability; too low (1e-5) slow convergence |
| **Warmup Steps** | 10 | 10% of training budget (70 steps). Prevents wild gradient spikes early in training |
| **Weight Decay** | 0.01 | L2 regularization. Prevents overfitting on small dataset |
| **Gradient Accumulation** | 1 | Batch size 8 sufficient; no accumulation needed for memory/stability |

### LoRA Alpha (α)

**Choice**: [2 × rank] (e.g., if r=16, then α=32)

**Justification**:
- Standard practice in LoRA literature: α = 2 × r
- This scaling factor controls the magnitude of LoRA updates relative to frozen weights
- Empirically shown to balance update magnitude and stability
- [ANY PROJECT-SPECIFIC REASONING]

---

## Optimization Configuration

### Learning Rate

**Choice**: [1e-4 / 2e-4 / 3e-4 / other]

**Justification**:
- Typical LoRA range: 1e-4 to 3e-4
- Smaller LR (1e-4): More conservative, longer to converge, less risk of divergence
- Larger LR (3e-4): Faster convergence, risk of overshooting loss minima
- Dataset size trade-off: 80 examples is small; lower LR often safer
- [YOUR REASONING FOR CHOSEN VALUE]

### Epochs

**Choice**: [2 / 3 / 4 / 5]

**Justification**:
- Very small dataset (80 examples) → 2-3 epochs to prevent overfitting
- If training loss plateaus after epoch 1, stop at 2 epochs
- If still improving at epoch 2, continue to 3
- General rule: If loss is near-zero by epoch 1, overfitting likely → reduce epochs
- [YOUR CHOSEN STRATEGY]

### Batch Size

**Choice**: [8 / 16 / 32 / other]

**Justification**:
- Hardware available: [e.g., 12GB GPU, 16GB RAM]
- Max batch size fitting in memory: [tested value]
- Smaller batches (8): More gradient noise, can be beneficial for small datasets
- Larger batches (32): More stable gradients, but may underutilize your hardware
- [YOUR CHOSEN VALUE TRADE-OFF]

### Warmup Steps

**Choice**: [0 / 10 / 20 / percentage of total]

**Justification**:
- Small dataset may not need warmup
- Alternatively: warmup for 10-20% of total steps to stabilize early training
- [YOUR REASONING]

---

## Training Observations

### Loss Curve Analysis

[Paste or describe your loss curve here. Answer the following:]

1. **Did loss decrease smoothly throughout training?**
   - YES / NO
   - Description: [e.g., "Loss descended steadily from 2.1 to 0.85 over 3 epochs"]

2. **Was there any plateau or inflection point?**
   - At which epoch? [specify]
   - Interpretation: [Expected / Unexpected]

3. **Did loss drop suspiciously fast (potential overfitting)?**
   - By epoch 1: loss ~[value]
   - By epoch 2: loss ~[value]
   - Assessment: [Overfitting / Expected / Concerning]

4. **Final Training Loss**: [value]

### Screenshots

- **training_config.png**: Full configuration panel before training
- **loss_curve.png**: Loss curve graph from training completion

---

## Multi-Run Summary (If Applicable)

If you ran fine-tuning more than once (adjusting hyperparameters based on loss curves):

### Run 1
- **Date**: [date]
- **Configuration**: r=16, α=32, lr=2e-4, epochs=3, batch_size=16
- **Outcome**: [e.g., "Loss decreased to 0.85; no overfitting; acceptable convergence"]
- **Decision**: [Keep this config / Try different hyperparameters]

### Run 2
- **Date**: [date]
- **Configuration**: [new values]
- **Changes Made**: [Why did you adjust these hyperparameters?]
- **Outcome**: [Better / Worse / Similar]
- **Final Decision**: [Which run's weights will you use for evaluation?]

---

## Hardware & Environment

- **GPU/CPU Used**: [e.g., "NVIDIA RTX 3060 (12GB)", "CPU only (Intel i7)"]
- **Training Time**: [e.g., "45 minutes for 3 epochs"]
- **Peak Memory Usage**: [e.g., "8.2GB GPU, 4GB swap"]
- **LlamaFactory Version**: [e.g., "0.6.0"]
- **Date of Training**: [date]

---

## Risk Assessment

### Overfitting Risk: [LOW / MEDIUM / HIGH]

**Reasoning**:
- Dataset size: 80 examples is small (higher overfitting risk)
- Model size: 3B parameters; LoRA significantly reduces overfitting vs. full fine-tune
- Diversity: [Did your curation ensure varied layouts? Varied currencies?]
- Hyperparameters: [Does your config (rank, epochs) mitigate overfitting?]

**Mitigation Strategies Employed**:
- [e.g., "Low rank (r=8) to reduce model capacity"]
- [e.g., "2 epochs only to prevent memorization"]
- [e.g., "Data diversity: 40+ unique layouts, 5+ currencies"]

### Convergence: [CONFIDENT / UNCERTAIN]

**Reasoning**:
- [Did loss curve show healthy convergence?]
- [Any divergence or NaN loss observed?]

---

## Next Steps

- [ ] Evaluation: Load fine-tuned model in LlamaFactory inference tab
- [ ] Test on same 20 held-out documents from Phase C
- [ ] Compare parse success rate: baseline vs. fine-tuned
- [ ] Document results in eval/before_vs_after.md

---

**Training Completed**: [DATE]
**Configuration Finalized By**: [YOUR NAME]
