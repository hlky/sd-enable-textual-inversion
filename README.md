# sd-enable-textual-inversion
Copy the files into your stable-diffusion folder to enable text-inversion in the Web UI.

If you experience any problems **with webui integration**, please create an issue [here](https://github.com/hlky/stable-diffusion-webui).

If you are having problems installing or running textual-inversion, see FAQ below. If your problem is not listed, the official repo is [here](https://github.com/rinongal/textual_inversion).


# How does it work?

[textual-inversion](https://textual-inversion.github.io/) - An Image is Worth One Word: Personalizing Text-to-Image Generation using Textual Inversion (credit: Tel Aviv University, NVIDIA)

![](https://textual-inversion.github.io/static/images/editing/teaser.JPG)

Just as humans create unique words to describe what they see, this model learns to describe specific concepts, such as personal objects or artistic styles, and create a new "word" to represent that description. The model's description of that word is stored in an embedding file, which can then be referenced by other pre-trained text-to-image models. This process has been coined as "Textual Inversion."

![](https://textual-inversion.github.io/static/images/editing/puppet.JPG)


# How to Use

Before you can do anything with the webui, you must first create embedding files by training the Textual-Inversion model. You will need 3-5 images of what you want the model to describe. You can use more images, but the paper recommends 5. For the best results, the images should be visually similar, and each image should be cropped to 512x512. Any other sizes will be rescaled (stretched).

**Step 1:** Place 3-5 images of the object/art style/scene/etc. into an empty folder

**Step 2:** In Anaconda run 
`python main.py 
  --base configs/latent-diffusion/txt2img-1p4B-finetune.yaml 
  -t 
  --actual_resume models/ldm/text2img-large/model.ckpt
  -n <name this run>
  --data_root path/to/image/folder
  --gpus 1`

**Step 3:** The model will continue to train until you stop it by entering CTRL+C. The recommended training time is 3000-7000 global steps. You can see what step the run is on in the progress bar. You can also monitor progress by reviewing the images at `logs/<your run name>/images`. I recommend sorting that folder by date modified, they tend to get jumbled up otherwise. 

**Step 4:** Once stopped, you will find a number of embedding files under `logs/<your run name>/checkpoints`. The one you want is **embeddings.pt**.

**Step 5:** In the WebUI, upload the embedding file you just created. Now, when writing a prompt, you can use `\*` to reference the object/art style/etc. 

Training data is the images of the thing you want to train on, apparently you only need 3-5 but best practices for choosing the images themselves are currently unknown afaik

One thing I have noticed when I tried training is that it resizes the image to 512 without respecting aspect ratio so you may want to 'outcrop' your image to a square, it appears backgrounds are 'filtered' out somehow and the central concept of the image picked up on

2 :cherries: examples :cherries: based on the embeddings in the sd-embeddings repo below:

![00254](https://user-images.githubusercontent.com/106811348/187011731-e0b0b48a-63c7-4ecc-81e6-104d1cb1e342.png)

![00184](https://user-images.githubusercontent.com/106811348/187011743-a4abd08e-2383-4207-95f5-c60f6d3183ba.png)

For training see:

[rinongal/textual_inversion](https://github.com/rinongal/textual_inversion)

[nicolai256/Stable-textual-inversion_win](https://github.com/nicolai256/Stable-textual-inversion_win)

Please note: training uses a HUGE amount of vram, you will struggle to get it working on <12gb vram, I have but only by changing the size: 512 to size: 448 in this section of v1-finetune or lowmemory version: 
```
data:
  target: main.DataModuleFromConfig
```

Alternatively you can test with an embedding from [here](https://github.com/hlky/sd-embeddings)
There are 2 available atm, see the readme.md of each one for more info i.e. amount of images in training dataset, steps trained
Note: they are not very good, more and better embeddings needed for the [sd-embeddings](https://github.com/hlky/sd-embeddings) repo

You use * in the prompt where you want the custom trained thing. Note: requires PHD in prompt crafting and messing around with sampler, steps, cfg, the prompt itself, to get anything resembling a good result (possibly caused by rubbish images used for training as well)
