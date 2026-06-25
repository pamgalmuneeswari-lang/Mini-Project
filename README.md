<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Django ChatGPT Clone</title>
    <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gray-900 text-gray-100 h-screen flex flex-col">

    <header class="bg-gray-800 p-4 text-center text-xl font-bold border-b border-gray-700">
        Django GPT Assistant
    </header>

    <div id="chat-box" class="flex-1 overflow-y-auto p-4 space-y-4 max-w-3xl mx-auto w-full">
        {% for chat in history %}
            <div class="text-right">
                <span class="bg-blue-600 text-white px-4 py-2 rounded-lg inline-block max-w-xs break-words">{{ chat.message }}</span>
            </div>
            <div class="text-left">
                <span class="bg-gray-700 text-gray-200 px-4 py-2 rounded-lg inline-block max-w-xs break-words response-bubble">{{ chat.response }}</span>
            </div>
        {% endfor %}
    </div>

    <footer class="bg-gray-800 p-4 border-t border-gray-700">
        <form id="chat-form" class="max-w-3xl mx-auto flex gap-2">
            {% csrf_token %}
            <input type="text" id="user-input" placeholder="Type your message here..." required
                   class="flex-1 bg-gray-700 text-white rounded-lg px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500">
            <button type="submit" class="bg-blue-600 hover:bg-blue-500 px-6 py-2 rounded-lg font-semibold transition">
                Send
            </button>
        </form>
    </footer>

    <script>
        const chatForm = document.getElementById('chat-form');
        const chatBox = document.getElementById('chat-box');
        const userInput = document.getElementById('user-input');

        // Helper function to turn plain URLs into clickable links
        function linkify(text) {
            const urlRegex = /(https?:\/\/[^\s]+)/g;
            return text.replace(urlRegex, function(url) {
                return '<a href="' + url + '" target="_blank" class="text-blue-400 underline hover:text-blue-300 break-all">' + url + '</a>';
            });
        }

        // Format any pre-existing history links on page load
        document.querySelectorAll('.response-bubble').forEach(bubble => {
            bubble.innerHTML = linkify(bubble.innerText);
        });

        chatForm.addEventListener('submit', async (e) => {
            e.preventDefault();
            const message = userInput.value.trim();
            if (!message) return;

            // Append user message immediately
            appendMessage(message, 'user');
            userInput.value = '';

            // Extract the CSRF token from the template's hidden inputs
            const csrfToken = document.querySelector('[name=csrfmiddlewaretoken]').value;

            // Send to Django API
            try {
                const response = await fetch('/api/ask/', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/x-www-form-urlencoded',
                        'X-CSRFToken': csrfToken // REQUIRED: Passing CSRF to header
                    },
                    body: new URLSearchParams({ 
                        'message': message,
                        'csrfmiddlewaretoken': csrfToken // Passing CSRF in the body parameters
                    })
                });
                
                const data = await response.json();
                
                if (data.response) {
                    appendMessage(data.response, 'bot');
                } else {
                    appendMessage('Error: ' + data.error, 'bot');
                }
            } catch (error) {
                appendMessage('Failed to connect to server.', 'bot');
            }
        });

        function appendMessage(text, sender) {
            const wrapper = document.createElement('div');
            wrapper.className = sender === 'user' ? 'text-right' : 'text-left';
            
            const bubble = document.createElement('span');
            bubble.className = sender === 'user' 
                ? 'bg-blue-600 text-white px-4 py-2 rounded-lg inline-block max-w-xs break-words' 
                : 'bg-gray-700 text-gray-200 px-4 py-2 rounded-lg inline-block max-w-xs break-words';
            
            if (sender === 'bot') {
                // Parse URL links seamlessly for AI outputs
                bubble.innerHTML = linkify(text);
            } else {
                bubble.innerText = text;
            }
            
            wrapper.appendChild(bubble);
            chatBox.appendChild(wrapper);
            chatBox.scrollTop = chatBox.scrollHeight; // Auto scroll down
        }
    </script>
</body>
</html>
