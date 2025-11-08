<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>AG ANIME — اختبار متجاوب</title>
<style>
  :root{
    --bg:#000; --text:#fff; --muted:#bdbdbd; --accent:#ff3b3b; --panel:#0b0b0b;
  }
  *{box-sizing:border-box;margin:0;padding:0}
  body{font-family:Arial,system-ui; background:var(--bg); color:var(--text); padding:12px; -webkit-font-smoothing:antialiased}
  .wrap{max-width:1100px;margin:0 auto;display:grid;grid-template-rows:auto 1fr auto;gap:12px}
  header{display:flex;flex-wrap:wrap;gap:10px;align-items:center;justify-content:space-between}
  .brand{display:flex;gap:10px;align-items:center}
  .logo{width:56px;height:56px;border-radius:10px;background:linear-gradient(45deg,#222,#111);display:grid;place-items:center}
  nav{display:flex;gap:8px;flex-wrap:wrap}
  nav a{color:var(--muted);text-decoration:none;padding:6px 10px;border-radius:8px;background:transparent}
  nav a.active{background:rgba(255,59,59,0.12);color:var(--text)}

  .content{display:grid;grid-template-columns:2fr 360px;gap:12px}
  .panel{background:#0a0a0a;padding:14px;border-radius:12px}
  .grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(200px,1fr));gap:10px}
  .card{display:flex;gap:10px;align-items:center;padding:10px;background:#0f0f10;border-radius:10px}
  .thumb{width:84px;height:84px;background:linear-gradient(45deg,#333,#222);border-radius:8px;display:grid;place-items:center;font-size:12px;color:var(--muted)}
  .meta h3{margin:0 0 6px 0;font-size:16px}
  .meta p{margin:0;color:var(--muted);font-size:13px}
  .actions{display:flex;flex-direction:column;gap:6px}
  .btn{padding:8px 10px;border-radius:8px;background:transparent;border:1px solid rgba(255,255,255,0.04);color:var(--muted);cursor:pointer}
  .btn.primary{background:var(--accent);color:#111;border:0}

  /* sidebar bot */
  .bot{background:#0d0d0d;padding:10px;border-radius:10px}
  .bot-messages{height:180px;overflow:auto;padding:8px;display:flex;flex-direction:column;gap:8px}
  .msg.user{align-self:flex-end;background:rgba(255,255,255,0.03);padding:8px;border-radius:8px}
  .msg.bot{align-self:flex-start;background:rgba(255,59,59,0.06);padding:8px;border-radius:8px}

  /* modal */
  .modal{position:fixed;inset:0;display:none;place-items:center;background:rgba(0,0,0,0.6)}
  .modal.open{display:grid}
  .modal-card{background:#080808;padding:14px;border-radius:10px;max-width:760px;width:92%}

  footer{text-align:center;color:var(--muted);padding:8px 0;font-size:13px}

  @media (max-width:900px){
    .content{grid-template-columns:1fr}
  }
</style>
</head>
<body>
  <div class="wrap">
    <header>
      <div class="brand">
        <div class="logo">AG</div>
        <div>
          <div style="font-weight:700">AG ANIME — موسوعة ون بيس</div>
          <div style="font-size:13px;color:var(--muted)">اختبار نسخة متجاوبة</div>
        </div>
      </div>

      <nav>
        <a href="#home" class="active" data-target="home">الرئيسية</a>
        <a href="#characters" data-target="characters">الشخصيات</a>
        <a href="#maps" data-target="maps">الخرائط</a>
      </nav>
    </header>

    <main class="content" id="home">
      <section class="panel">
        <h2 style="color:var(--accent);margin:0 0 8px 0">شخصيات بارزة</h2>
        <div id="charactersGrid" class="grid"></div>
      </section>

      <aside class="panel">
        <h3 style="color:var(--accent);margin:0 0 8px 0">AG-BOT</h3>
        <div class="bot">
          <div id="botMessages" class="bot-messages" aria-live="polite"></div>
          <div style="display:flex;gap:8px;margin-top:8px">
            <input id="botInput" placeholder="اسأل AG-BOT..." style="flex:1;padding:8px;border-radius:8px;border:1px solid #222;background:transparent;color:var(--text)">
            <button id="botSend" class="btn primary">اسأل</button>
          </div>
        </div>
      </aside>
    </main>

    <footer>© اختبار — AG ANIME</footer>
  </div>

  <!-- modal -->
  <div id="modal" class="modal" onclick="closeModal(event)">
    <div class="modal-card" onclick="event.stopPropagation()">
      <div style="display:flex;justify-content:space-between;align-items:center">
        <h3 id="modalTitle">عنوان</h3>
        <button class="btn" onclick="closeModal()">إغلاق</button>
      </div>
      <p id="modalDesc" style="color:var(--muted)"></p>
    </div>
  </div>

<script>
  console.log('Script loaded — debug start');

  // مُبسّط DB
  const DB = {
    characters: [
      {id:'luffy', name:'مونكي دي لوفي', role:'قائد', desc:'قائد الطاقم.'},
      {id:'zoro', name:'رورونوا زورو', role:'سياف', desc:'سياف الطاقم.'},
      {id:'nami', name:'نامي', role:'الملاح', desc:'ملاح الطاقم.'},
      {id:'sanji', name:'سانجي', role:'الطباخ', desc:'طباخ الطاقم.'}
    ]
  };

  // عناصر DOM
  const charactersGrid = document.getElementById('charactersGrid');
  const botMessages = document.getElementById('botMessages');
  const botInput = document.getElementById('botInput');
  const botSend = document.getElementById('botSend');
  const modal = document.getElementById('modal');
  const modalTitle = document.getElementById('modalTitle');
  const modalDesc = document.getElementById('modalDesc');

  // render
  function makeCard(ch){
    const el = document.createElement('div');
    el.className = 'card';
    el.innerHTML = `
      <div class="thumb">IMG</div>
      <div class="meta">
        <h3>${ch.name}</h3>
        <p>${ch.role}</p>
      </div>
      <div class="actions">
        <button class="btn" onclick='openModal("${ch.id}")'>عرض</button>
      </div>
    `;
    return el;
  }
  function renderHome(){
    charactersGrid.innerHTML = '';
    DB.characters.forEach(c => charactersGrid.appendChild(makeCard(c)));
    console.log('Rendered characters:', DB.characters.length);
  }

  // modal
  function openModal(id){
    const d = DB.characters.find(x=>x.id===id);
    if(!d) return alert('لا توجد بيانات');
    modalTitle.textContent = d.name;
    modalDesc.textContent = d.desc;
    modal.classList.add('open');
    console.log('Open modal for', id);
  }
  function closeModal(e){
    if(e && e.stopPropagation) e.stopPropagation();
    modal.classList.remove('open');
  }

  // simple bot KB
  const BOT_KB = [
    {q:['لوفي','luffy'], a:'لوفي قائد طاقم قبعة القش.'},
    {q:['زورو'], a:'زورو سياف شجاع.'},
    {q:['نامي'], a:'نامي الملاح.'}
  ];
  function botQuery(text){
    const t = text.trim().toLowerCase();
    for(const item of BOT_KB){
      for(const k of item.q) if(t.includes(k)) return item.a;
    }
    return 'عذراً، لم أعثر على إجابة.';
  }

  // bot handlers
  botSend.addEventListener('click', ()=>{
    const q = botInput.value.trim();
    if(!q) return;
    addBotMsg('user', q);
    setTimeout(()=> {
      const ans = botQuery(q);
      addBotMsg('bot', ans);
    }, 200);
    botInput.value = '';
  });
  botInput.addEventListener('keydown', (e)=> { if(e.key === 'Enter') botSend.click(); });

  function addBotMsg(kind, text){
    const d = document.createElement('div');
    d.className = 'msg ' + (kind==='user'?'user':'bot');
    d.textContent = text;
    botMessages.appendChild(d);
    botMessages.scrollTop = botMessages.scrollHeight;
  }

  // init
  renderHome();
  document.addEventListener('keydown', e => { if(e.key === 'Escape') closeModal(); });

  console.log('Init done — ready.');
</script>
</body>
</html>
