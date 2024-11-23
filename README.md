# InterCom-and-AI-Prompt-for-Customer-Support
To create a Python program that can assist customer support agents using Intercom and AI prompting, we can integrate with the Intercom API to handle customer inquiries, and use AI models (e.g., GPT-3/ChatGPT) for better response generation.

The goal is to allow customer support agents to work efficiently by automating some of the customer interactions and providing intelligent prompts. This Python script will use Intercom API to read and respond to customer messages, and OpenAI's GPT for generating responses to support queries.
Prerequisites:

    Intercom API Access: You need to get your Intercom API Access Token to interact with the Intercom platform.
    OpenAI GPT-3 Access: You need to get an OpenAI API Key for generating AI-powered responses.
    Python packages:
        requests (for interacting with the Intercom API)
        openai (for generating AI responses)

Steps:

    Integrate with Intercom to fetch messages.
    Use GPT-3 to generate replies to queries.
    Automatically send responses via the Intercom API.

Python Code:

import requests
import openai
import json
import time

# Intercom API Credentials (you will need to replace these with your actual Intercom credentials)
INTERCOM_ACCESS_TOKEN = 'your_intercom_access_token'
INTERCOM_API_URL = 'https://api.intercom.io/messages'

# OpenAI API Key (replace this with your own API key from OpenAI)
OPENAI_API_KEY = 'your_openai_api_key'
openai.api_key = OPENAI_API_KEY

# Function to fetch conversations from Intercom
def get_intercom_conversations():
    headers = {
        'Authorization': f'Bearer {INTERCOM_ACCESS_TOKEN}',
        'Accept': 'application/json',
        'Content-Type': 'application/json'
    }
    
    response = requests.get('https://api.intercom.io/conversations', headers=headers)
    if response.status_code == 200:
        return response.json()
    else:
        print(f"Failed to fetch conversations: {response.status_code}")
        return []

# Function to get the latest message from a conversation
def get_latest_message(conversation_id):
    headers = {
        'Authorization': f'Bearer {INTERCOM_ACCESS_TOKEN}',
        'Accept': 'application/json',
        'Content-Type': 'application/json'
    }
    
    response = requests.get(f'https://api.intercom.io/conversations/{conversation_id}/parts', headers=headers)
    if response.status_code == 200:
        data = response.json()
        return data['conversation_parts']['conversation_parts'][-1]['body']
    else:
        print(f"Failed to fetch messages: {response.status_code}")
        return ""

# Function to generate a response using OpenAI's GPT
def generate_ai_response(prompt):
    try:
        response = openai.Completion.create(
            engine="text-davinci-003",
            prompt=prompt,
            max_tokens=150,
            n=1,
            stop=None,
            temperature=0.7
        )
        return response.choices[0].text.strip()
    except Exception as e:
        print(f"Error with OpenAI API: {str(e)}")
        return "Sorry, I am unable to generate a response right now."

# Function to send a reply back to Intercom
def send_reply_to_intercom(conversation_id, message):
    headers = {
        'Authorization': f'Bearer {INTERCOM_ACCESS_TOKEN}',
        'Accept': 'application/json',
        'Content-Type': 'application/json'
    }
    
    payload = {
        'message_type': 'comment',
        'body': message,
        'conversation_id': conversation_id
    }

    response = requests.post(INTERCOM_API_URL, headers=headers, data=json.dumps(payload))
    if response.status_code == 200:
        print(f"Successfully sent reply to conversation {conversation_id}")
    else:
        print(f"Failed to send reply: {response.status_code}")

# Main function to handle incoming messages and automate responses
def handle_customer_queries():
    while True:
        # Fetch conversations
        conversations = get_intercom_conversations()
        
        if conversations:
            for conversation in conversations['conversations']:
                conversation_id = conversation['id']
                customer_message = get_latest_message(conversation_id)

                if customer_message:
                    print(f"New message from customer: {customer_message}")
                    
                    # Generate AI response for the customer's message
                    ai_response = generate_ai_response(customer_message)
                    print(f"AI Response: {ai_response}")
                    
                    # Send the AI response back to Intercom as a reply
                    send_reply_to_intercom(conversation_id, ai_response)
        
        # Wait for a while before checking for new messages again
        time.sleep(10)  # Check every 10 seconds

# Start the bot to handle customer queries
if __name__ == "__main__":
    handle_customer_queries()

Breakdown of the Code:

    Intercom API Integration:
        get_intercom_conversations(): Fetches ongoing conversations from Intercom.
        get_latest_message(conversation_id): Retrieves the latest message in a specific conversation.
        send_reply_to_intercom(conversation_id, message): Sends a reply to the customer in the conversation.

    OpenAI GPT Integration:
        generate_ai_response(prompt): Uses GPT to generate a response based on the customer’s query.

    Automated Handling:
        The main loop (handle_customer_queries()) fetches messages from Intercom, generates responses via OpenAI, and sends them back to the platform automatically. The loop runs indefinitely, checking for new messages every 10 seconds.

How it Works:

    Intercom Messages: The bot fetches incoming customer messages using the Intercom API.
    AI Response: Once a message is received, it is processed by the OpenAI API to generate an intelligent response.
    Automated Reply: The bot automatically sends the generated response back to the customer via Intercom.

Next Steps:

    Enhance AI responses: Use advanced techniques like fine-tuning the OpenAI model or integrating knowledge bases for more accurate responses.
    Real-time Monitoring: Add logging, error handling, and notifications to ensure smooth operation.
    Personalized Responses: Consider implementing personalized responses based on customer data from Intercom (e.g., name, purchase history, etc.).

This setup can be an excellent starting point for providing AI-powered, automated customer support via Intercom.
