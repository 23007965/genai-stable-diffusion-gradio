## Prototype Development for Image Generation Using the Stable Diffusion Model and Gradio Framework

### AIM:
To design and deploy a prototype application for image generation utilizing the Stable Diffusion model, integrated with the Gradio UI framework for interactive user engagement and evaluation.

### PROBLEM STATEMENT:
To develop a prototype application that generates images from user-provided text prompts using the Stable Diffusion model. The application should provide an interactive Gradio user interface where users can enter prompts and adjust image-generation parameters such as inference steps, guidance scale, width, and height.

### DESIGN STEPS:

#### STEP 1:

Set up the Stable Diffusion API

- Import the required Python libraries.
- Load the Hugging Face API key using the .env file.
- Define the text-to-image API endpoint.
- Create a function to send the user prompt and parameters to the Stable Diffusion model.

#### STEP 2:

Implement image generation

- Accept the user's text prompt and optional negative prompt.
- Set parameters such as:
- Inference steps
- Guidance scale
- Image width
- Image height
- Send these parameters to the Stable Diffusion API.
- Convert the generated Base64 image data into a PIL image.

#### STEP 3:

Develop the Gradio interface

- Create an interactive Gradio interface using gr.Blocks().
- Add a textbox for entering the image prompt.
- Add a Submit button.
- Provide advanced options for negative prompt, inference steps, guidance scale, width, and height.
- Display the generated image as the output.
- Launch the application using Gradio for interactive user evaluation.

### PROGRAM:
```python
import os
import io
import IPython.display
from PIL import Image
import base64 
from dotenv import load_dotenv, find_dotenv
_ = load_dotenv(find_dotenv()) # read local .env file
hf_api_key = os.environ['HF_API_KEY']
```
```python
# Helper function
import requests, json

#Text-to-image endpoint
def get_completion(inputs, parameters=None, ENDPOINT_URL=os.environ['HF_API_TTI_BASE']):
    headers = {
      "Authorization": f"Bearer {hf_api_key}",
      "Content-Type": "application/json"
    }   
    data = { "inputs": inputs }
    if parameters is not None:
        data.update({"parameters": parameters})
    response = requests.request("POST",
                                ENDPOINT_URL,
                                headers=headers,
                                data=json.dumps(data))
    return json.loads(response.content.decode("utf-8"))
```
```python
import gradio as gr 

#A helper function to convert the PIL image to base64 
# so you can send it to the API
def base64_to_pil(img_base64):
    base64_decoded = base64.b64decode(img_base64)
    byte_stream = io.BytesIO(base64_decoded)
    pil_image = Image.open(byte_stream)
    return pil_image

def generate(prompt, negative_prompt, steps, guidance, width, height):
    params = {
        "negative_prompt": negative_prompt,
        "num_inference_steps": steps,
        "guidance_scale": guidance,
        "width": width,
        "height": height
    }
    
    output = get_completion(prompt, params)
    pil_image = base64_to_pil(output)
    return pil_image
```
```python
with gr.Blocks() as demo:
    gr.Markdown("# Image Generation with Stable Diffusion")
    with gr.Row():
        with gr.Column(scale=4):
            prompt = gr.Textbox(label="Your prompt") #Give prompt some real estate
        with gr.Column(scale=1, min_width=50):
            btn = gr.Button("Submit") #Submit button side by side!
    with gr.Accordion("Advanced options", open=False): #Let's hide the advanced options!
            negative_prompt = gr.Textbox(label="Negative prompt")
            with gr.Row():
                with gr.Column():
                    steps = gr.Slider(label="Inference Steps", minimum=1, maximum=100, value=25,
                      info="In many steps will the denoiser denoise the image?")
                    guidance = gr.Slider(label="Guidance Scale", minimum=1, maximum=20, value=7,
                      info="Controls how much the text prompt influences the result")
                with gr.Column():
                    width = gr.Slider(label="Width", minimum=64, maximum=512, step=64, value=512)
                    height = gr.Slider(label="Height", minimum=64, maximum=512, step=64, value=512)
    output = gr.Image(label="Result") #Move the output up too
            
    btn.click(fn=generate, inputs=[prompt,negative_prompt,steps,guidance,width,height], outputs=[output])

gr.close_all()
demo.launch(share=True, server_port=int(os.environ['PORT4']))
```
```python
gr.close_all()
```
### OUTPUT:

##### Prompt : A beautiful mountain landscape with snow-covered mountains, a lake, green trees, and a colorful sunset, realistic photography

<img width="512" height="512" alt="download" src="https://github.com/user-attachments/assets/237c628e-3611-4ecb-8dff-e54ebbdc8740" />


### RESULT:
The prototype application was successfully developed and deployed using the Stable Diffusion model and Gradio framework.
