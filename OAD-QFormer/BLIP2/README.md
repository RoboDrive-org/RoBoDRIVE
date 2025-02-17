## BLIP-2: Guided Language-Image Pre-training with Frozen Image Encoders and Large Language Models

We use BLIP-2 as a preliminary implementation of OAD-QFormer. This is a general and efficient pre-training strategy that can easily harvest the development of pre-trained visual models and large language models (LLMs) for vision-language pre-training.


### Install:
```
pip install salesforce-lavis
```
or install from source following LAVIS instruction.

### Demo:
Try out our [Notebook Demo](https://github.com/salesforce/LAVIS/blob/main/examples/blip2_instructed_generation.ipynb) on instructed vision-to-language generation: [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/salesforce/LAVIS/blob/main/examples/blip2_instructed_generation.ipynb)


### BLIP-2 Model Zoo 
```python
# ==================================================
# Architectures                  Types
# ==================================================
# blip2_opt                      pretrain_opt2.7b, caption_coco_opt2.7b, pretrain_opt6.7b, caption_coco_opt6.7b
# blip2_t5                       pretrain_flant5xl, caption_coco_flant5xl, pretrain_flant5xxl
# blip2                          pretrain, coco
```
- Use ```pretrained_{LLM}``` model types for zero-shot image-to-text generation with prompts.
- Use ```caption_coco_{LLM}``` model types to generate coco-style captions.
- Use ```blip2``` model architecture for image-text feature extraction and retrieval.

### Image-to-text Generation Example
Let’s see how to use BLIP-2 models to perform zero-shot instructed image-to-text generation. We first load a sample image from local.
```python
import torch
from PIL import Image
# setup device to use
device = torch.device("cuda") if torch.cuda.is_available() else "cpu"
# load sample image
raw_image = Image.open("../../docs/_static/merlion.png").convert("RGB")
display(raw_image.resize((596, 437)))
```

Then we load a pre-trained BLIP-2 model with its preprocessors (transforms).
```python
import torch
from lavis.models import load_model_and_preprocess
# loads BLIP-2 pre-trained model
model, vis_processors, _ = load_model_and_preprocess(name="blip2_t5", model_type="pretrain_flant5xxl", is_eval=True, device=device)
# prepare the image
image = vis_processors["eval"](raw_image).unsqueeze(0).to(device)
```

Given the image and a text prompt, ask the model to generate the response.
```python
model.generate({"image": image, "prompt": "Question: which city is this? Answer:"})
# 'singapore'
```

Ask the model to explain its answer.
```python
model.generate({
    "image": image,
    "prompt": "Question: which city is this? Answer: singapore. Question: why?"})
# 'it has a statue of a merlion'    
```




Ask a follow-up question.
```python
# prepare context prompt
context = [
    ("which city is this?", "singapore"),
    ("why?", "it has a statue of a merlion"),
]
question = "where is the name merlion coming from?"
template = "Question: {} Answer: {}."
prompt = " ".join([template.format(context[i][0], context[i][1]) for i in range(len(context))]) + " Question: " + question + " Answer:"
print(prompt)
# generate model's response
model.generate({"image": image,"prompt": prompt})
# 'merlion is a portmanteau of mermaid and lion'
```

### Feature Extraction Example
BLIP-2 supports the Unified Feature Extraction Interface of LAVIS. Checkout this [notebook](https://github.com/salesforce/LAVIS/blob/3446bac20c5646d35ae383ebe6d13cec4f8b00cb/examples/blip2_feature_extraction.ipynb) for an example.

### Image-Text Matching Example
BLIP-2 can compute the image-text matching score using the same interface as BLIP. Checkout this [notebook](https://github.com/salesforce/LAVIS/blob/3446bac20c5646d35ae383ebe6d13cec4f8b00cb/examples/blip2_image_text_matching.ipynb) for an example. 

### Benchmark Evaluation 
Follow [Dataset Download](https://opensource.salesforce.com/LAVIS//latest/getting_started.html#auto-downloading-and-loading-datasets) to prepare common vision-language datasets.

Run [these scripts](https://github.com/salesforce/LAVIS/tree/main/run_scripts/blip2/eval) for evaluating pretrained and finetuned models. 

### Training
Stage-1 Pre-training (from scratch): 
```bash run_scripts/blip2/train/pretrain_stage1.sh```

Stage-2 Pre-training: 
```bash run_scripts/blip2/train/pretrain_stage2.sh```

Finetune for image captioning: 
```bash run_scripts/blip2/train/train_caption_coco.sh```

The [config files](https://github.com/salesforce/LAVIS/tree/main/lavis/projects/blip2/train) can be modified for customized training.
