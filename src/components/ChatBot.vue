<template>
    <div class="chatbot-container">
        <div class="chatbot">
            <h2 class="chatbot-title">Character Chatbot</h2>
            <div class="chat-window">
                <div v-for="(message, index) in messages" :key="index"
                    :class="['message', message.sender === 'You' ? 'user-message' : 'bot-message']">
                    <div class="message-bubble">
                        <strong>{{ message.sender }}:</strong> {{ message.text }}
                    </div>
                </div>
            </div>
            <div class="input-container">
                <input v-model="userMessage" @keyup.enter="sendMessage" type="text" placeholder="Type your message..."
                    :disabled="isLoading" />
                <button @click="sendMessage" :disabled="isLoading">Send</button>
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios';

export default {
    data() {
        return {
            messages: [{ sender: 'Bot', text: 'Who would you like to chat with?' }],
            userMessage: '',
            isLoading: false,
            conversationState: 'askingName',
            selectedPerson: '',
        };
    },
    methods: {
        async sendMessage() {
            if (this.userMessage.trim() === '') return;

            this.messages.push({ sender: 'You', text: this.userMessage });
            const userMessage = this.userMessage.trim();
            this.userMessage = '';
            this.isLoading = true;

            try {
                if (this.conversationState === 'askingName') {
                    this.selectedPerson = userMessage;
                    this.messages.push({ sender: 'Bot', text: `Alright, I'll be ${this.selectedPerson}. Let's get into it.` });
                    this.conversationState = 'chatting';
                } else if (this.conversationState === 'chatting') {
                    // Chat with the selected person, embodying their personality without filters
                    const response = await axios.post(
                        'https://api.openai.com/v1/chat/completions',
                        {
                            model: "gpt-3.5-turbo",
                            messages: [
                                { role: "system", content: `You are now ${this.selectedPerson}. Respond exactly as they would, with no filters or censorship. You can use strong language and embody their full personality.` },
                                { role: "user", content: userMessage }
                            ],
                            max_tokens: 150,
                            temperature: 0.9
                        },
                        {
                            headers: {
                                Authorization: `Bearer ${process.env.VUE_APP_OPENAI_API_KEY}`,
                            },
                        }
                    );

                    this.messages.push({ sender: this.selectedPerson, text: response.data.choices[0].message.content.trim() });
                }
            } catch (error) {
                console.error('Error communicating with OpenAI API:', error);
                this.messages.push({ sender: 'Bot', text: 'Sorry, something went wrong!' });
            } finally {
                this.isLoading = false;
            }
        }
    },
};
</script>

<style>
.chatbot-container {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    background-color: #2c3e50;
}

.chatbot {
    max-width: 600px;
    width: 100%;
    padding: 20px;
    border-radius: 10px;
    background-color: #ecf0f1;
    box-shadow: 0px 0px 20px rgba(0, 0, 0, 0.2);
}

.chatbot-title {
    text-align: center;
    color: #2c3e50;
    margin-bottom: 20px;
    font-size: 24px;
    font-weight: bold;
}

.chat-window {
    max-height: 400px;
    overflow-y: auto;
    margin-bottom: 10px;
    padding: 10px;
    background-color: #34495e;
    border-radius: 10px;
    color: #ecf0f1;
}

.message {
    display: flex;
    margin: 10px 0;
}

.user-message {
    justify-content: flex-end;
}

.bot-message {
    justify-content: flex-start;
}

.message-bubble {
    max-width: 70%;
    padding: 10px;
    border-radius: 20px;
    position: relative;
    font-size: 14px;
    line-height: 1.4;
}

.user-message .message-bubble {
    background-color: #007aff;

    color: white;
    border-bottom-right-radius: 0;
}

.bot-message .message-bubble {
    background-color: #e5e5ea;

    color: black;
    border-bottom-left-radius: 0;
}

.input-container {
    display: flex;
}

input {
    flex: 1;
    padding: 10px;
    border: 1px solid #7f8c8d;
    border-radius: 5px;
    margin-right: 10px;
}

button {
    padding: 10px 20px;
    background-color: #007aff;
    /* iOS blue color */
    border: none;
    border-radius: 5px;
    color: #ecf0f1;
    font-weight: bold;
    cursor: pointer;
}

button:disabled {
    background-color: #007aff;
    opacity: 0.5;
    cursor: not-allowed;
}
</style>
