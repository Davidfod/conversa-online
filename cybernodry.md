<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cyber Chat 3.0</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-database.js"></script>

    <style>
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

        * { margin: 0; padding: 0; box-sizing: border-box; font-family: 'Segoe UI', BlinkMacSystemFont, sans-serif; }

        body { 
            background: var(--bg-chat); 
            color: var(--text-main); 
            display: flex; 
            height: 100vh; 
            width: 100vw;
            overflow-x: hidden;
            overflow-y: auto; 
            -webkit-overflow-scrolling: touch;
        }

        /* BARRA LATERAL ESQUERDA (CANAIS) */
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
            color: var(--text-muted); 
            text-transform: uppercase; 
            letter-spacing: 1px; 
            font-weight: 700;
            margin-bottom: 8px;
        }

        .channel-list {
            display: flex;
            flex-direction: column;
            gap: 4px;
        }

        .btn-channel {
            width: 100%;
            display: flex;
            align-items: center;
            gap: 8px;
            background: transparent;
            border: none;
            color: var(--text-main);
            padding: 8px;
            border-radius: 4px;
            cursor: pointer;
            font-size: 0.95rem;
            text-align: left;
        }

        .btn-channel:hover, .btn-channel.active {
            background: rgba(79, 84, 92, 0.32);
            color: var(--text-white);
        }

        .btn-channel.active {
            background: rgba(79, 84, 92, 0.6);
            font-weight: bold;
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
            flex-shrink: 0;
        }

        .chat-messages { 
            flex: 1; 
            padding: 20px 16px; 
            overflow-y: auto; 
            display: flex; 
            flex-direction: column; 
            gap: 16px; 
            -webkit-overflow-scrolling: touch;
        }

        /* MENSAGENS E RESPOSTAS */
        .message { 
            display: flex; 
            gap: 16px; 
            padding: 6px 8px;
            border-radius: 4px;
            position: relative;
        }

        .message:hover { background: rgba(4, 4, 5, 0.07); }
        .message img { width: 40px; height: 40px; border-radius: 50%; object-fit: cover; flex-shrink: 0; }
        .message-content { display: flex; flex-direction: column; justify-content: center; width: 100%; }
        .message .user-name { font-size: 0.95rem; color: var(--text-white); font-weight: 500; margin-bottom: 4px; }

        .message .reply-tag {
            font-size: 0.75rem;
            color: var(--accent);
            margin-bottom: 2px;
            font-style: italic;
        }

        .message .user-text { font-size: 0.95rem; color: var(--text-main); white-space: pre-wrap; word-break: break-word; }

        .btn-reply-msg {
            position: absolute;
            right: 10px;
            top: 5px;
            background: var(--bg-card);
            border: 1px solid var(--border-color);
            color: var(--text-main);
            padding: 2px 8px;
            border-radius: 4px;
            font-size: 0.75rem;
            cursor: pointer;
            display: none;
        }

        .message:hover .btn-reply-msg { display: block; }

        /* INPUT DE TEXTO E CAIXA DE REPLICAÇÃO */
        .chat-input-container { padding: 0 16px 24px 16px; background: var(--bg-chat); flex-shrink: 0; }

        .reply-preview-box {
            background: rgba(0,0,0,0.2);
            padding: 6px 12px;
            border-radius: 4px 4px 0 0;
            font-size: 0.8rem;
            display: none;
            justify-content: space-between;
            color: var(--accent);
            border-left: 3px solid var(--accent);
        }

        .chat-input-form { display: flex; flex-direction: column; background: var(--bg-input); border-radius: 8px; }
        .chat-input { flex: 1; background: transparent; border: none; padding: 12px 16px; color: var(--text-white); outline: none; font-size: 0.95rem; }

        /* BARRA LATERAL DIREITA (PERFIL) */
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
        }

        .profile-card { background: var(--bg-card); padding: 20px 16px; border-radius: 8px; text-align: center; }
        .profile-card img { width: 80px; height: 80px; border-radius: 50%; object-fit: cover; margin-bottom: 12px; border: 2px solid var(--accent); }
        .profile-card h3 { font-size: 1.1rem; color: var(--text-white); font-weight: 600; }

        .edit-profile-box { display: flex; flex-direction: column; gap: 16px; }
        .edit-profile-box label { font-size: 0.7rem; color: var(--text-muted); text-transform: uppercase; font-weight: 700; display: block; margin-bottom: 4px; }
        .edit-profile-box input { width: 100%; background: var(--bg-card); border: 1px solid rgba(0,0,0,0.3); padding: 10px; border-radius: 4px; color: var(--text-main); outline: none; }
        .btn-save { background: #5865F2; color: var(--text-white); border: none; padding: 10px; border-radius: 4px; font-weight: 600; cursor: pointer; }

        @media (max-width: 768px) {
            body { flex-direction: column; height: auto; }
            .sidebar-left, .sidebar-right { width: 100%; }
            .chat-area { height: 60vh; }
        }

        /* Customização de barras de rolagem */
        ::-webkit-scrollbar { width: 8px; height: 8px; }
        ::-webkit-scrollbar-track { background: var(--bg-channels); }
        ::-webkit-scrollbar-thumb { background: #202225; border-radius: 4px; }
        ::-webkit-scrollbar-thumb:hover { background: #111214; }
    </style>
</head>
<body>

    <div class="sidebar-left">
        <div>
            <h2>Canais de Texto</h2>
            <div class="channel-list">
                <button class="btn-channel active" onclick="switchChannel('chat-geral')"><i class="fa-solid fa-hashtag"></i> chat-geral</button>
                <button class="btn-channel" onclick="switchChannel('comandos')"><i class="fa-solid fa-hashtag"></i> comandos</button>
                <button class="btn-channel" onclick="switchChannel('midia')"><i class="fa-solid fa-hashtag"></i> midia</button>
            </div>
        </div>
        <div class="version-text">SISTEMA PRIVADO 3.0</div>
    </div>

    <div class="chat-area">
        <div class="chat-header" id="chatHeader">
            <i class="fa-solid fa-hashtag"></i> chat-geral
        </div>
        <div class="chat-messages" id="chatMessages"></div>
        <div class="chat-input-container">
            <div class="reply-preview-box" id="replyBox">
                <span id="replyText">Repondendo a...</span>
                <i class="fa-solid fa-xmark" style="cursor:pointer;" onclick="cancelReply()"></i>
            </div>
            <form class="chat-input-form" onsubmit="sendMessage(event)">
                <input type="text" id="messageInput" class="chat-input" placeholder="Conversar no chat" autocomplete="off" maxlength="500">
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
                <label>Link da Foto ou GIF (URL)</label>
                <input type="text" id="inputAvatar" placeholder="Cole o link .gif aqui">
            </div>
            <button class="btn-save" onclick="updateProfile()">Salvar Alterações</button>
        </div>
    </div>

    <script>
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
        
        let currentChannel = 'chat-geral';
        let messagesRef = database.ref('messages/' + currentChannel);

        const DEFAULT_AVATAR = 'https://i.gifer.com/ZZ5H.gif'; // Avatar padrão em GIF animado
        let currentUser = { name: '', avatar: '' };
        let selectedReplyUser = null;

        function initUser() {
            const savedName = localStorage.getItem('cyber_name');
            const savedAvatar = localStorage.getItem('cyber_avatar');
            currentUser.name = savedName || "CyberUser_" + Math.floor(Math.random() * 900 + 100);
            currentUser.avatar = savedAvatar || DEFAULT_AVATAR;
            
            document.getElementById('currentName').innerText = currentUser.name;
            document.getElementById('currentAvatar').src = currentUser.avatar;
            document.getElementById('inputName').value = currentUser.name;
            document.getElementById('inputAvatar').value = savedAvatar || '';
        }
        initUser();

        function switchChannel(channelName) {
            currentChannel = channelName;
            document.getElementById('chatHeader').innerHTML = `<i class="fa-solid fa-hashtag"></i> ${channelName}`;
            document.querySelectorAll('.btn-channel').forEach(btn => btn.classList.remove('active'));
            event.currentTarget.classList.add('active');
            
            document.getElementById('chatMessages').innerHTML = '';
            messagesRef.off();
            messagesRef = database.ref('messages/' + currentChannel);
            listenMessages();
        }

        function listenMessages() {
            messagesRef.limitToLast(50).on('child_added', (snapshot) => {
                const data = snapshot.val();
                const chatMessages = document.getElementById('chatMessages');
                const messageDiv = document.createElement('div');
                messageDiv.classList.add('message');

                const img = document.createElement('img');
                // Força o carregamento correto do link de imagem ou GIF
                img.src = data.avatar || DEFAULT_AVATAR;

                const contentDiv = document.createElement('div');
                contentDiv.classList.add('message-content');

                if (data.replyTo) {
                    const replyDiv = document.createElement('div');
                    replyDiv.classList.add('reply-tag');
                    replyDiv.textContent = `↳ Respondendo a @${data.replyTo}`;
                    contentDiv.appendChild(replyDiv);
                }

                const nameDiv = document.createElement('div');
                nameDiv.classList.add('user-name');
                nameDiv.textContent = data.name;

                const textDiv = document.createElement('div');
                textDiv.classList.add('user-text');
                textDiv.textContent = data.text;

                const replyBtn = document.createElement('button');
                replyBtn.classList.add('btn-reply-msg');
                replyBtn.textContent = 'Responder';
                replyBtn.onclick = () => setReply(data.name);

                contentDiv.appendChild(nameDiv);
                contentDiv.appendChild(textDiv);
                messageDiv.appendChild(img);
                messageDiv.appendChild(contentDiv);
                messageDiv.appendChild(replyBtn);
                chatMessages.appendChild(messageDiv);
                chatMessages.scrollTop = chatMessages.scrollHeight;
            });
        }
        listenMessages();

        function setReply(username) {
            selectedReplyUser = username;
            document.getElementById('replyBox').style.display = 'flex';
            document.getElementById('replyText').textContent = `Respondendo a @${username}`;
            document.getElementById('messageInput').focus();
        }

        function cancelReply() {
            selectedReplyUser = null;
            document.getElementById('replyBox').style.display = 'none';
        }

        function updateProfile() {
            const nameInput = document.getElementById('inputName').value.trim();
            const avatarInput = document.getElementById('inputAvatar').value.trim();
            if (nameInput !== "") currentUser.name = nameInput;
            if (avatarInput !== "") currentUser.avatar = avatarInput;

            localStorage.setItem('cyber_name', currentUser.name);
            localStorage.setItem('cyber_avatar', currentUser.avatar);
            
            document.getElementById('currentName').innerText = currentUser.name;
            
            // Recarrega o elemento src para garantir o play do GIF animado na hora
            const avatarElement = document.getElementById('currentAvatar');
            avatarElement.src = '';
            avatarElement.src = currentUser.avatar;
        }

        function sendMessage(event) {
            event.preventDefault();
            const input = document.getElementById('messageInput');
            const text = input.value.trim();
            if (text !== '') {
                messagesRef.push({
                    name: currentUser.name,
                    avatar: currentUser.avatar,
                    text: text,
                    replyTo: selectedReplyUser
                });
                input.value = '';
                cancelReply();
            }
        }
    </script>
</body>
</html>
