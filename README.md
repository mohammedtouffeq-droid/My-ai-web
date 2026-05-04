<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Blackbox AI - Gemini Chat</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            background: linear-gradient(135deg, #0f0f23 0%, #1a1a2e 50%, #16213e 100%);
            height: 100vh;
            display: flex;
            flex-direction: column;
            color: #e2e8f0;
        }

        .header {
            background: rgba(15, 15, 35, 0.95);
            backdrop-filter: blur(20px);
            padding: 1rem 2rem;
            border-bottom: 1px solid rgba(255, 255, 255, 0.1);
            box-shadow: 0 4px 30px rgba(0, 0, 0, 0.3);
        }

        .header h1 {
            font-size: 1.5rem;
            font-weight: 600;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
        }

        .chat-container {
            flex: 1;
            overflow-y: auto;
            padding: 2rem;
            max-width: 800px;
            margin: 0 auto;
            width: 100%;
        }

        .message {
            margin-bottom: 1.5rem;
            display: flex;
            opacity: 0;
            transform: translateY(20px);
            animation: slideIn 0.4s ease forwards;
        }

        .message.user {
            justify-content: flex-end;
        }

        .message.assistant {
            justify-content: flex-start;
        }

        .message-bubble {
            max-width: 70%;
            padding: 1rem 1.25rem;
            border-radius: 1.25rem;
            position: relative;
            backdrop-filter: blur(20px);
            box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
            border: 1px solid rgba(255, 255, 255, 0.1);
        }

        .message.user .message-bubble {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
        }

        .message.assistant .message-bubble {
            background: rgba(40, 40, 60, 0.8);
            color: #e2e8f0;
            border: 1px solid rgba(100, 100, 150, 0.3);
        }

        .message-bubble.loading {
            background: rgba(100, 100, 150, 0.3);
            border-color: rgba(100, 150, 255, 0.5);
            display: flex;
            align-items: center;
            gap: 0.75rem;
        }

        .typing-indicator {
            display: flex;
            gap: 0.25rem;
        }

        .typing-dot {
            width: 8px;
            height: 8px;
            border-radius: 50%;
            background: rgba(255, 255, 255, 0.6);
            animation: typing 1.4s infinite ease-in-out;
        }

        .typing-dot:nth-child(1) { animation-delay: -0.32s; }
        .typing-dot:nth-child(2) { animation-delay: -0.16s; }

        @keyframes typing {
            0%, 80%, 100% { transform: scale(0.8); opacity: 0.5; }
            40% { transform: scale(1); opacity: 1; }
        }

        @keyframes slideIn {
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }

        .input-container {
            padding: 2rem;
            background: rgba(15, 15, 35, 0.95);
            backdrop-filter: blur(20px);
            border-top: 1px solid rgba(255, 255, 255, 0.1);
            box-shadow: 0 -4px 30px rgba(0, 0, 0, 0.3);
        }

        .input-wrapper {
            display: flex;
            gap: 1rem;
            max-width: 800px;
            margin: 0 auto;
        }

        #messageInput {
            flex: 1;
            padding: 1rem 1.5rem;
            border: 1px solid rgba(255, 255, 255, 0.2);
            border-radius: 2rem;
            background: rgba(40, 40, 60, 0.8);
            color: #e2e8f0;
            font-size: 1rem;
            outline: none;
            backdrop-filter: blur(20px);
            transition: all 0.3s ease;
        }

        #messageInput:focus {
            border-color: rgba(100, 150, 255, 0.6);
            box-shadow: 0 0 0 3px rgba(100, 150, 255, 0.1);
        }

        #sendBtn {
            padding: 1rem 2rem;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            border: none;
            border-radius: 2rem;
            cursor: pointer;
            font-weight: 600;
            font-size: 1rem;
            transition: all 0.3s ease;
            backdrop-filter: blur(20px);
        }

        #sendBtn:hover:not(:disabled) {
            transform: translateY(-2px);
            box-shadow: 0 10px 25px rgba(102, 126, 234, 0.4);
        }

        #sendBtn:disabled {
            opacity: 0.6;
            cursor: not-allowed;
            transform: none;
        }

        .welcome-message {
            text-align: center;
            color: rgba(226, 232, 240, 0.6);
            font-size: 1.1rem;
            margin: 4rem 0;
            max-width: 500px;
            margin-left: auto;
            margin-right: auto;
        }

        @media (max-width: 768px) {
            .chat-container {
                padding: 1rem;
            }
            
            .input-container {
                padding: 1rem;
            }
            
            .message-bubble {
                max-width: 85%;
            }
            
            .input-wrapper {
                flex-direction: column;
            }
            
            #sendBtn {
                width: 100%;
            }
        }

        ::-webkit-scrollbar {
            width: 8px;
        }

        ::-webkit-scrollbar-track {
            background: rgba(40, 40, 60, 0.5);
        }

        ::-webkit-scrollbar-thumb {
            background: rgba(100, 100, 150, 0.6);
            border-radius: 4px;
        }

        ::-webkit-scrollbar-thumb:hover {
            background: rgba(100, 150, 255, 0.8);
        }
    </style>
</head>
<body>
    <div class="header">
        <h1>🖤 Blackbox AI</h1>
    </div>

    <div class="chat-container" id="chatContainer">
        <div class="welcome-message">
            Ask me anything... I'm powered by Google's Gemini AI.
        </div>
    </div>

    <div class="input-container">
        <div class="input-wrapper">
            <input type="text" id="messageInput" placeholder="Type your message..." autocomplete="off">
            <button id="sendBtn">Send</button>
        </div>
    </div>

    <script>
        const API_KEY = 'AIzaSyAWZtqiChxNV6Dh3bM12fomqdIitxD3yEM';
        const API_URL = `https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash-latest:generateContent?key=${API_KEY}`;

        const chatContainer = document.getElementById('chatContainer');
        const messageInput = document.getElementById('messageInput');
        const sendBtn = document.getElementById('sendBtn');

        let conversationHistory = [];

        // Send message function
        async function sendMessage() {
            const message = messageInput.value.trim();
            if (!message) return;

            // Add user message
            addMessage(message, 'user');
            messageInput.value = '';
            sendBtn.disabled = true;

            // Add loading indicator
            const loadingId = addLoadingMessage();

            try {
                const response = await fetch(API_URL, {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                    },
                    body: JSON.stringify({
                        contents: [{
                            role: 'user',
                            parts: [{ text: message }]
                        }],
                        generationConfig: {
                            temperature: 0.7,
                            topK: 40,
                            topP: 0.95,
                            maxOutputTokens: 2048,
                        }
                    })
                });

                if (!response.ok) {
                    throw new Error(`API Error: ${response.status}`);
                }

                const data = await response.json();
                const aiResponse = data.candidates[0].content.parts[0].text;

                // Replace loading with actual response
                replaceLoadingMessage(loadingId, aiResponse);

            } catch (error) {
                console.error('Error:', error);
                replaceLoadingMessage(loadingId, 'Sorry, I encountered an error. Please try again.');
            } finally {
                sendBtn.disabled = false;
                messageInput.focus();
            }
        }

        // Add message to chat
        function addMessage(text, sender) {
            const messageDiv = document.createElement('div');
            messageDiv.className = `message ${sender}`;
            messageDiv.innerHTML = `
                <div class="message-bubble">
                    ${text}
                </div>
            `;
            chatContainer.appendChild(messageDiv);
            chatContainer.scrollTop = chatContainer.scrollHeight;
        }

        // Add loading message
        function addLoadingMessage() {
            const loadingId = Date.now();
            const messageDiv = document.createElement('div');
            messageDiv.id = `loading-${loadingId}`;
            messageDiv.className = 'message assistant';
            messageDiv.innerHTML = `
                <div class="message-bubble loading">
                    <div class="typing-indicator">
                        <div class="typing-dot"></div>
                        <div class="typing-dot"></div>
                        <div class="typing-dot"></div>
                    </div>
                </div>
            `;
            chatContainer.appendChild(messageDiv);
            chatContainer.scrollTop = chatContainer.scrollHeight;
            return loadingId;
        }

        // Replace loading message with actual response
        function replaceLoadingMessage(loadingId, text) {
            const loadingElement = document.getElementById(`loading-${loadingId}`);
            if (loadingElement) {
                const messageDiv = loadingElement;
                messageDiv.className = 'message assistant';
                messageDiv.innerHTML = `
                    <div class="message-bubble">
                        ${text.replace(/\n/g, '<br>')}
                    </div>
                `;
            }
        }

        // Event listeners
        sendBtn.addEventListener('click', sendMessage);

        messageInput.addEventListener('keypress', (e) => {
            if (e.key === 'Enter' && !e.shiftKey) {
                e.preventDefault();
                sendMessage();
            }
        });

        // Focus input on load
        messageInput.focus();
    </script>
</body>
</html>
