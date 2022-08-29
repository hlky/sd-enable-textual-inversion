# sd-enable-textual-inversion
Copy the files into your stable-diffusion folder to enable text-inversion in the Web UI.

If you experience any problems **with webui integration**, please create an issue [here](https://github.com/hlky/stable-diffusion-webui).

If you are having problems installing or running textual-inversion, see FAQ below. If your problem is not listed, the official repo is [here](https://github.com/rinongal/textual_inversion).

<br>

# How does it work?

[textual-inversion](https://textual-inversion.github.io/) - An Image is Worth One Word: Personalizing Text-to-Image Generation using Textual Inversion (credit: Tel Aviv University, NVIDIA)

![](https://textual-inversion.github.io/static/images/editing/teaser.JPG)

> We learn to generate specific concepts, like personal objects or artistic styles, by describing them using new "words" in the embedding space of pre-trained text-to-image models. These can be used in new sentences, just like any other word. 

Essentially, this model will take some pictures of an object, style, etc. and learn how to describe it in a way that can be understood by text-to-image models such as Stable Diffusion. This allows you to reference specific things in your prompts, or concepts that are easier to express with pictures rather than words.

<br>

# How to Use

Before you can do anything with the WebUI, you must first create an embedding file by training the Textual-Inversion model. Alternatively, you can test with one of the pre-made embeddings from [here](https://github.com/hlky/sd-embeddings). 

In the WebUI, place the embeddings file in the embeddings file upload box. Then you can reference the embedding by using `*` in your prompt. 

Examples from the paper:
![](https://textual-inversion.github.io/static/images/editing/puppet.JPG)

<br>

# How to Train Textual-Inversion

** Training is best done by using the [original repo](https://github.com/rinongal/textual_inversion) **

> **WARNING:** This is a very memory-intensive model and, as of writing, is not optimized to work with SD. You will need an Nvidia GPU with at least 10GB of VRAM to even get this to train at all on your local device, and a GPU with 20GB+ to train in a reasonable amount of time. If you do not have the system resources, you should use Colab or stick with pretrained embeddings until SD is better supported.

**Note that these instructions are for training on your local device, instructions may vary for training in Colab.**

You will need 3-5 images of what you want the model to describe. You can use more images, but the paper recommends 5. For the best results, the images should be visually similar, and each image should be cropped to 512x512. Any other sizes will be rescaled (stretched) and may produce strange results.

<br>

### **Step 1:** 

Place 3-5 images of the object/artstyle/scene/etc. into an empty folder.


### **Step 2:** In Anaconda run 

`python main.py 
  --base configs/stable-diffusion/v1-finetune.yaml 
  -t 
  --actual_resume models/ldm/text2img-large/model.ckpt
  -n <name this run>
  --data_root path/to/image/folder
  --gpus 1
  --init-word <your init word>`
  
  > --base points the script at the training configuration file
  
  > --actual_resume points the script at the Textual-Inversion model

  > --n gives the training run a name, which will also be used as the output folder name.
  
  > --gpus Leave at 1 unless you know what you're doing.
  
  > --init-word is a single word the model will start with when looking at your images for the first time. Should be simple, ie: "sculpture", "girl", "mountains"


### **Step 3:** 

The model will continue to train until you stop it by entering CTRL+C. The recommended training time is 3000-7000 iterations (global steps). You can see what step the run is on in the progress bar. You can also monitor progress by reviewing the images at `logs/<your run name>/images`. I recommend sorting that folder by date modified, they tend to get jumbled up otherwise.


### **Step 4:** 

Once stopped, you will find a number of embedding files under `logs/<your run name>/checkpoints`. The one you want is **embeddings.pt**.


### **Step 5:** 

In the WebUI, upload the embedding file you just created. Now, when writing a prompt, you can use `*` to reference the whatever the embedding file describes.

> "A picture of * in the style of Rembrandt"

> "A photo of * as a corgi"

> "A coffee mug in the style of *"

<br>

# Pro tips

Reminder: Official Repo here ==> [rinongal/textual_inversion](https://github.com/rinongal/textual_inversion)

Unofficial fork, more stable on Windows (8/28/22) ==> [nicolai256/Stable-textual-inversion_win](https://github.com/nicolai256/Stable-textual-inversion_win)

- When using embeddings in your prompts, the authors note that markers (`*`) are sensitive to puncutation. Avoid using periods or commas directly after `*`
- The model tends to converge faster and provide more accurate results when using language such as "a photo of" or "* as a photograph" in your prompts
- When training Textual-Inversion, the paper says that using more than 5 images leads to less cohesive results. Some users seem to disagree. Try experimenting.
- When training, more than one init-word can be specified by adding them to the list at `initializer_words: ["sculpture", "ice"]` in `v1-finetune.yaml`. Order may matter (unconfirmed)
- You can train multiple embedding files, then merge them with `merge_embeddings.py -sd` to reference multiple things. See official repo for more details.

<br>

# FAQ

### Q: How much VRAM does this require, why am I receiving a CUDA Out of Memory Error?

**A:** This model is very VRAM heavy, with 20GB being the recommended amount. It is possible to run this model on a GPU with <12GB of VRAM, but no guarantee. Try changing `size: 512` to `size: 448` in `v1-finetune.yaml -> data: -> params:` for both `train:` and `validation:`. If that is not enough, then it's probably best to use a Colab notebook or other GPU hosting service to do your training.

<br>

### Q: Why am I receiving a "SIGUSR1" error? Why am I receiving an NCNN error? Why am I receiving `OSError: cannot open resource`?

**A:** The script `main.py` was written without Windows in mind. 

You will need to open `main.py` and add the following line after the last import near the top of the script:

`os.environ["PL_TORCH_DISTRIBUTED_BACKEND"] = "gloo"`

Next, find the following lines near the end of the script. Change `SIGUSR1` and `SIGUSR2` to `SIGTERM`:

```
import signal

signal.signal(signal.SIGUSR1, melk)
signal.signal(signal.SIGUSR2, divein)
```

Finally, open the file `ldm/utils.py` and find this line: `font = ImageFont.truetype('data/DejaVuSans.ttf', size=size)`. Comment it out and replace it with this: `font = ImageFont.load_default()`

<br>

### Q: Why am I receiving an error about multiple devices detected?

**A:** Make sure you are using the `--gpus 1` argument. If you are still receiving the error, open `main.py` and find the following lines:

```
if not cpu:
    ngpu = len(lightning_config.trainer.gpus.strip(",").split(','))
else:
    ngpu = 1
```

Comment these lines out, then below them add `ngpu = 1` (Or whatever # of GPUs you want to use). Make sure that it is at the same indentation level as the line below it.

<br>

### Q: Why am I receiving an error about `if trainer.global_rank == 0:`?

**A:** Open `main.py` and scroll to the end of the file. On the last few lines, comment out the line where it says `if trainer.global_rank == 0:` and the line below it.

<br>

### Q: Why am I receiving errors about shapes? (IE: value tensor of shape [1280] cannot be broadcast to indexing result of shape [0, 768])

**A:** There are two known reasons for shape errors:

- The sanity check is failing when starting Textual-Inversion training. Try leaving out the `--actual-resume` argument when launching main.py. Chances are, the next error you receive will be an Out of Memory Error. See earlier in the FAQ for that.
- Stable Diffusion is erroring out when you try to use an embeddings file. This is likely because you ran the Textual-Inversion training with the wrong configuration. As of writing, TI and SD are not integrated. Make sure you have downloaded the `config/v1-finetune.yaml` file from this repo and that you use `--base configs/stable-diffusion/v1-finetune.yaml` when training embeddings. Retrain and try again.

<br>
