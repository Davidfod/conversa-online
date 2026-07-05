<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cyber Chat & Call 3.0</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-database.js"></script>

    <style>
        /* Cores baseadas no tema Dark do Discord */
        :root {
            --bg-channels: #2f3136;
            --bg-chat: #36393f;
            --bg-profile: #2f3136;
            --bg-card: #202225;
            --bg-input: #40444b;
            --accent: #00ffcc;
            --text-main: #dcddde;
            --text-white: #ffffff;
            --text-muted: #72767d;
            --border-color: rgba(0, 0, 0, 0.2);
        }

        * { 
            margin: 0; 
            padding: 0; 
            box-sizing: border-box; 
            font-family: 'Segoe UI', BlinkMacSystemFont, sans-serif; 
        }

        /* CORREÇÃO AQUI: Permitir rolagem no corpo da página se necessário, ótimo para celular */
        body { 
            background: var(--bg-chat); 
            color: var(--text-main); 
            display: flex; 
            height: 100vh; 
            width: 100vw;
            overflow-x: hidden;
            overflow-y: auto; 
            -webkit-overflow-scrolling: touch; /* Deixa o deslizar do dedo suave no iPhone/Android */
        }

        /* BARRA LATERAL ESQUERDA */
        .sidebar-left { 
            width: 240px; 
            background: var(--bg-channels); 
            display: flex; 
            flex-direction: column; 
            padding: 20px 12px; 
            justify-content: space-between;
            flex-shrink: 0;
            overflow-y: auto;
        }

        .sidebar-left h2 { 
            font-size: 0.75rem; 
            margin-bottom: 12px; 
            color: var(--text-muted); 
            text-transform: uppercase; 
            letter-spacing: 1px; 
            font-weight: 700;
        }

        .btn-call { 
            width: 100%; 
            display: flex; 
            align-items: center; 
            gap: 12px; 
            background: transparent; 
            border: none; 
            color: var(--text-main); 
            padding: 10px 8px; 
            border-radius: 6px; 
            cursor: pointer; 
            font-weight: 500; 
            font-size: 0.95rem;
            transition: background 0.2s; 
            text-align: left;
        }

        .btn-call i { 
            font-size: 1.1rem;
            color: var(--text-muted);
            width: 24px;
            text-align: center;
        }

        .btn-call:hover {
            background: rgba(79, 84, 92, 0.32);
            color: var(--text-white);
        }

        .btn-call.active { 
            background: rgba(79, 84, 92, 0.6); 
            color: var(--accent);
        }

        .version-text { 
            font-size: 0.7rem; 
            color: var(--text-muted); 
            text-align: center;
            letter-spacing: 1px;
            margin-top: 20px;
        }

        /* ÁREA CENTRAL DO CHAT */
        .chat-area { 
            flex: 1; 
            display: flex; 
            flex-direction: column; 
            background: var(--bg-chat); 
            min-width: 320px;
        }

        .chat-header { 
            height: 48px;
            padding: 0 16px; 
            background: var(--bg-chat); 
            border-bottom: 1px solid var(--border-color); 
            font-weight: bold; 
            font-size: 1rem; 
            display: flex;
            align-items: center;
            gap: 8px;
            color: var(--text-white);
            box-shadow: 0 1px 2px rgba(0,0,0,0.2);
            flex-shrink: 0;
        }

        /* CORREÇÃO AQUI: Ativado rolagem total por toque/mouse nas mensagens */
        .chat-messages { 
            flex: 1; 
            padding: 20px 16px; 
            overflow-y: auto; 
            display: flex; 
            flex-direction: column; 
            gap: 16px; 
            -webkit-overflow-scrolling: touch;
        }

        .message { 
            display: flex; 
            gap: 16px; 
            padding: 4px 8px;
            border-radius: 4px;
        }

        .message:hover {
            background: rgba(4, 4, 5, 0.07);
        }

        .message img { 
            width: 40px; 
            height: 40px; 
            border-radius: 50%; 
            object-fit: cover; 
            flex-shrink: 0;
        }

        .message-content { 
            display: flex; 
            flex-direction: column; 
            justify-content: center;
        }

        .message .user-name { 
            font-size: 0.95rem; 
            color: var(--text-white); 
            font-weight: 500; 
            margin-bottom: 4px;
        }

        .message .user-text { 
            font-size: 0.95rem; 
            color: var(--text-main);
            white-space: pre-wrap; 
            word-break: break-word; 
            line-height: 1.375;
        }

        .chat-input-container { 
            padding: 0 16px 24px 16px; 
            background: var(--bg-chat);
            flex-shrink: 0;
        }

        .chat-input-form { 
            display: flex; 
            background: var(--bg-input);
            border-radius: 8px;
            padding: 2px 4px;
        }

        .chat-input { 
            flex: 1; 
            background: transparent;
            border: none;
            padding: 12px 16px; 
            color: var(--text-white); 
            outline: none; 
            font-size: 0.95rem; 
        }

        /* BARRA LATERAL DIREITA (PERFIL) */
        /* CORREÇÃO AQUI: Rolagem total habilitada para deslizar o dedo ou usar scroll do mouse */
        .sidebar-right { 
            width: 280px; 
            background: var(--bg-profile); 
            border-left: 1px solid var(--border-color); 
            padding: 20px 16px; 
            display: flex; 
            flex-direction: column; 
            gap: 20px; 
            overflow-y: auto; 
            flex-shrink: 0;
            -webkit-overflow-scrolling: touch;
        }

        @media (max-width: 768px) {
            body { flex-direction: column; height: auto; overflow-y: auto; }
            .sidebar-left, .sidebar-right { width: 100%; height: auto; }
            .chat-area { height: 60vh; }
        }

        .sidebar-right h2 {
            font-size: 0.75rem;
            color: var(--text-muted);
            text-transform: uppercase;
            letter-spacing: 0.5px;
            font-weight: 700;
        }

        .profile-card { 
            background: var(--bg-card); 
            padding: 20px 16px; 
            border-radius: 8px; 
            text-align: center; 
        }

        .profile-card img { 
            width: 80px; 
            height: 80px; 
            border-radius: 50%; 
            object-fit: cover; 
            margin-bottom: 12px; 
            border: 2px solid var(--accent); 
        }

        .profile-card h3 { 
            font-size: 1.1rem; 
            color: var(--text-white); 
            font-weight: 600;
        }

        .edit-profile-box { 
            display: flex; 
            flex-direction: column; 
            gap: 16px; 
        }

        .edit-profile-box label { 
            font-size: 0.7rem; 
            color: var(--text-muted); 
            text-transform: uppercase; 
            font-weight: 700; 
            margin-bottom: 6px;
            display: block;
        }

        .edit-profile-box input { 
            width: 100%;
            background: var(--bg-card); 
            border: 1px solid rgba(0,0,0,0.3); 
            padding: 10px; 
            border-radius: 4px; 
            color: var(--text-main); 
            outline: none; 
            font-size: 0.9rem; 
        }

        .edit-profile-box input:focus {
            border-color: var(--accent);
        }

        .btn-save { 
            background: #5865F2; 
            color: var(--text-white); 
            border: none; 
            padding: 10px; 
            border-radius: 4px; 
            font-weight: 600; 
            cursor: pointer; 
            transition: background 0.2s;
            font-size: 0.9rem;
        }

        .btn-save:hover { 
            background: #4752c4;
        }

        /* Customização fina de barras de rolagem */
        ::-webkit-scrollbar { width: 8px; height: 8px; }
        ::-webkit-scrollbar-track { background: var(--bg-channels); }
        ::-webkit-scrollbar-thumb { background: #202225; border-radius: 4px; }
        ::-webkit-scrollbar-thumb:hover { background: #111214; }
    </style>
</head>
<body>

    <div class="sidebar-left">
        <div>
            <h2>Canais de Voz</h2>
            <button class="btn-call" id="callBtn" onclick="toggleCall()">
                <i class="fa-solid fa-volume-high"></i>
                <span>Call 1</span>
            </button>
        </div>
        <div class="version-text">SISTEMA PRIVADO 3.0</div>
    </div>

    <div class="chat-area">
        <div class="chat-header">
            <i class="fa-solid fa-hashtag"></i> <span>chat-geral</span>
        </div>
        <div class="chat-messages" id="chatMessages"></div>
        <div class="chat-input-container">
            <form class="chat-input-form" onsubmit="sendMessage(event)">
                <input type="text" id="messageInput" class="chat-input" placeholder="Conversar em #chat-geral" autocomplete="off" maxlength="500">
            </form>
        </div>
    </div>

    <div class="sidebar-right">
        <h2>Membro da Sala</h2>
        <div class="profile-card">
            <img id="currentAvatar" src="" alt="Sua Foto">
            <h3 id="currentName">Carregando...</h3>
        </div>
        <div class="edit-profile-box">
            <div>
                <label>Nome de Usuário</label>
                <input type="text" id="inputName" maxlength="25">
            </div>
            <div>
                <label>Link da Foto (URL)</label>
                <input type="text" id="inputAvatar" placeholder="https://...">
            </div>
            <button class="btn-save" onclick="updateProfile()">Salvar Alterações</button>
        </div>
    </div>

    <script>
        // Suas configurações oficiais do Firebase mantidas intactas
        const firebaseConfig = {
            apiKey: "AIzaSyDegW2sEjWvx3Ih7U6vs45Gp8PeXyfJ90w",
            authDomain: "cyber-a41d9.firebaseapp.com",
            projectId: "cyber-a41d9",
            storageBucket: "cyber-a41d9.appspot.com",
            messagingSenderId: "563443982918",
            appId: "1:563443982918:web:61cf4b88f1b4ddd89ee047",
            databaseURL: "https://cyber-a41d9-default-rtdb.firebaseio.com/"
        };
        
        firebase.initializeApp(firebaseConfig);
        const database = firebase.database();
        const messagesRef = database.ref('messages');

        const DEFAULT_AVATAR = 'https://picsum.photos/id/64/150/150';
        let localStream = null;
        let currentUser = { name: '', avatar: '' };

        function initUser() {
            const savedName = localStorage.getItem('cyber_name');
            const savedAvatar = localStorage.getItem('cyber_avatar');
            currentUser.name = savedName || "CyberUser_" + Math.floor(Math.random() * 900 + 100);
            currentUser.avatar = savedAvatar || `https://picsum.photos/id/${Math.floor(Math.random() * 50) + 10}/150/150`;
            
            document.getElementById('currentName').innerText = currentUser.name;
            document.getElementById('currentAvatar').src = currentUser.avatar;
            document.getElementById('inputName').value = currentUser.name;
            document.getElementById('inputAvatar').value = savedAvatar || '';
        }
        initUser();

        messagesRef.limitToLast(50).on('child_added', (snapshot) => {
            const data = snapshot.val();
            const chatMessages = document.getElementById('chatMessages');
            const messageDiv = document.createElement('div');
            messageDiv.classList.add('message');

            const img = document.createElement('img');
            img.src = data.avatar || DEFAULT_AVATAR;
            img.onerror = function() { this.src = DEFAULT_AVATAR; };

            const contentDiv = document.createElement('div');
            contentDiv.classList.add('message-content');

            const nameDiv = document.createElement('div');
            nameDiv.classList.add('user-name');
            nameDiv.textContent = data.name;

            const textDiv = document.createElement('div');
            textDiv.classList.add('user-text');
            textDiv.textContent = data.text;

            contentDiv.appendChild(nameDiv);
            contentDiv.appendChild(textDiv);
            messageDiv.appendChild(img);
            messageDiv.appendChild(contentDiv);
            chatMessages.appendChild(messageDiv);
            chatMessages.scrollTop = chatMessages.scrollHeight;
        });

        function updateProfile() {
            const nameInput = document.getElementById('inputName').value.trim();
            const avatarInput = document.getElementById('inputAvatar').value.trim();
            if (nameInput !== "") currentUser.name = nameInput;
            if (avatarInput !== "") currentUser.avatar = avatarInput;

            localStorage.setItem('cyber_name', currentUser.name);
            localStorage.setItem('cyber_avatar', avatarInput);
            document.getElementById('currentName').innerText = currentUser.name;
            document.getElementById('currentAvatar').src = currentUser.avatar;
        }

        function sendMessage(event) {
            event.preventDefault();
            const input = document.getElementById('messageInput');
            const text = input.value.trim();
            if (text !== '') {
                messagesRef.push({
                    name: currentUser.name,
                    avatar: currentUser.avatar,
                    text: text
                });
                input.value = '';
            }
        }

        async function toggleCall() {
            const btn = document.getElementById('callBtn');
            if (!btn.classList.contains('active')) {
                try {
                    localStream = await navigator.mediaDevices.getUserMedia({ audio: true });
                    btn.classList.add('active');
                    btn.querySelector('span').innerText = 'Em Call (Ativo)';
                } catch (err) {
                    alert('Permissão de áudio recusada.');
                }
            } else {
                if (localStream) {
                    localStream.getTracks().forEach(track => track.stop());
                    localStream = null;
                }
                btn.classList.remove('active');
                btn.querySelector('span').innerText = 'Call 1';
            }
        }
    </script>
</body>
</html>
