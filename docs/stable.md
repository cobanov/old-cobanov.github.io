# Cobanov's Stable Diffusion Notes

## Type's of Fine-tuning

> Source: [reddit post](https://www.reddit.com/r/StableDiffusion/comments/y1eec5/automatic1111_just_added_support_for_hypernetwork/)



### 1. Textual Inversion

Textual Inversion - trains a word with one or more vectors that approximate your image. So if it is something it already has seen lots of examples of, it might have the concept and just need to 'point' at it. It is just expanding the vocabulary of model but all information it uses is already in the model.

---

### 2. Dreambooth

Dreambooth - this is essentially model fine tuning, which changes the weights of the main model. Dreambooth differs from typical fine tuning in that in tries to keep from forgetting/overwriting adjacent concepts during the tuning.

---

### 3. Hypernetwork

Hypernetworks - this is basically an adaptive head - it takes information from late in the model but injects information from the prompt 'skipping' the rest of the model. So it is similar to fine tuning the last 2 layers of a model but it gets much more signal from the prompt (it is taking the clip embedding of the prompt right before the output layer).

#### Discussions:

- [Hypernetwork training topic ](https://github.com/AUTOMATIC1111/stable-diffusion-webui/discussions/2284)

## Links

- [Stable Diffusion Models](https://rentry.org/sdmodels)
