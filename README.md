# Pre-download Hugging Face Models

This repository contains a Python script (`pre_download_model.py`) to pre-download and cache machine learning models from the Hugging Face Hub.

## The Problem: Hugging Face Rate-Limiting

When using frameworks like Streamlit, web apps can often restart or reload. If your application downloads models from the Hugging Face Hub on startup, each reload can trigger a new download.

This frequent downloading can lead to being rate-limited by Hugging Face, which will prevent your application from working, showing an error similar to this:

`huggingface_hub.utils._errors.HfHubHTTPError: 429 Client Error: Too Many Requests for url: https://huggingface.co/api/models/microsoft/Florence-2-large (Request ID: ...) - You have been rate limited.`

![Example of rate limit](image001.png)

## The Solution: Pre-download and Cache

This script provides a simple solution: download the models you need *once*. The models will be saved to the Hugging Face cache on your local machine.

Once cached, your main application can load the models from the local cache instead of downloading them from the internet every time. This is much faster and avoids any rate-limiting issues.

## Requirements

To run the script, you'll need Python and the following libraries installed:
- `transformers`
- `torch`

You can install them using pip:
```bash
pip install transformers torch
```

## How to Use

### 1. Copy the Script to Your Project

**You SHOULD copy the `pre_download_model.py` script from this repository into your own project's folder.** It's recommended to rename it to something that reflects your project, for example, `download_my_project_models.py`.

This approach keeps your project-specific configurations separate and leaves the original script unchanged for future use.

### 2. Configure The Models to Download

Open your newly created script (e.g., `download_my_project_models.py`) and modify the `SIMPLE_MODELS` and `ADVANCED_MODELS` lists at the top of the file to include the models you need.

#### For simple models

Most models can be downloaded by simply adding their Hugging Face identifier to the `SIMPLE_MODELS` list. These are downloaded using the standard `AutoModel` and `AutoProcessor` classes from the `transformers` library.

```python
# pre_download_model.py

SIMPLE_MODELS = [
    "openai/clip-vit-base-patch32",
    "another/model-id", # Add your model here
]
```

#### For models requiring special configuration

Some models require special settings, like `trust_remote_code=True` or a specific model class (e.g., `AutoModelForCausalLM`). For these, add a configuration dictionary to the `ADVANCED_MODELS` list.

```python
# pre_download_model.py

ADVANCED_MODELS = [
    {
        "id": "microsoft/Florence-2-large",
        "model_class": AutoModelForCausalLM, # Uses a specific model class
        "trust_remote_code": True,           # Requires trusting remote code
    },
    # Add other advanced model configurations here
]
```

You can customize the `id`, `model_class`, `processor_class`, and `trust_remote_code` for each advanced model. If you use a new model class, remember to import it from `transformers`.

#### How to check for special settings

How do you know if a model belongs in `SIMPLE_MODELS` or `ADVANCED_MODELS`? You can find out by checking its Hugging Face model page.

1. **Find the Model Page**: Go to the Hugging Face Hub (`https://huggingface.co/models`) and search for your model (e.g., `microsoft/Florence-2-large`).

2. **Check the "Model Card"**: The main page for the model is called the "Model Card." This page usually has a "How to use" or similar section with code examples.

3. **Look for Clues in the Code:**
    * **`trust_remote_code=True`**: If the example code includes `trust_remote_code=True` when loading the model, you **must** add the model to the `ADVANCED_MODELS` list and set `"trust_remote_code": True`.
    * **Specific Model Classes**: The `transformers` library has many specific model classes (like `AutoModelForCausalLM`, `T5ForConditionalGeneration`, etc.). If the example code uses one of these instead of the general `AutoModel`, it's safest to add it to `ADVANCED_MODELS` and specify that `model_class`.

**Rule of Thumb:** If the Hugging Face page shows a simple `AutoModel.from_pretrained("your-model-id")` with no other special parameters, you can add it to the `SIMPLE_MODELS` list. For anything more complex, `ADVANCED_MODELS` is the right choice.

### 3. Run the script

**Important 🚨🚨:** It's crucial to run this script in the same virtual environment (`venv`) as your GenAI or Streamlit project. This ensures the models are downloaded to the correct cache directory, so your main application can find them.

Execute the script from your terminal:

```bash
python your_new_script_name.py
```

The script will download each model and you'll see progress in the terminal.

### 4. Run your application

That's it! Now you can run your main application (e.g., your Streamlit app). It will load the models from the local cache, making it faster and more reliable.

## Benefits

* **Avoid Rate-Limiting**: By downloading each model only once, you won't hit Hugging Face's rate limits.
* **Faster Startup**: Loading from a local cache is significantly faster than downloading from the internet.
* **Offline Access**: After the initial download, your application can load the models even without an internet connection.

## FAQ

### Why does the script use `AutoModel` instead of specific classes like `CLIPModel`?

That's an excellent question. While you can use specific model classes like `CLIPModel` or `CLIPProcessor`, this script uses the more general `AutoModel` and `AutoProcessor` for a few key reasons:

1. **Simplicity for Users**: By using `AutoModel`, you can add most new models to the `SIMPLE_MODELS` list without needing to know their specific class name (`CLIPModel`, `BertModel`, `T5ForConditionalGeneration`, etc.). You just need the model's Hugging Face identifier.

2. **Universal and Maintainable Code**: Using the `Auto` classes allows a single, generic `download_model` function to handle the vast majority of models. This makes the script cleaner and easier to maintain than having a separate download function for each model type.

The `transformers` library's `Auto` classes are smart. When you provide an identifier like `"openai/clip-vit-base-patch32"`, `AutoModel` inspects the model's configuration on the Hugging Face Hub and automatically loads the correct architecture (in this case, `CLIPModel`). The script therefore achieves the same result but in a more flexible and user-friendly way.

The `ADVANCED_MODELS` list exists for the rare cases where this automatic process needs to be overridden, such as when `trust_remote_code` is required.