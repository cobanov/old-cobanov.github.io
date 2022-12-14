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

# Textual Inversion vs Hypernetworks
Textual Inversion and Hyper Network have different driving principles, and Textual Inversion has much smaller capacity of learning results than Hyper Network.

Textual Inversion has a slower learning speed than Hyper Network, so it is more suitable for learning specific objects, characters, features, etc. than abstract things such as patterns and painting styles. Also, in order to memorize the pattern and painting style, it is necessary to prepare data that has been unified to some extent, such as coloring and color usage, so it is more difficult to prepare learning data than Hyper Network.

Also, Hyper Network can only embed one at a time, but even so, it is OK if you prepare a large amount of data using various patterns of composition, materials, and techniques and let it learn. Therefore, it can be said that Hyper Network is more suitable for improving the accuracy of illustrations. However, textual inversion is easier to handle if you want to remember specific patterns and characteristics.

## Training Hypernetworks
hyper network layer structure
If write "1, 2, 1", hypernetworks are composed of 2 fully connected layers whose intermediate dim is 2x, which is same as up to now.
The more you add the number, like "1, 2, 4, 2, 1", the more the structure of hypernetworks becomes deeper. Deep hypernetworks are suited for training with large datasets.

Add layer normalization
If checked, add layer normalization after every fully connected layer. It would be meaningful to prevent hypernetworks from overfitting and make training more stable.

https://rentry.org/sd-e621-textual-inversion