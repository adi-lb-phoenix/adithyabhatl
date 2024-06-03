
# code your own diffusion model under 10 lines of code.

Diffusion models are a type of deep learning models capable of generating images from images or from a text prompt . But you obviously knew that . In this blog I will introduce you on how to import the diffusion models and use them along with the pipelines the hugging face diffusers repo offers .

![](https://cdn-images-1.medium.com/max/2048/1*bEDSIsym77PEanm1EjhM1g.png)

    !pip install -Uq diffusers transformers fastcore
    import logging
    from pathlib import Path
    import matplotlib.pyplot as plt
    import torch 
    from diffusers import StableDiffusionPipeline  
    from fastcore.all import concat 
    from huggingface_hub import  notebook_login
    from PIL import Image
    logging.disable(logging.WARNING)
    torch.manual_seed(1)
    def image_grid(imgs,rows,cols):
        w,h = imgs[0].size
        grid=Image.new("RGB",size=(cols*w,rows*h))
        for i , img in enumerate(imgs):grid.paste(img,box=(i%cols*w,i//cols*h))
        return grid

    if not (Path.home()/'.cache/huggingface'/'token').exists(): notebook_login()
    

Create an account in the hugging face website [https://huggingface.co/](https://huggingface.co/settings/tokens) following which in settings column make sure to copy the tokens to login in the jupyter notebook for downloading the models .

    from diffusers import StableDiffusionImg2ImgPipeline
    from fastdownload import FastDownload
    pipe = StableDiffusionImg2ImgPipeline.from_pretrained("CompVis/stable-diffusion-v1-4",
                                                          revision='fp16',
                                                          torch_dtype=torch.float16,).to("cuda")
    pipe.enable_attention_slicing()

Here in the above lines of code .

    StableDiffusionImg2ImgPipeline 

is a pipeline which sort of combines inputs needed to create a diffusion model . The first argument of the .from_pretrained that is “ CompVis/stable-diffusion-v1-4” function is repository of the diffusion model in the hugging face website , which as of today is [https://huggingface.co/CompVis/stable-diffusion-v1-4](https://huggingface.co/CompVis/stable-diffusion-v1-4) . The revision argument refers to previson of the model you want to download , it can be 32 bit or 16 , its a handy argument because the loading and downloading of the model will vary depending on your revision type .The torch_dtype refers to the floating point precision of the loaded checkpoints. For example, if you want to save bandwidth by loading a fp16 variant, you should specify torch_dtype=torch.float16 to *convert the weights* to fp16. Otherwise, the fp16 weights are converted to the default fp32 precision. You can also load the original checkpoint without defining the variant argument, and convert it to fp16 with torch_dtype=torch.float16. In this case, the default fp32 weights are downloaded first, and then they’re converted to fp16 after loading.

Source : [https://huggingface.co/docs/diffusers/v0.19.3/en/using-diffusers/loading#checkpoint-variants](https://huggingface.co/docs/diffusers/v0.19.3/en/using-diffusers/loading#checkpoint-variants)

.to(“cuda”) is basically saying that the model will be loaded on to cuda computing device , why should we mention cuda is the question ? Because computations for cpu and cuda do not have the same processes , cuda is exponentially faster and thats why we need to mention .to(“cuda”) .

pipe.enable_attention_slicing() is again a way that helps in loading models a certain way If you have low GPU RAM available, make sure to add the line after sending it to cuda for less VRAM usage (to the cost of speed)”

So pipe now contains the model that we can now tinker with

    p=FastDownload().download('https://i.guim.co.uk/img/media/846cbc6489c031362413af3f79a315b112b10ea2/52_0_512_640/master/512.jpg?width=300&quality=45&auto=format&fit=max&dpr=2&s=76af6e5d6e7a6400399d6e1771dee3a7')
    init_image=Image.open(p).convert("RGB")
    init_image

the following line’s will have downloaded and stored a pencil sketch of a face .

    torch.manual_seed(100)
    prompt="face of a human in  joy"
    images=pipe(prompt=prompt,num_images_per_prompt=3,image=init_image,strength=0.8,num_inference_steps=50).images
    image_grid(images,rows=1,cols=3)

to know more about torch.manual_seed you can refer the link [https://sahilchachra.medium.com/paper-summary-torch-manual-seed-3407-is-all-you-need-9ef0f7aa7d78](https://sahilchachra.medium.com/paper-summary-torch-manual-seed-3407-is-all-you-need-9ef0f7aa7d78)

passing the prompt within the pipeline and its parameters will generate images whose count will be placed in the “num_images_per_prompt” parameter . Please not that this parameter is a must .

Strength represents how close the model has to generate images the map the prompt , higher the strength value lesser is the diversity .

the “image” parameter takes the input image from which the next image will be derived from to match the prompt to some extent depending on the strength value .

inference steps here mean the number of times the images must undergo the knife . So the image is generate after de-noising and noisy image . So inference steps mean the number of times a de-noising process takes place . Higher the inference steps , higher the image quality .

So after executing the above lines , you have used diffusion models to create an image from a random image .

So what else can you do with it . I can now use the image generated by the model to be draw in a way that a famous artist would have drawn . Pick your famous artist to see how similarities .

    torch.manual_seed(1000)
    prompt="electronic art   of an angry human by Leonardo da Vinci"
    images=pipe(prompt=prompt,num_images_per_prompt=3,image=init_image,strength=1,num_inference_steps=70).images
    image_grid(images,rows=1,cols=3)

In my case I did this and mahn oh mahn , I was blown away . I searched for the Leonardo da Vinci art and compared it the models output and I was like “wooooooowww” Technology is the closest thing to magic .

check this out : [https://github.com/adi-bhatl-1998/fastai_practice/blob/main/diffusion-model-2.ipynb](https://github.com/adi-bhatl-1998/fastai_practice/blob/main/diffusion-model-2.ipynb)
