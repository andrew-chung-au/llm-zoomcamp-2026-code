## Question

How do I start using Google Gemini models in the module 1 notebook through the OpenAI-compatible endpoint?

## Answer

To get started, you need three things:

1. A Gemini API key saved in your `.env` file, for example as `GEMINI_API_KEY`.
2. An OpenAI client pointed at Google’s OpenAI-compatible base URL.
3. Your selected Google Gemini model name in your request.

Here is example code that loads the API key from `.env`, creates the Gemini client, and defines the `llm` helper:

```python
import os
from dotenv import load_dotenv
from openai import OpenAI

load_dotenv()

client = OpenAI(
    api_key=os.getenv("GEMINI_API_KEY"),
    base_url="https://generativelanguage.googleapis.com/v1beta/openai/"
)

def llm(instructions, user_prompt, model="gemini-3.1-flash-lite"):
    message_history = [
        {"role": "developer", "content": instructions},
        {"role": "user", "content": user_prompt}
    ]

    response = client.chat.completions.create(
        model=model,
        messages=message_history
    )

    return response.choices[0].message.content
```

This uses the older chat completions style via the OpenAI-compatible endpoint, whereas many course examples use the newer Responses format. That means you will need to change the notebook code in a few places, especially where it reads the model response and where it handles tools or function calls.