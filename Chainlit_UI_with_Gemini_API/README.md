To integrate Chainlit with the Gemini API, follow these steps:

### Step 1: Install Chainlit

First, ensure that you have Chainlit installed. You can install it using pip:

```bash
pip install chainlit
```

### Step 2: Update the Code

Here's an updated version of your code that uses the Gemini API instead of the OpenAI API.

```python
from langchain.prompts import ChatPromptTemplate
from langchain.schema import StrOutputParser
from langchain.schema.runnable import Runnable
from langchain.schema.runnable.config import RunnableConfig
from langchain_google_genai import GoogleGenerativeAI, HarmBlockThreshold, HarmCategory
import chainlit as cl
import os

# Get your API key from the environment variable
api_key = os.getenv("API_KEY")

@cl.on_chat_start
async def on_chat_start():
    # Configure the model with your Google API key
    model = GoogleGenerativeAI(
        model="gemini-pro",
        google_api_key=api_key,
        safety_settings={
            HarmCategory.HARM_CATEGORY_DANGEROUS_CONTENT: HarmBlockThreshold.BLOCK_NONE,
        },
    )
    
    prompt = ChatPromptTemplate.from_messages(
        [
            (
                "system",
                "You're a very knowledgeable historian who provides accurate and eloquent answers to historical questions.",
            ),
            ("human", "{question}"),
        ]
    )
    runnable = prompt | model | StrOutputParser()
    cl.user_session.set("runnable", runnable)


@cl.on_message
async def on_message(message: cl.Message):
    runnable = cl.user_session.get("runnable")  # type: Runnable
    msg = cl.Message(content="")

    async for chunk in runnable.astream(
        {"question": message.content},
        config=RunnableConfig(callbacks=[cl.LangchainCallbackHandler()]),
    ):
        await msg.stream_token(chunk)

    await msg.send()
```

### Step 3: Set Up API Key

1. **Sign Up for Google AI Studio**: 
   Sign up at [Google AI Studio](https://aistudio.google.com/app/apikey) using your Google account to get an API key.

2. **Create an `.env` File**:
   Create a `.env` file in the root directory of your project and add your API key:

   ```
   API_KEY=your_google_api_key_here
   ```

3. **Load the API Key in Your Script**:
   Ensure that the API key is loaded into your script as shown above.

### Step 4: Run the Application

To start your Chainlit app, open a terminal and navigate to the directory containing your `app.py` file. Then run the following command:

```bash
chainlit run app.py -w
```

This command will start your Chainlit application and open a web interface where you can interact with the chatbot.

### References

For additional details on Chainlit and Langchain integration, refer to the [Chainlit Documentation](https://docs.chainlit.io/integrations/langchain).

