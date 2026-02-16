<!DOCTYPE html>
<html lang="en" class="light">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Universal AI Chat</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/marked/marked.min.js"></script>
    <style>
        /* Custom scrollbar and mobile fixes */
        ::-webkit-scrollbar { width: 6px; }
        ::-webkit-scrollbar-track { background: transparent; }
        ::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 10px; }
        .dark ::-webkit-scrollbar-thumb { background: #475569; }
        
        body { height: 100svh; overflow: hidden; }
        #app { height: 100svh; display: flex; flex-direction: column; }
        
        /* Markdown Styling */
        .prose pre { background: #1e293b; color: white; padding: 1rem; border-radius: 0.5rem; overflow-x: auto; margin: 0.5rem 0; }
        .prose code { font-family: monospace; background: rgba(0,0,0,0.1); padding: 0.2rem 0.4rem; border-radius: 0.25rem; }
        .dark .prose code { background: rgba(255,255,255,0.1); }
        .prose p { margin-bottom: 0.75rem; }

        /* Sidebar Transition */
        #sidebar { transition: transform 0.3s ease-in-out; z-index: 50; }
        @media (max-width: 768px) {
            #sidebar.closed { transform: translateX(-100%); }
        }
    </style>
</head>
<body class="bg-white dark:bg-slate-900 text-slate-900 dark:text-slate-100 transition-colors duration-200">

    <div id="loading" class="fixed inset-0 z-[100] flex items-center justify-center bg-white dark:bg-slate-900">
        <div class="flex flex-col items-center">
            <div class="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-600"></div>
            <p class="mt-4 font-medium">Initializing AI Interface...</p>
        </div>
    </div>

    <div id="app" class="hidden">
        <header class="h-14 border-b dark:border-slate-800 flex items-center justify-between px-4 bg-white/80 dark:bg-slate-900/80 backdrop-blur-md">
            <div class="flex items-center gap-3">
                <button id="toggleSidebar" class="p-2 hover:bg-slate-100 dark:hover:bg-slate-800 rounded-lg">
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="3" y1="12" x2="21" y2="12"></line><line x1="3" y1="6" x2="21" y2="6"></line><line x1="3" y1="18" x2="21" y2="18"></line></svg>
                </button>
                <h1 class="font-bold text-lg tracking-tight">AI <span class="text-blue-600">Studio</span></h1>
            </div>
            <div class="flex items-center gap-2">
                <button id="themeToggle" class="p-2 hover:bg-slate-100 dark:hover:bg-slate-800 rounded-lg">
                    <span class="dark:hidden">🌙</span><span class="hidden dark:inline">☀️</span>
                </button>
                <button id="openSettings" class="p-2 hover:bg-slate-100 dark:hover:bg-slate-800 rounded-lg">
                    ⚙️
                </button>
            </div>
        </header>

        <div class="flex flex-1 overflow-hidden relative">
            <aside id="sidebar" class="fixed md:relative w-72 h-full bg-slate-50 dark:bg-slate-950 border-r dark:border-slate-800 flex flex-col closed md:transform-none">
                <div class="p-4">
                    <button id="newChat" class="w-full py-2 px-4 bg-blue-600 hover:bg-blue-700 text-white rounded-full font-medium transition-all flex items-center justify-center gap-2">
                        <span>+</span> New Chat
                    </button>
                </div>
                <div id="chatHistory" class="flex-1 overflow-y-auto px-2 space-y-1">
                    </div>
            </aside>

            <main class="flex-1 flex flex-col bg-white dark:bg-slate-900 relative">
                <div id="messages" class="flex-1 overflow-y-auto p-4 md:p-8 space-y-6">
                    <div id="emptyState" class="h-full flex flex-col items-center justify-center text-center space-y-4">
                        <div class="text-5xl">✨</div>
                        <h2 class="text-2xl font-bold">How can I help you today?</h2>
                        <p class="text-slate-500 max-w-sm">Enter your OpenRouter API key in settings to start chatting.</p>
                    </div>
                </div>

                <div class="p-4 bg-white dark:bg-slate-900">
                    <div class="max-w-4xl mx-auto relative flex items-end gap-2 bg-slate-100 dark:bg-slate-800 rounded-2xl p-2 focus-within:ring-2 ring-blue-500 transition-all">
                        <textarea id="userInput" rows="1" placeholder="Type a message..." 
                            class="flex-1 bg-transparent border-none focus:ring-0 resize-none py-2 px-3 max-h-40 outline-none"></textarea>
                        <button id="sendBtn" class="bg-blue-600 text-white p-2 rounded-xl hover:opacity-90 disabled:opacity-50 disabled:cursor-not-allowed">
                            <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="22" y1="2" x2="11" y2="13"></line><polygon points="22 2 15 22 11 13 2 9 22 2"></polygon></svg>
                        </button>
                    </div>
                    <p class="text-[10px] text-center mt-2 text-slate-400">Powered by OpenRouter. AI can make mistakes.</p>
                </div>
            </main>
        </div>

        <div id="settingsModal" class="fixed inset-0 z-[60] hidden flex items-center justify-center p-4 bg-black/50 backdrop-blur-sm">
            <div class="bg-white dark:bg-slate-900 w-full max-w-md rounded-2xl shadow-2xl overflow-hidden">
                <div class="p-6 border-b dark:border-slate-800 flex justify-between items-center">
                    <h3 class="text-xl font-bold">Settings</h3>
                    <button id="closeSettings" class="text-2xl">&times;</button>
                </div>
                <div class="p-6 space-y-4">
                    <div>
                        <label class="block text-sm font-medium mb-1">OpenRouter API Key</label>
                        <input type="password" id="apiKeyInput" class="w-full p-2 rounded-lg border dark:bg-slate-800 dark:border-slate-700" placeholder="sk-or-...">
                    </div>
                    <div>
                        <label class="block text-sm font-medium mb-1">Model ID</label>
                        <input type="text" id="modelInput" class="w-full p-2 rounded-lg border dark:bg-slate-800 dark:border-slate-700" placeholder="deepseek/deepseek-r1:free">
                    </div>
                    <div>
                        <label class="block text-sm font-medium mb-1">System Prompt</label>
                        <textarea id="systemPromptInput" class="w-full p-2 rounded-lg border dark:bg-slate-800 dark:border-slate-700" rows="3" placeholder="You are a helpful assistant..."></textarea>
                    </div>
                    <button id="saveSettings" class="w-full py-2 bg-blue-600 text-white rounded-lg font-bold">Save Configuration</button>
                </div>
            </div>
        </div>
    </div>

    <script>
        // --- State Management ---
        let state = {
            apiKey: localStorage.getItem('ai_key') || '',
            model: localStorage.getItem('ai_model') || 'deepseek/deepseek-r1:free',
            systemPrompt: localStorage.getItem('ai_system') || 'You are a helpful, concise AI assistant.',
            chats: JSON.parse(localStorage.getItem('ai_chats')) || [],
            currentChatId: null,
            isDark: localStorage.getItem('ai_theme') === 'dark'
        };

        // --- DOM Elements ---
        const els = {
            app: document.getElementById('app'),
            loader: document.getElementById('loading'),
            messages: document.getElementById('messages'),
            userInput: document.getElementById('userInput'),
            sendBtn: document.getElementById('sendBtn'),
            chatHistory: document.getElementById('chatHistory'),
            newChatBtn: document.getElementById('newChat'),
            toggleSidebar: document.getElementById('toggleSidebar'),
            sidebar: document.getElementById('sidebar'),
            themeToggle: document.getElementById('themeToggle'),
            settingsModal: document.getElementById('settingsModal'),
            openSettings: document.getElementById('openSettings'),
            closeSettings: document.getElementById('closeSettings'),
            saveSettings: document.getElementById('saveSettings'),
            apiKeyInput: document.getElementById('apiKeyInput'),
            modelInput: document.getElementById('modelInput'),
            systemPromptInput: document.getElementById('systemPromptInput'),
            emptyState: document.getElementById('emptyState')
        };

        // --- Initialization ---
        function init() {
            if (state.isDark) document.documentElement.classList.add('dark');
            els.apiKeyInput.value = state.apiKey;
            els.modelInput.value = state.model;
            els.systemPromptInput.value = state.systemPrompt;
            
            renderHistory();
            els.loader.classList.add('hidden');
            els.app.classList.remove('hidden');
            
            // Auto-resize textarea
            els.userInput.addEventListener('input', function() {
                this.style.height = 'auto';
                this.style.height = (this.scrollHeight) + 'px';
            });
        }

        // --- Actions ---
        const saveState = () => {
            localStorage.setItem('ai_key', state.apiKey);
            localStorage.setItem('ai_model', state.model);
            localStorage.setItem('ai_system', state.systemPrompt);
            localStorage.setItem('ai_chats', JSON.stringify(state.chats));
            localStorage.setItem('ai_theme', state.isDark ? 'dark' : 'light');
        };

        const renderHistory = () => {
            els.chatHistory.innerHTML = '';
            state.chats.forEach(chat => {
                const item = document.createElement('div');
                item.className = `group flex items-center justify-between p-3 rounded-xl cursor-pointer hover:bg-slate-200 dark:hover:bg-slate-800 ${state.currentChatId === chat.id ? 'bg-slate-200 dark:bg-slate-800' : ''}`;
                item.innerHTML = `
                    <div class="flex-1 truncate mr-2" onclick="loadChat('${chat.id}')">
                        <span class="text-sm font-medium">💬 ${chat.title}</span>
                    </div>
                    <button onclick="deleteChat('${chat.id}')" class="opacity-0 group-hover:opacity-100 p-1 hover:text-red-500 text-xs">🗑️</button>
                `;
                els.chatHistory.appendChild(item);
            });
        };

        const createNewChat = () => {
            const id = Date.now().toString();
            state.chats.unshift({ id, title: 'New Conversation', messages: [] });
            state.currentChatId = id;
            saveState();
            renderHistory();
            renderMessages();
            if (window.innerWidth < 768) els.sidebar.classList.add('closed');
        };

        window.loadChat = (id) => {
            state.currentChatId = id;
            renderMessages();
            renderHistory();
            if (window.innerWidth < 768) els.sidebar.classList.add('closed');
        };

        window.deleteChat = (id) => {
            state.chats = state.chats.filter(c => c.id !== id);
            if (state.currentChatId === id) state.currentChatId = null;
            saveState();
            renderHistory();
            renderMessages();
        };

        const renderMessages = () => {
            const chat = state.chats.find(c => c.id === state.currentChatId);
            els.messages.innerHTML = '';
            
            if (!chat || chat.messages.length === 0) {
                els.emptyState.classList.remove('hidden');
                return;
            }
            
            els.emptyState.classList.add('hidden');
            chat.messages.forEach(msg => {
                appendMessageToUI(msg.role, msg.content);
            });
            els.messages.scrollTo(0, els.messages.scrollHeight);
        };

        const appendMessageToUI = (role, content) => {
            const div = document.createElement('div');
            div.className = `flex ${role === 'user' ? 'justify-end' : 'justify-start'} animate-in fade-in slide-in-from-bottom-2`;
            
            const inner = document.createElement('div');
            inner.className = `max-w-[85%] md:max-w-[70%] p-4 rounded-2xl prose dark:prose-invert ${
                role === 'user' 
                ? 'bg-blue-600 text-white rounded-tr-none' 
                : 'bg-slate-100 dark:bg-slate-800 rounded-tl-none'
            }`;
            
            inner.innerHTML = role === 'user' ? content : marked.parse(content);
            div.appendChild(inner);
            els.messages.appendChild(div);
            els.messages.scrollTo(0, els.messages.scrollHeight);
            return inner;
        };

        async function sendMessage() {
            const text = els.userInput.value.trim();
            if (!text || !state.apiKey) {
                if (!state.apiKey) alert("Please set your API Key in Settings!");
                return;
            }

            if (!state.currentChatId) createNewChat();
            
            const chat = state.chats.find(c => c.id === state.currentChatId);
            
            // Set title if first message
            if (chat.messages.length === 0) chat.title = text.substring(0, 30) + (text.length > 30 ? '...' : '');

            // Add user message
            const userMsg = { role: 'user', content: text };
            chat.messages.push(userMsg);
            els.userInput.value = '';
            els.userInput.style.height = 'auto';
            renderMessages();

            // Prepare AI placeholder
            const aiMsgDiv = appendMessageToUI('assistant', '...');
            let fullAiContent = "";

            try {
                const response = await fetch("https://openrouter.ai/api/v1/chat/completions", {
                    method: "POST",
                    headers: {
                        "Authorization": `Bearer ${state.apiKey}`,
                        "Content-Type": "application/json"
                    },
                    body: JSON.stringify({
                        model: state.model,
                        messages: [
                            { role: "system", content: state.systemPrompt },
                            ...chat.messages
                        ],
                        stream: true
                    })
                });

                const reader = response.body.getReader();
                const decoder = new TextDecoder();
                aiMsgDiv.innerHTML = "";

                while (true) {
                    const { done, value } = await reader.read();
                    if (done) break;
                    
                    const chunk = decoder.decode(value);
                    const lines = chunk.split('\n').filter(line => line.trim() !== '');
                    
                    for (const line of lines) {
                        const message = line.replace(/^data: /, '');
                        if (message === '[DONE]') break;
                        try {
                            const parsed = JSON.parse(message);
                            const content = parsed.choices[0].delta.content || "";
                            fullAiContent += content;
                            aiMsgDiv.innerHTML = marked.parse(fullAiContent);
                            els.messages.scrollTo(0, els.messages.scrollHeight);
                        } catch (e) { /* ignore parse errors in stream */ }
                    }
                }
                
                chat.messages.push({ role: 'assistant', content: fullAiContent });
                saveState();
                renderHistory();

            } catch (err) {
                aiMsgDiv.innerHTML = `<span class="text-red-500 italic">Error: ${err.message}. Check your API key and network.</span>`;
            }
        }

        // --- Event Listeners ---
        els.sendBtn.onclick = sendMessage;
        els.userInput.onkeydown = (e) => { if (e.key === 'Enter' && !e.shiftKey) { e.preventDefault(); sendMessage(); } };
        els.newChatBtn.onclick = createNewChat;
        
        els.toggleSidebar.onclick = () => els.sidebar.classList.toggle('closed');
        
        els.themeToggle.onclick = () => {
            state.isDark = !state.isDark;
            document.documentElement.classList.toggle('dark');
            saveState();
        };

        els.openSettings.onclick = () => els.settingsModal.classList.remove('hidden');
        els.closeSettings.onclick = () => els.settingsModal.classList.add('hidden');
        
        els.saveSettings.onclick = () => {
            state.apiKey = els.apiKeyInput.value;
            state.model = els.modelInput.value || 'deepseek/deepseek-r1:free';
            state.systemPrompt = els.systemPromptInput.value;
            saveState();
            els.settingsModal.classList.add('hidden');
        };

        // Initialize App
        init();
    </script>
</body>
</html>
