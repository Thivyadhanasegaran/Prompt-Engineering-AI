### ✅ Install Required Libraries

!pip install transformers datasets accelerate peft bitsandbytes evaluate rouge_score gradio --upgrade

### ✅ Load and Configure the Base Model with Quantization and LoRA

from transformers import AutoModelForSeq2SeqLM, AutoTokenizer, BitsAndBytesConfig
from peft import LoraConfig, get_peft_model, prepare_model_for_kbit_training

model_name = "google/flan-t5-large"

### Quantization config for efficient 4-bit training

bnb_config = BitsAndBytesConfig(
load_in_4bit=True,
bnb_4bit_use_double_quant=True,
bnb_4bit_quant_type="nf4",
bnb_4bit_compute_dtype="float16"
)

### Load tokenizer and model

tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForSeq2SeqLM.from_pretrained(
model_name,
quantization_config=bnb_config,
device_map="auto"
)

### Prepare model for k-bit training

model = prepare_model_for_kbit_training(model)

### LoRA config: inject trainable adapters into query and value projection layers

lora_config = LoraConfig(
r=16,
lora_alpha=32,
target_modules=["q", "v"],
lora_dropout=0.05,
bias="none",
task_type="SEQ_2_SEQ_LM"
)

model = get_peft_model(model, lora_config)

### ✅ Upload Dataset from Local Files

from google.colab import files
uploaded = files.upload()

### ✅ Define Preprocessing Function

def preprocess(batch):
input_texts = ["Java Question: " + q for q in batch["instruction"]]
target_texts = batch["output"]
return tokenizer(
input_texts,
text_target=target_texts,
truncation=True,
padding="max_length",
max_length=256,
)

### ✅ Load and Split the Dataset

from datasets import load_dataset
dataset = load_dataset("json", data_files="java_qa_dataset.json")["train"]

### 80-10-10 split for train, eval, and test

train_test = dataset.train_test_split(test_size=0.2, seed=42)
eval_test = train_test["test"].train_test_split(test_size=0.5, seed=42)

train_data = train_test["train"]
eval_data = eval_test["train"]
test_data = eval_test["test"]

### ✅ Tokenize Dataset Splits

tokenized_train = train_data.map(preprocess, batched=True)
tokenized_eval = eval_data.map(preprocess, batched=True)
tokenized_test = test_data.map(preprocess, batched=True)

### ✅ Define Training Configuration and Launch Fine-Tuning

from transformers import TrainingArguments, Trainer

training_args = TrainingArguments(
output_dir="./flan-t5-lora-java-results",
per_device_train_batch_size=2,
per_device_eval_batch_size=2,
learning_rate=2e-4,
num_train_epochs=5,
logging_dir="./logs",
logging_steps=10,
save_strategy="epoch",
report_to="none"
)

trainer = Trainer(
model=model,
args=training_args,
train_dataset=tokenized_train,
eval_dataset=tokenized_eval,
tokenizer=tokenizer
)

trainer.train()
model.save_pretrained("./flan-t5-lora-java-results")

### ✅ Run Sample Predictions

from transformers import AutoTokenizer
import torch

device = "cuda" if torch.cuda.is_available() else "cpu"
model.to(device)
model.eval()

java_questions = [
"What is the difference between final, finally and finalize in Java?",
"What is method overloading in Java?",
"What is garbage collection?",
"Explain static vs instance variables.",
"What is the use of final keyword?"
]

for question in java_questions:
input_text = "Java Question: " + question
inputs = tokenizer(input_text, return_tensors="pt").to(device)

    with torch.no_grad():
        outputs = model.generate(
            **inputs,
            max_new_tokens=100,
            do_sample=False,
            temperature=0.7,
            repetition_penalty=1.2
        )

    answer = tokenizer.decode(outputs[0], skip_special_tokens=True)
    print(f"\n🧠 Q: {question}\n✅ A: {answer}")

### ✅ Evaluate Performance with BLEU and ROUGE

import evaluate
bleu = evaluate.load("bleu")
rouge = evaluate.load("rouge")

sample_questions = [
"What is a constructor in Java?",
"What is method overloading in Java?",
"What is garbage collection?",
]

reference_answers = [
"A constructor is a special method used to initialize objects in Java.",
"Method overloading is when multiple methods have the same name but different parameters.",
"Garbage collection is the process of automatically reclaiming memory used by unreferenced objects."
]

predicted_answers = []
for q in sample_questions:
inputs = tokenizer("Java Question: " + q, return_tensors="pt").to("cuda")
with torch.no_grad():
outputs = model.generate(\*\*inputs, max_new_tokens=100)
answer = tokenizer.decode(outputs[0], skip_special_tokens=True)
predicted_answers.append(answer)

### Compute metrics

bleu_score = bleu.compute(predictions=predicted_answers, references=[[r] for r in reference_answers])
rouge_score = rouge.compute(predictions=predicted_answers, references=reference_answers)

print("\n📊 Evaluation Metrics:")
print(f"BLEU Score : {bleu_score['bleu']:.4f}")
print(f"ROUGE-1 : {rouge_score['rouge1']:.4f}")
print(f"ROUGE-2 : {rouge_score['rouge2']:.4f}")
print(f"ROUGE-L : {rouge_score['rougeL']:.4f}")

### ✅ Deploy as Gradio App

import gradio as gr

def answer_java_question(question):
prompt = "Answer this Java programming question clearly:\nJava Question: " + question
inputs = tokenizer(prompt, return_tensors="pt").to("cuda")

    with torch.no_grad():
        outputs = model.generate(
            **inputs,
            max_new_tokens=128,
            do_sample=True,
            temperature=0.8,
            top_k=50,
            top_p=0.95,
            repetition_penalty=1.2
        )

    return tokenizer.decode(outputs[0], skip_special_tokens=True)

### Setup Gradio Interface

theme = gr.themes.Default(
primary_hue="indigo",
secondary_hue="blue"
).set(
body_background_fill="###f8fafc",
container_radius="xl",
input_background_fill="white",
input_border_color="###cbd5e1",
input_border_width="1px",
button_primary_background_fill="indigo",
block_title_text_weight="bold",
block_title_text_size="xl"
)

with gr.Blocks(theme=theme, title="Java Programming Q&A Bot", css="""
###main-container { max-width: 800px; margin: auto; }
###footer {
margin-top: 2em;
color: ###94a3b8;
font-style: italic;
text-align: center;
font-size: 0.9em;
}
""") as demo:
gr.Image("https://upload.wikimedia.org/wikipedia/en/3/30/Java_programming_language_logo.svg", width=80)
gr.Markdown("###### 💡 Java Programming Q&A Bot")
gr.Markdown("🔍 Ask any **Java-related** programming question and get accurate, AI-powered answers instantly!")

    with gr.Row():
        with gr.Column(scale=3):
            input_box = gr.Textbox(lines=3, placeholder="Enter your Java programming question here...", label="❓ Your Java Question")
            submit_btn = gr.Button("Get Answer 💬")

        with gr.Column(scale=2):
            output_box = gr.Textbox(label="✅ AI Response", lines=6, interactive=False)

    gr.Examples(
        examples=[
            "What is a constructor in Java?",
            "What is the difference between abstract class and interface?",
            "How does garbage collection work in Java?",
            "What is the use of 'final' keyword in Java?",
            "Explain static vs instance variables in Java."
        ],
        inputs=[input_box]
    )

    gr.Markdown("💬 Powered by `flan-t5-large` fine-tuned with LoRA · Built with ❤️ by Thivya Dhanasegaran", elem_id="footer")

    submit_btn.click(fn=answer_java_question, inputs=input_box, outputs=output_box, show_progress="full")

demo.launch(share=True)
