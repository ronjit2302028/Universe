# Universe
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Messaging App UI - Demo</title>
  <style>
    :root{
      --bg:#0f1724;
      --panel:#0b1220;
      --muted:#9aa4b2;
      --accent:#4f46e5;
      --msg-send:#0ea5a5;
      --glass: rgba(255,255,255,0.03);
      --radius:12px;
      font-family: Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
    }
    *{box-sizing:border-box}
    body{margin:0;background:linear-gradient(180deg,#071028 0%, var(--bg) 100%);color:#e6eef6;min-height:100vh}

    .app{
      width:1100px;max-width:98%;height:80vh;margin:36px auto;border-radius:16px;overflow:hidden;display:flex;box-shadow:0 10px 40px rgba(2,6,23,0.7)
    }

    /* LEFT: contacts */
    .sidebar{
      width:320px;background:linear-gradient(180deg,var(--panel), rgba(15,23,36,0.85));padding:18px;display:flex;flex-direction:column;gap:12px
    }
    .brand{display:flex;align-items:center;gap:10px}
    .logo{width:44px;height:44px;border-radius:10px;background:linear-gradient(135deg,var(--accent),#06b6d4);display:flex;align-items:center;justify-content:center;font-weight:700}
    .brand h1{font-size:16px;margin:0}
    .search{background:var(--glass);padding:10px;border-radius:10px;color:var(--muted);display:flex;align-items:center;gap:8px}
    .search input{background:transparent;border:0;outline:none;color:inherit;width:100%}

    .contacts{overflow:auto;padding-right:6px}
    .contact{display:flex;align-items:center;gap:12px;padding:10px;border-radius:10px;cursor:pointer}
    .contact:hover{background:rgba(255,255,255,0.02)}
    .avatar{width:48px;height:48px;border-radius:10px;flex:0 0 48px;display:flex;align-items:center;justify-content:center;font-weight:700}
    .meta{flex:1}
    .meta .name{font-weight:600}
    .meta .last{font-size:13px;color:var(--muted);margin-top:4px}
    .badge{font-size:12px;color:var(--muted)}

    /* MAIN: chat area */
    .chat{
      flex:1;background:linear-gradient(180deg, rgba(255,255,255,0.02), transparent);display:flex;flex-direction:column;position:relative;overflow:hidden
    }
    .chat-header{padding:18px;border-bottom:1px solid rgba(255,255,255,0.03);display:flex;align-items:center;gap:12px}
    .chat-header .title{font-weight:700}
    .messages{flex:1;padding:18px 22px;overflow:auto;display:flex;flex-direction:column;gap:12px}

    .msg-row{display:flex;gap:12px;align-items:flex-end}
    .msg-row.you{justify-content:flex-end}
    .bubble{max-width:68%;padding:10px 12px;border-radius:12px;background:rgba(255,255,255,0.03);line-height:1.3}
    .bubble.you{background:linear-gradient(90deg,var(--msg-send), #06b6d4);color:#062021;border-bottom-right-radius:6px}
    .time{font-size:11px;color:var(--muted);margin-top:6px;text-align:right}

    .input-bar{padding:12px;border-top:1px solid rgba(255,255,255,0.03);display:flex;gap:10px;align-items:center}
    .input{flex:1;background:var(--glass);padding:10px;border-radius:999px;display:flex;gap:8px;align-items:center}
    .input textarea{width:100%;border:0;background:transparent;color:inherit;outline:none;resize:none;height:34px;padding:4px}
    .btn{background:linear-gradient(90deg,var(--accent),#06b6d4);border:0;color:white;padding:10px 14px;border-radius:10px;cursor:pointer}

    /* small screens */
    @media (max-width:900px){
      .app{flex-direction:column;height:92vh}
      .sidebar{width:100%;flex-shrink:0;order:1}
      .chat{order:2}
    }
  </style>
</head>
<body>
  <div class="app">
    <aside class="sidebar">
      <div class="brand">
        <div class="logo">M</div>
        <div>
          <h1>MiniChat</h1>
          <div style="font-size:12px;color:var(--muted)">Connected</div>
        </div>
      </div>

      <div class="search"><span>🔎</span><input id="searchInput" placeholder="Search people or messages" /></div>

      <div class="contacts" id="contactList">
        <!-- contacts injected here -->
      </div>
    </aside>

    <main class="chat">
      <div class="chat-header">
        <div style="display:flex;align-items:center;gap:12px">
          <div id="activeAvatar" class="avatar" style="background:linear-gradient(135deg,#8b5cf6,#06b6d4)">A</div>
          <div>
            <div id="activeName" class="title">Pick a conversation</div>
            <div id="activeStatus" style="font-size:13px;color:var(--muted)">Offline</div>
          </div>
        </div>
      </div>

      <div class="messages" id="messages">
        <div style="color:var(--muted);text-align:center;margin-top:60px">No conversation selected — click a contact on the left.</div>
      </div>

      <div class="input-bar">
        <div class="input">
          <textarea id="messageInput" placeholder="Type a message..." rows="1"></textarea>
        </div>
        <button id="sendBtn" class="btn">Send</button>
      </div>
    </main>
  </div>

  <script>
    // Simple mock data
    const contacts = [
      {id:1,name:'Arafat',avatar:'A',last:'Hey, you around?',time:'11:02 AM',messages:[
        {from:'them',text:'Hey, can you review my code?',time:'10:55 AM'},
        {from:'you',text:'Sure — send it here.',time:'10:56 AM'},
        {from:'them',text:'Sending now 👇',time:'11:00 AM'}
      ]},
      {id:2,name:'Bithi',avatar:'B',last:'Got it, thanks!',time:'Yesterday',messages:[
        {from:'them',text:'Thanks for the help!',time:'Yesterday'},
      ]},
      {id:3,name:'Charu',avatar:'C',last:'Let\'s meet',time:'Tue',messages:[
        {from:'they',text:"Let's meet tomorrow",time:'Tue'}
      ]}
    ];

    let activeId = null;

    const contactListEl = document.getElementById('contactList');
    const messagesEl = document.getElementById('messages');
    const activeNameEl = document.getElementById('activeName');
    const activeStatusEl = document.getElementById('activeStatus');
    const activeAvatarEl = document.getElementById('activeAvatar');
    const messageInput = document.getElementById('messageInput');
    const sendBtn = document.getElementById('sendBtn');
    const searchInput = document.getElementById('searchInput');

    function renderContacts(list){
      contactListEl.innerHTML = '';
      list.forEach(c => {
        const div = document.createElement('div');
        div.className = 'contact';
        div.innerHTML = `
          <div class="avatar" style="background:linear-gradient(135deg,#${hashColor(c.name)} , #06b6d4)">${c.avatar}</div>
          <div class="meta">
            <div class="name">${c.name}</div>
            <div class="last">${c.last}</div>
          </div>
          <div class="badge">${c.time}</div>
        `;
        div.onclick = () => openConversation(c.id);
        contactListEl.appendChild(div);
      });
    }

    function openConversation(id){
      activeId = id;
      const convo = contacts.find(c=>c.id===id);
      activeNameEl.textContent = convo.name;
      activeStatusEl.textContent = 'Online';
      activeAvatarEl.textContent = convo.avatar;
      activeAvatarEl.style.background = `linear-gradient(135deg,#${hashColor(convo.name)}, #06b6d4)`;
      renderMessages(convo.messages);
    }

    function renderMessages(msgs){
      messagesEl.innerHTML='';
      msgs.forEach(m=>{
        const row = document.createElement('div');
        const sender = (m.from === 'you') ? 'you' : 'them';
        row.className = 'msg-row ' + (sender==='you' ? 'you' : 'them');
        const bubble = document.createElement('div');
        bubble.className = 'bubble ' + (sender==='you' ? 'you' : '');
        bubble.innerHTML = `<div>${escapeHtml(m.text)}</div><div class="time">${m.time}</div>`;
        row.appendChild(bubble);
        messagesEl.appendChild(row);
      });
      // scroll to bottom
      messagesEl.scrollTop = messagesEl.scrollHeight;
    }

    function sendMessage(){
      const text = messageInput.value.trim();
      if(!text || !activeId) return;
      const convo = contacts.find(c=>c.id===activeId);
      const now = new Date();
      const time = now.toLocaleTimeString([], {hour:'2-digit',minute:'2-digit'});
      convo.messages.push({from:'you',text,time});
      convo.last = text.length>20 ? text.slice(0,20)+"..." : text;
      convo.time = 'Now';
      renderContacts(contacts);
      renderMessages(convo.messages);
      messageInput.value='';
      // auto reply simulation
      setTimeout(()=>{
        convo.messages.push({from:'them',text:'Nice — got it!',time:'Now'});
        renderContacts(contacts);
        if(activeId===convo.id) renderMessages(convo.messages);
      },800 + Math.random()*1000);
    }

    // helpers
    function escapeHtml(unsafe) {
      return unsafe
        .replaceAll('&','&amp;')
        .replaceAll('<','&lt;')
        .replaceAll('>','&gt;')
        .replaceAll('"','&quot;')
        .replaceAll("'","&#039;");
    }

    function hashColor(str){
      // makes a simple hex from name
      let h=0; for(let i=0;i<str.length;i++) h = str.charCodeAt(i) + ((h<<5)-h);
      let color = (h & 0x00FFFFFF).toString(16).toUpperCase();
      return color.padStart(6,'0').slice(0,6);
    }

    // event listeners
    sendBtn.addEventListener('click', sendMessage);
    messageInput.addEventListener('keydown', (e)=>{ if(e.key==='Enter' && !e.shiftKey){ e.preventDefault(); sendMessage(); } });
    searchInput.addEventListener('input', ()=>{
      const q = searchInput.value.toLowerCase();
      renderContacts(contacts.filter(c=>c.name.toLowerCase().includes(q) || c.last.toLowerCase().includes(q)));
    });

    // initial render
    renderContacts(contacts);

    // expose for debugging in console
    window._mini = {contacts,openConversation};
  </script>
</body>
</html>

