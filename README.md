# ai-helper
./
<!doctype html>
<html lang="ru">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>AI Помощник — OpenXI</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <style>
    html,body{height:100%}
    .glass{background:linear-gradient(135deg, rgba(255,255,255,0.06), rgba(255,255,255,0.02)); backdrop-filter: blur(6px);}
    .bubble-user{background:linear-gradient(90deg,#d1fae5,#bbf7d0); color:#042c1f}
    .bubble-assistant{background:linear-gradient(90deg,#eef2ff,#e9d5ff); color:#1f1140}
    .scrollbar-thin::-webkit-scrollbar{height:8px;width:8px}
    .scrollbar-thin::-webkit-scrollbar-thumb{background:rgba(0,0,0,0.15);border-radius:999px}
    .typing-indicator {display:inline-block;width:12px;height:12px;border-radius:50%;background-color:#8b5cf6;margin:0 3px;opacity:0.6;}
    @keyframes pulse {0%,100%{transform:scale(1);opacity:0.6;}50%{transform:scale(1.5);opacity:1;}}
    .typing-indicator:nth-child(1){animation:pulse 1.5s infinite 0s;}
    .typing-indicator:nth-child(2){animation:pulse 1.5s infinite 0.3s;}
    .typing-indicator:nth-child(3){animation:pulse 1.5s infinite 0.6s;}
    .pulse-ring{display:block;width:24px;height:24px;border-radius:50%;background:#ef4444;animation:pulse-ring 1.5s infinite;}
    @keyframes pulse-ring{0%{transform:scale(0.8);opacity:1;}80%,100%{transform:scale(1.5);opacity:0;}}
  </style>
</head>
<body class="h-full bg-gradient-to-br from-slate-900 via-slate-800 to-slate-700 text-slate-100">
  <div class="max-w-7xl mx-auto p-6">

    <header class="flex items-center justify-between mb-6">
      <h1 class="text-2xl font-extrabold">AI Помощник — OpenXI</h1>
      <div class="text-xs text-slate-300">Статус: <span id="status">online</span></div>
    </header>

    <main class="grid grid-cols-12 gap-6">
      <aside class="col-span-3 glass p-4 rounded-2xl shadow-lg flex flex-col gap-4">
        <h3 class="font-semibold">Быстрые вопросы</h3>
        <div class="grid grid-cols-1 gap-2">
          <button class="quick-cmd p-2 rounded-lg bg-slate-800 text-xs" data-cmd="Как получить валюту в OpenXI?">Валюта</button>
          <button class="quick-cmd p-2 rounded-lg bg-slate-800 text-xs" data-cmd="Объясни правила OpenXI.ru">Правила OpenXI</button>
          <button class="quick-cmd p-2 rounded-lg bg-slate-800 text-xs" data-cmd="Объясни OpenXI">Что такое OpenXI</button>
          <button class="quick-cmd p-2 rounded-lg bg-slate-800 text-xs" data-cmd="Объясни мне статьи">Статьи</button>
          <button class="quick-cmd p-2 rounded-lg bg-slate-800 text-xs" data-cmd="Администратор и владелец сайта">Администратор/Владелец</button>
        </div>

        <h3 class="font-semibold mt-4">Голосовой ввод</h3>
        <button id="voice-btn" class="w-full py-2 rounded-lg bg-indigo-500 text-white font-semibold flex items-center justify-center gap-2">
          <span>🎤</span> Говорить
        </button>
        <div id="voice-status" class="text-xs text-slate-300 mt-2 text-center hidden">
          <div class="pulse-ring mx-auto mb-1"></div>Слушаю... Говорите сейчас
        </div>
        <div id="voice-error" class="text-xs text-red-400 mt-2 text-center hidden"></div>
      </aside>

      <section class="col-span-9 bg-gradient-to-b from-slate-800/50 to-slate-900/40 p-4 rounded-2xl shadow-lg flex flex-col">
        <div id="messages" class="flex-1 overflow-y-auto p-3 space-y-3 scrollbar-thin" style="min-height:320px;">
          <div class="flex justify-start">
            <div class="max-w-[75%] p-3 rounded-2xl shadow bubble-assistant text-slate-900">
              <div class="text-sm">Привет! Я AI-помощник OpenXI. Задавайте вопросы о правилах сайтов, получении валюты, работе вашего искусственного интеллекта или запросите статьи.</div>
            </div>
          </div>
        </div>

        <form id="form" class="mt-3 flex gap-3 items-end">
          <textarea id="input" rows="1" placeholder="Спросите что‑нибудь…" class="flex-1 p-3 rounded-2xl bg-slate-900/40 resize-none text-sm"></textarea>
          <button class="px-4 py-2 rounded-2xl bg-indigo-500 hover:bg-indigo-400 font-semibold">Отправить</button>
        </form>
      </section>
    </main>

  </div>

  <script>
    const messagesEl = document.getElementById('messages');
    const inputEl = document.getElementById('input');
    const voiceBtn = document.getElementById('voice-btn');
    const voiceStatus = document.getElementById('voice-status');
    const voiceError = document.getElementById('voice-error');
    let isListening = false;
    let recognition = null;

    if ('SpeechRecognition' in window || 'webkitSpeechRecognition' in window) {
      const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
      recognition = new SpeechRecognition();
      recognition.continuous = false;
      recognition.lang = 'ru-RU';
      recognition.interimResults = false;
      recognition.maxAlternatives = 1;

      recognition.onstart = function() {
        isListening = true;
        voiceStatus.classList.remove('hidden');
        voiceError.classList.add('hidden');
        voiceBtn.innerHTML = '<span>⏹</span> Остановить';
      };

      recognition.onresult = function(event) {
        const transcript = event.results[0][0].transcript;
        inputEl.value = transcript;
        voiceStatus.classList.add('hidden');
        voiceBtn.innerHTML = '<span>🎤</span> Говорить';
        isListening = false;
        sendMessage();
      };

      recognition.onerror = function(event) {
        voiceStatus.classList.add('hidden');
        voiceError.classList.remove('hidden');
        voiceError.textContent = event.error === 'no-speech' ? 'Речь не распознана.' : 'Ошибка: ' + event.error;
        voiceBtn.innerHTML = '<span>🎤</span> Говорить';
        isListening = false;
      };

      recognition.onend = function() {
        if (isListening) {
          voiceStatus.classList.add('hidden');
          voiceBtn.innerHTML = '<span>🎤</span> Говорить';
          isListening = false;
        }
      };
    } else {
      voiceBtn.disabled = true;
      voiceBtn.innerHTML = '<span>🎤</span> Не поддерживается';
      voiceError.classList.remove('hidden');
      voiceError.textContent = 'Ваш браузер не поддерживает голосовой ввод.';
    }

    voiceBtn.addEventListener('click', () => {
      if (!recognition) return;
      if (!isListening) { try { recognition.start(); } catch(e){ console.error(e); } } else { recognition.stop(); }
    });

    inputEl.addEventListener('input', function() { this.style.height = 'auto'; this.style.height = (this.scrollHeight) + 'px'; });

    document.querySelectorAll('.quick-cmd').forEach(btn => { btn.addEventListener('click', () => { inputEl.value = btn.dataset.cmd; sendMessage(); }); });

    document.getElementById('form').addEventListener('submit', e => { e.preventDefault(); sendMessage(); });

    function addBubble(role, text){
      const wrapper = document.createElement('div');
      wrapper.className = role==='user'?'flex justify-end':'flex justify-start';
      wrapper.innerHTML = `<div class="max-w-[75%] p-3 rounded-2xl shadow ${role==='user'?'bubble-user':'bubble-assistant'} text-slate-900"><div class="text-sm">${text}</div></div>`;
      messagesEl.appendChild(wrapper);
      messagesEl.scrollTop = messagesEl.scrollHeight;
    }

    async function sendMessage(){
      const q = inputEl.value.trim();
      if(!q) return;
      addBubble('user', q);
      inputEl.value=''; inputEl.style.height='auto';
      setTimeout(async ()=>{ const answer = await generateAutoReply(q); addBubble('assistant', answer); }, 500);
    }

    async function generateAutoReply(q){
      const lower = q.toLowerCase();
      if(lower.includes('валют')) return 'ESCI — это игровая валюта Random Game 5. Чтобы её получить, выполняйте задания, участвуйте в акциях и выполняйте ежедневные миссии.';
      if(lower.includes('правил')) return 'Правила OpenXI: будьте честны, не нарушайте законы сайта, уважайте других пользователей. Подробнее см. на openxi.ru.';
      if(lower.includes('ai помощник')) return 'AI-помощник объясняет правила, отвечает на вопросы о сайте, валюте и возможностях OpenXI, помогает по русскому и математике.';
      if(lower.includes('openxi')) return 'OpenXI — это сообщество (community) openxi.us, включающее игры, сайты и AI-помощника, который помогает клиентам и объясняет работу искусственного интеллекта.';
      if(lower.includes('статьи')) {
        try {
          const res = await fetch('https://asadbek470.github.io/ban1/');
          const html = await res.text();
          const tempDiv = document.createElement('div');
          tempDiv.innerHTML = html;
          tempDiv.querySelectorAll('script, style').forEach(el => el.remove());
          return tempDiv.innerText || 'Статьи пусты.';
        } catch(e){ return 'Не удалось загрузить статьи. Попробуйте позже.'; }
      }
      if(lower.includes('администратор') || lower.includes('владелец')) return 'Администратор сайта: Абдурауфов Мухаммад Камол. Владелец сайта: Абдумаликов Асадбек.';
      return 'Я AI-помощник OpenXI. Могу объяснить правила сайтов, работу AI, валюту ESCI и OpenXI community. Задайте вопрос подробнее.';
    }
  </script>
</body>
</html>
