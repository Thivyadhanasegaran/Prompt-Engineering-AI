# Fine-Tuning-a-Large-Language-Model-flan-t5-large

# 💡 Fine-Tuning Flan-T5-Large for Java Programming Q&A

This project demonstrates how to fine-tune `google/flan-t5-large` using Parameter-Efficient Fine-Tuning (PEFT) with LoRA on a custom Java question-answering dataset. The final model powers an AI assistant that answers Java-related programming questions using a Gradio interface.

---

## ▶️ Live Demo

   [Watch Live Demo](https://drive.google.com/file/d/1RBqae3tb65vTit41mbOgZQRVIW1z5I6z/view?usp=drive_link)


## 📂 Project Structure

```
fine-tuning-java-qa-llm/
├── code/
│   └── Fine-Tuning a Large Language Model_Final_Code.md    # Full pipeline with detailed comments
├── dataset/
│   └── java_qa_dataset.json                      # Custom instruction–output style dataset
├── Fine-Tuning a Large Language Model_Requirements        # Dependencies to reproduce environment
├── README.md
├── Assignment 5_Fine-Tuning a Large Language Model_Report.pdf
├── Assignment 5-Fine-Tuning a Large Language Model_Presentation.mp4
```

---

## 🚀 Model Overview

- **Base Model**: `google/flan-t5-large`
- **Fine-Tuning Method**: LoRA (PEFT)
- **Training Platform**: Google Colab (T4 GPU)
- **Quantization**: 4-bit with `bitsandbytes`
- **Dataset**: 1000+ Java programming Q&A pairs
- **Interface**: Gradio-powered web app

---

## 📌 Key Features

- ✅ Efficient training using LoRA adapters (only `q`, `v` projections updated)
- ✅ Custom Java-specific Q&A dataset with clean formatting
- ✅ Real-time interactive UI using Gradio
- ✅ Evaluation using BLEU and ROUGE scores
- ✅ Demonstrates great alignment with domain-specific knowledge

---

## ⚙️ Setup Instructions

### 1. Install dependencies

Install via pip (Colab or local):

```bash
pip install -r Fine-Tuning a Large Language Model_Requirements.txt
```

### 2. Launch notebook

Open the `Fine-Tuning a Large Language Model_Final_Code.md` in Jupyter or Colab and run all cells in order.

---

## 3.📊 Evaluation Results

| Metric  | Score  |
| ------- | ------ |
| BLEU    | 0.4548 |
| ROUGE-1 | 0.6545 |
| ROUGE-2 | 0.5847 |
| ROUGE-L | 0.6545 |

These metrics indicate strong performance in capturing key phrases, structure, and fluency in Java Q&A.

---

## 4.🧪Try the Model (Gradio)

You can run the Gradio interface with:

```python
demo.launch(share=True)
```

Example question:

> "What is method overloading in Java?"

Expected output:

> "Method overloading allows multiple methods with the same name but different parameters within the same class."

---

## 📁 Requirements

See `Fine-Tuning a Large Language Model_Requirements.txt` for full environment details.

- transformers
- datasets
- accelerate
- peft
- bitsandbytes
- evaluate
- rouge_score
- gradio
- torch

---

## 📈 Lessons Learned

- LoRA significantly reduces GPU memory requirements and speeds up training.
- Instruction-style datasets help models generalize better in Q&A tasks.
- Gradio makes it easy to turn models into usable applications quickly.

---

## 📚 References

- [Hugging Face Transformers](https://huggingface.co/docs/transformers)
- [Flan-T5 Paper](https://arxiv.org/abs/2210.11416)
- [LoRA: Low-Rank Adaptation](https://arxiv.org/abs/2106.09685)
- [Gradio](https://gradio.app)
- [BLEU & ROUGE via Evaluate](https://huggingface.co/docs/evaluate)

---

## 👩‍💻 Author

**Thivya Dhanasegaran**
