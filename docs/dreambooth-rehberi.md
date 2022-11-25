# Dreambooth Extension for Stable-Diffusion-WebUI

This is a WIP port of [Shivam Shriao's Diffusers Repo](https://github.com/ShivamShrirao/diffusers/tree/main/examples/dreambooth), which is a modified version of the default [Huggingface Diffusers Repo](https://github.com/huggingface/diffusers) optimized for better performance on lower-VRAM GPUs.

It also adds several other features, including training multiple concepts simultaneously, and (Coming soon) Inpainting training.

## Installation

To install, simply go to the "Extensions" tab in the SD Web UI, select the "Available" sub-tab, pick "Load from:" to load the list of extensions, and finally, click "install" next to the Dreambooth entry.

![image](https://user-images.githubusercontent.com/1633844/200368737-7fe322de-00d6-4b28-a321-5e09f072d397.png)

_For 8bit adam to run properly, it may be necessary to install the CU116 version of torch and torchvision, which can be accomplished below:_

Refer to the appropriate script below for extra flags to install requirements:

<https://github.com/d8ahazard/sd_dreambooth_extension/blob/main/webui-user-dreambooth.bat>
<https://github.com/d8ahazard/sd_dreambooth_extension/blob/main/webui-user-dreambooth.sh>

Setting the torch command to:
`TORCH_COMMAND=pip install torch==1.12.1+cu116 torchvision==0.13.1+cu116 --extra-index-url https://download.pytorch.org/whl/cu116`
will ensure that the proper torch version is installed when webui-user is executed, and then left alone after that, versus trying to install conflicting versions.

We also need a newer version of diffusers, as SD-WebUI uses version 0.3.0, while DB training requires > 0.6.0, so we use 0.7.2. Not having the right diffusers version is the cause of the 'UNet2DConditionModel' object has no attribute 'enable_gradient_checkpointing' error message, as well as safety checker warnings.

To force sd-web-ui to _only_ install one set of requirements, we can specify the command line argument:

set/export REQS_FILE=.\extensions\sd_dreambooth_extension\requirements.txt

And last, if you wish to completely skip the "native" install routine of Dreambooth, you can set the following environment flag:
DREAMBOOTH_SKIP_INSTALL=True

This is ideal for "offline mode", where you don't want the script to constantly check things from pypi.

After installing via the WebUI, it is recommended to set the above flags and re-launch the entire Stable-diffusion-webui, not just reload it.

## Usage

### Create a Model

1. Go to the Dreambooth tab.
2. Under the "Create Model" sub-tab, enter a new model name and select the source checkpoint to train from.
   The source checkpoint will be extracted to models\dreambooth\MODELNAME\working - the original will not be touched.
   2b. Optionally, you can also specify a huggingface model directory and token to create the Dreambooth dataset from huggingface.co.
   Model path format should be like so: 'runwayml/stable-diffusion-v1-5'
3. Click "Create". This will take a minute or two, but when done, the UI should indicate that a new model directory has been set up.

### Training (Basic Settings)

1. After creating a new model, select the new model name from the "Model" dropdown at the very top.
2. Select the "Train Model" sub-tab.
3. Fill in the paramters as described below:

_Concepts List_ - The path to a JSON file or a JSON string containing multiple concepts. See [here](https://raw.githubusercontent.com/d8ahazard/sd_dreambooth_extension/main/dreambooth/concepts_list.json) for an example.

If a concepts list is specified, then the instance prompt, class prompt, instance data dir, and class data dir fields will be ignored.

_Instance Prompt_ - A short descriptor of your subject using a UNIQUE keyword and a classifier word. If training a dog, your instance prompt could be "photo of zkz dog".
The key here is that "zkz" is not a word that might overlap with something in the real world "fluff", and "dog" is a generic word to describe your subject. This is only necessary if using prior preservation.
You can use `[filewords]` as placeholder for reading caption from the image filename or a seprarte .txt file containing caption, for example, `[filewords], in the style of zymkyr`. This syntax is the same as textual inversion templates.

_Class Prompt_ - A keyword indicating what type of "thing" your subject is. If your instance prompt is "photo of zkz dog", your class prompt would be "photo of a dog".
Leave this blank to disable prior preservation training.

_Dataset Directory_ - The path to the directory where the images described in Instance Prompt are kept. _REQUIRED_

_Classification dataset directory_ - The path to the directory where the images described in Class Prompt are kept. If a class prompt is specified and this is left blank,
images will be generated to /models/dreambooth/MODELNAME/classifiers/

_Total number of classification images to use_ - Leave at 0 to disable prior preservation. For best results you want ~n\*10 classification images - so if you have 40 training photos, then set this to 400. This is just a guess.

_Training steps_ - How many total training steps to complete. According to [this guide](https://github.com/nitrosocke/dreambooth-training-guide), you should train for appx 100 steps per sample image. So, if you have 40 instance/sample images, you would train for 4k steps. This is, of course, a rough approximation, and other values will have an effect on final output fidelity.

_Batch size_ - How many training steps to process simultaneously. You probably want to leave this at 1.

_Class batch size_ - How many classification images to generate simultaneously. Set this to whatever you can safely process at once using Txt2Image, or just leave it alone.

_Learning rate_ - You probably don't want to touch this.

_Resolution_ - The resolution to train images at. You probably want to keep this number at 512 or lower unless your GPU is insane. Lowering this (and the resolution of training images)
may help with lower-VRAM GPUs.

_Save a checkpoint every N steps_ - How frequently to save a checkpoint from the trained data. I should probably change the default of this to 1000.

_Generate a preview image every N steps_ - How frequently will an image be generated as an example of training progress.

_Preview image prompt_ - The prompt to use to generate preview image. Leave blank to use the instance prompt.

_Preview image negative prompt_ - Like above, but negative. Leave blank to do nothing. :P

_Number of samples to generate_ - Self explainatory?

_Sample guidance scale_ - Like CFG Scale in Txt2Image/Img2Img, used for generating preview.

_Sample steps_ - Same as sample guidance scale, but the number of steps to run to generate preview.

### Advanced Settings

_Use CPU Only_ - As indicated, this is more of a last resort if you can't get it to train with any other settings. Also, as indicated, it will be abysmally slow.
Also, you _cannot_ use 8Bit-Adam with CPU Training, or you'll have a bad time.

_Don't Cache Latents_ - Why is this not just called "cache" latents? Because that's what the original script uses, and I'm trying to maintain the ability to update this as easily as possible. Anyway...when this box is _checked_ latents will not be cached. When latents are not cached, you will save a bit of VRAM, but train slightly slower.

_Train Text Encoder_ - Not required, but recommended. Enabling this will probably cost a bit more VRAM, but also purportedly increase output image fidelity.

_Use 8Bit Adam_ - Enable this to save VRAM. Should now work on both windows and Linux without needing WSL.

_Center Crop_ - Crop images if they aren't the right dimensions? I don't use this, and I recommend you just crop your images "right".

_Gradient Checkpointing_ - Enable this to save VRAM at the cost of a bit of speed.

_Scale Learning Rate_ - I don't use this, not sure what impact it has on performance or output quality.

_Mixed Precision_ - Set to 'fp16' to save VRAM at the cost of speed.

_Everything after 'Mixed Precision'_ - Adjust at your own risk. Performance/quality benefits from changing these remain to be tested.

The next two were added after I wrote the above bit, so just ignore me being a big liar.

_Pad Tokens_ - Pads the text tokens to a longer length for some reason.

_Max Token Length_ - raise the tokenizer's default limit above 75. Requires Pad Tokens for > 75.

_Apply Horizontal Flip_ - "Apply horizontal flip augmentation". Flips images horizontally at random, which can potentially offer better editability?

_Use EMA for finetuning_ - Use exponential moving average weight to reduce overfitting during the last iterations.

### Continuing Training

Once a model has been trained for any number of steps, a config file is saved which contains all of the parameters from the UI.

If you wish to continue training a model, you can simply select the model name from the dropdown and then click the blue button next to the model name dropdown to load previous parameters.

![image](https://user-images.githubusercontent.com/1633844/200369076-8debef69-4b95-4341-83ac-cbbb02ee02f6.png)

## Use DreamBooth to Fine-Tune Stable Diffusion in Google Colab

### Prepare Images

#### Choosing Images

When choosing images, it’s recommended to keep the following in mind to get the best results:

- Upload a variety of images of your subject. If you’re uploading images of a person, try something like 70% close-ups, 20% from the chest up, 10% full body, so Stable Diffusion also gets some idea of the rest of the subject and not only the face.
- Try to change things up as much as possible in each picture. This means:
- Varying the body pose
- Taking pictures on different days, in different lighting conditions, and with different backgrounds
- Showing a variety of expressions and emotions
- When generating new images, whatever you capture will be over-represented. For example, if you take multiple pictures with the same green field behind you, it’s likely that the generated images of you will also contain the green field, even if you want a dystopic background. This can apply to anything, like jewelry, clothes, or even people in the background. If you want to avoid seeing that element in your generated image, make sure not to repeat it in every shot. On the other hand, if you want it in the generated images, make sure it’s in your pictures more often.
- It’s recommended that you provide ~50 images of what you’d like to train Stable Diffusion on to get great results. However, I’ve only used 20-30 so far, and the results are pretty good. If you’re just starting out and want to test it out, I think 20-30 images should be good enough for now, and you can get 50 images after you’ve seen it work.

#### Resize & Crop to 512 x 512px

Once you’ve chosen your images, you should prepare them.

First, we need to resize and crop our images to be 512 x 512px. We can easily do this using the website <https://birme.net>.

To do this, just:

- Visit the website
- Upload your images
- Set your dimensions to 512 x 512px
- Adjust the cropping area to center your subject
- Click on Save as Zip to download the archive.
- You can then unzip it on your computer, and we’ll use them a bit later.
- Birme.net - Resize Images
- Resizing Images using Birme.net

#### Renaming Your Images

We’ll also want to rename our images to contain the subject’s name:

1. Firstly, the subject name should be one unique/random/unknown keyword. This is because Stable Diffusion also has some knowledge of The Sandman from other sources other than the one played by Tom Sturridge and we don’t want it to get confused and make a combination of interpretations of The Sandman. As such, I’ll call it Sandman2022 to make sure it’s unique.

2. Renaming images to subject (1), subject (2) .. subject (30). This is because, using this method, you can train multiple subjects at once. If you want to fine-tune Stable Diffusion with Sandman, your friend Kevin, and your cat, you can give it prepare images for each of them. For the Sandman you’d have Sandman2022 (1), Sandman2022 (2) … Sandman (30), for Kevin you’d have KevinKevinson2022 (1), KevinKevinson2022 (2) … KevinKevinson (30), and for your cat you’d have DexterTheCat (1), DexterTheCat (2) … DexterTheCat(30).
   Here’s me renaming my images for Sandman2022 in bulk on Windows. Just select them all, right click one of them and click Rename and give it what name you want and click anywhere to finish the renaming. Everything else will be renamed as well.
