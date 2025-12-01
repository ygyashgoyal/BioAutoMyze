# BioAutoMyze: An AI Assistant for Biochemical Network Transcription

BioAutoMyze is a multimodal AI system that converts **biological network diagrams** into **structured mathematical models**.  
Given a pathway diagram (image), the system extracts:

- Species / nodes  
- Reactions and regulatory edges  
- Ordinary Differential Equations (ODEs)  
- Jacobian matrices  
- (Optionally) stability-related summaries

The project is built around a **LoRA-fine-tuned Pixtral-12B** multimodal model and deployed as an interactive web assistant.

> 🌐 Live demo: **http://bioautomyze.iiitd.edu.in/**  
> 📄 Project report: B.Tech thesis at IIIT-Delhi (Yash Goyal & Devansh Kumar, advised by Dr. Sriram K)

---

## 1. Motivation

Biochemical networks are often communicated via **diagrams**, not just textual reaction lists.  
Existing tools (PySB, BioNetGen, COPASI, etc.) take **textual reactions** as input and cannot directly consume diagrams.

This project aims to bridge that gap:

> **Diagram → (AI) → Reactions + ODEs + Jacobian**

using a fine-tuned multimodal LLM trained on synthetic and manually curated biochemical diagrams.

---

## 2. Key Features

- 🧬 **Diagram-to-model transcription**
  - Extract nodes, edges, reactions, ODEs, and Jacobians from network images.

- 🧠 **Multimodal fine-tuning of Pixtral-12B**
  - LoRA adapters on attention & MLP layers + multimodal projector.
  - Vision encoder frozen for efficiency.

- 🧪 **Domain-specific dataset**
  - ~240+ 1D biochemical network diagrams.
  - Each paired with: species, reactions, ODEs, Jacobian, and validity labels.
  - Includes **invalid** networks to train error detection.

- 🧯 **Error detection**
  - Can sometimes flag diagrams as **invalid** when they violate modeling rules  
    (e.g., inhibitory interactions from the source node \( \phi \)).

- 🌐 **Web deployment**
  - Accessible via browser; users upload diagrams and interact via a chatbot-style UI.

---

## 3. System Overview

### 3.1 High-Level Pipeline

1. **Input**: Diagram image \( I \) + textual prompt \( P \)  
2. **Model**: Fine-tuned Pixtral-12B with LoRA adapters  
3. **Output**: Structured transcription \( \hat{J} \) containing:
   - `nodes`, `edges`, `reactions`, `odes`, `jacobian`, `valid`, etc.

Mathematically:
\[
f_\phi(I, P) = \hat{J}
\]

where \( \phi \) are the LoRA parameters.

### 3.2 Architecture

- Base model: `mistral-community/pixtral-12b`
- Fine-tuning method: **LoRA / RS-LoRA**
  - Applied to:
    - `q_proj, k_proj, v_proj, o_proj`
    - `gate_proj, up_proj, down_proj`
  - Rank `r = 16`, alpha `= 32`, LoRA dropout `= 0.2`
- Vision encoder: **frozen**
- Multimodal projector: **trainable**

---

## 4. Dataset

### 4.1 Construction

- Networks are either:
  - **Programmatically generated** via Python, or  
  - **Manually drawn** to reflect realistic regulatory motifs.
- Variations include:
  - Presence/absence of cycles
  - Activation vs inhibition
  - Self-loops
  - Source/decay node \( \phi \)
  - Intentionally invalid topologies

### 4.2 Annotations

Each sample includes:

- Species list
- Reaction list (mass-action style)
- ODEs
- Jacobian matrix
- Validity flag / error description (for invalid networks)

### 4.3 JSONL Format (Pixtral-compatible)

Each line (sample) looks like:

```json
{
  "image": "/path/to/diagram.png",
  "messages": [
    {
      "role": "user",
      "content": [
        { "type": "text",  "text": "Analyze this network." },
        { "type": "image", "image": "/path/to/diagram.png" }
      ]
    },
    {
      "role": "assistant",
      "content": [
        {
          "type": "text",
          "text": "Reactions: ... ODEs: ... Jacobian: ..."
        }
      ]
    }
  ]
}
