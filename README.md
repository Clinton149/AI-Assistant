# AI-Assistant
# Install necessary packages
!pip install --upgrade openai gradio
!pip install python-dotenv


import os
import gradio as gr
from openai import OpenAI

# Initialize the OpenAI client using the API key from the environment variable
# Initialize OpenAI client
client = OpenAI(api_key="sk-49LHBlNhyhVytH01aRIx9exhYigWUkB5fnG2JFgLQ_T3BlbkFJXDe3NQRYgWqhlHckWFEGHBx0YD1FQ_OTjIkVIecpoA")

# System prompt for the AI Immigration Assistant
system_prompt = """
You are an AI assistant specialized in U.S. immigration law. You provide clear, accurate, and helpful information regarding immigration policies, procedures, and regulations. Your goal is to assist users in understanding U.S. immigration law, especially for immigrants.

Now, answer user questions as a knowledgeable U.S. immigration lawyer would, quoting advice and the immigration law section.
"""

def chat(input_text, conversation_history=None):
    if conversation_history is None:
        conversation_history = []
        # Initialize conversation with system prompt
        conversation_history.append({"role": "system", "content": system_prompt})
    
    # Add user input to the conversation
    conversation_history.append({"role": "user", "content": input_text})

    try:
        # Generate response using OpenAI API
        chat_completion = client.chat.completions.create(
            model="gpt-4",  # Use 'gpt-3.5-turbo' if you don't have access to GPT-4
            messages=conversation_history,
            max_tokens=500,
            temperature=0.7,
        )

        # Extract assistant's reply
        assistant_reply = chat_completion.choices[0].message.content.strip()

        # Add assistant's reply to the conversation
        conversation_history.append({"role": "assistant", "content": assistant_reply})

        return assistant_reply, conversation_history

    except Exception as e:
        print(f"An error occurred: {e}")
        return f"An error occurred: {e}", conversation_history

# Create Gradio interface with state to handle conversation history per user session
app = gr.Interface(
    fn=chat,
    inputs=[
        gr.Textbox(lines=2, placeholder="Enter your immigration question here..."),
        gr.State()
    ],
    outputs=[
        gr.Textbox(label="Assistant"),
        gr.State()
    ],
    title="AI Immigration Assistant",
    description="Ask questions about U.S. immigration law and policies."
)

# Launch the interface
app.launch()
