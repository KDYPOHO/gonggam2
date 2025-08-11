const $ = (s) => document.querySelector(s);
const $$ = (s) => document.querySelectorAll(s);
const store = {
  get(k, f){ try { return JSON.parse(localStorage.getItem(k)) ?? f; } catch { return f; } },
  set(k, v){ localStorage.setItem(k, JSON.stringify(v)); }
};
(function seed(){
  if (store.get('__seeded', false)) return;
  const notices = [
    {id: crypto.randomUUID(), title:"[�꾩닔] 9/1 �뺢린紐⑥엫 + �� 硫ㅻ쾭 OT", body:"2�숆린 �댁쁺 �뚭컻�� �꾨줈�앺듃 �� 諛곗젙�� 吏꾪뻾�⑸땲��.", ctaText:"李몄꽍/遺덉갭 泥댄겕", ctaUrl:"https://forms.gle/your-form", pinned:true, deadline:"2025-08-28T23:00", createdAt: Date.now(), read:false},
    {id: crypto.randomUUID(), title:"援우쫰 �좎껌 �덈궡", body:"�꾨뱶 吏묒뾽 �됱긽 �ы몴 諛� �ъ씠利� �섏슂 議곗궗", ctaText:"援우쫰 �� �닿린", ctaUrl:"https://forms.gle/your-goods", pinned:false, deadline:null, createdAt: Date.now()-3600*1000, read:false},
  ];
  const schedules = [
    {id: crypto.randomUUID(), at:"2025-09-01T13:00", text:"�뺢린紐⑥엫 (�ㅽ뵆 2痢� �뚯쓽��)"},
    {id: crypto.randomUUID(), at:"2025-09-10T18:00", text:"�ㅽ꽣�� �μ삤��"}
  ];
  store.set('notices', notices);
  store.set('schedules', schedules);
  store.set('moods', []);
  store.set('__seeded', true);
})();

function renderNextEvent(){
  const list = store.get('schedules', [])
    .map(s => ({...s, ts: new Date(s.at || (s.date + 'T' + (s.time||'00:00'))).getTime()}))
    .filter(s => !Number.isNaN(s.ts) && s.ts >= Date.now())
    .sort((a,b)=>a.ts-b.ts);
  const box = $('#nextEvent'); const badge = $('#ddayBadge');
  if (!list.length){ box.textContent="�ㅺ��ㅻ뒗 �쇱젙�� �놁뒿�덈떎."; badge.textContent="D-?"; $('#ctaArea').innerHTML=""; return; }
  const n = list[0]; const d = Math.ceil((n.ts - Date.now())/(1000*60*60*24));
  badge.textContent = d<=0 ? "D�멏AY" : `D��${d}`;
  box.textContent = `${new Date(n.ts).toLocaleString()} �� ${n.text}`;
  const notices = store.get('notices', []).filter(n => n.ctaUrl);
  const hot = notices.sort((a,b)=> (a.deadline?new Date(a.deadline):Infinity) - (b.deadline?new Date(b.deadline):Infinity))[0];
  const ctaArea = $('#ctaArea');
  if (hot){ const urgency = hot.deadline ? ` (留덇컧 ${new Date(hot.deadline).toLocaleDateString()})` : "";
    ctaArea.innerHTML = `<a class="cta" href="${hot.ctaUrl}" target="_blank" rel="noopener">${hot.ctaText||'�닿린'}${urgency}</a>`;
  } else { ctaArea.innerHTML=""; }
}

function renderNotices(){
  const listEl = $('#noticeList');
  const onlyUnread = $('#showOnlyUnread').checked;
  const onlyPinned = $('#showOnlyPinned').checked;
  const items = store.get('notices', [])
    .sort((a,b)=> (b.pinned?-1:0) - (a.pinned?-1:0) || b.createdAt - a.createdAt)
    .filter(n => (onlyUnread ? !n.read : true))
    .filter(n => (onlyPinned ? n.pinned : true));

  listEl.innerHTML = items.map(n=>{
    const dday = n.deadline ? Math.ceil((new Date(n.deadline).getTime() - Date.now())/(1000*60*60*24)) : null;
    const dLabel = (dday!==null) ? (dday<=0 ? 'D�멏AY' : `D��${dday}`) : '';
    const labels = [
      n.pinned ? '<span class="label pin">怨좎젙</span>' : '',
      n.title.includes('[�꾩닔]') ? '<span class="label required">�꾩닔</span>' : '',
      dLabel ? `<span class="label">${dLabel}</span>` : ''
    ].join('');
    const unreadDot = n.read ? '' : '<span class="dot"></span>';
    const cta = n.ctaUrl ? `<a class="cta" href="${n.ctaUrl}" target="_blank" rel="noopener">${n.ctaText||'�닿린'}</a>` : '';
    return `<li class="item" data-id="${n.id}">
      <div class="row"><div>${unreadDot}<strong>${n.title}</strong>${labels}</div></div>
      <div class="meta">${new Date(n.createdAt).toLocaleString()}${n.deadline?` �� 留덇컧 ${new Date(n.deadline).toLocaleString()}`:''}</div>
      <div>${n.body}</div>
      <div class="row" style="margin-top:8px;gap:8px">
        ${cta}
        <button class="ghost sm markread">${n.read?'�쎌쓬':'�쎌� �딆쓬'}</button>
        <button class="ghost sm togglepin">${n.pinned?'怨좎젙 �댁젣':'怨좎젙'}</button>
        <button class="ghost sm del">��젣</button>
      </div>
    </li>`;
  }).join('');

  listEl.querySelectorAll('.item').forEach(li=>{
    li.addEventListener('click', (e)=>{ const id = li.dataset.id; if (e.target.closest('button')) return; markRead(id, true); });
    li.querySelector('.markread')?.addEventListener('click', ()=> markRead(li.dataset.id));
    li.querySelector('.togglepin')?.addEventListener('click', ()=> togglePin(li.dataset.id));
    li.querySelector('.del')?.addEventListener('click', ()=> delNotice(li.dataset.id));
  });
}
function markRead(id, force=false){ const n = store.get('notices', []); const i=n.findIndex(x=>x.id===id); if(i<0)return; n[i].read = force?true:!n[i].read; store.set('notices', n); renderNotices(); }
function togglePin(id){ const n = store.get('notices', []); const i=n.findIndex(x=>x.id===id); if(i<0)return; n[i].pinned = !n[i].pinned; store.set('notices', n); renderNotices(); }
function delNotice(id){ if(!confirm('��젣�좉퉴��?'))return; const n = store.get('notices', []).filter(x=>x.id!==id); store.set('notices', n); renderNotices(); }

$('#noticeForm')?.addEventListener('submit', (e)=>{
  e.preventDefault();
  const n = {
    id: crypto.randomUUID(),
    title: document.getElementById('nTitle').value.trim(),
    body: document.getElementById('nBody').value.trim(),
    ctaText: document.getElementById('nCTA').value.trim(),
    ctaUrl: document.getElementById('nURL').value.trim(),
    pinned: document.getElementById('nPinned').checked,
    deadline: document.getElementById('nDeadline').value? new Date(document.getElementById('nDeadline').value).toISOString() : null,
    createdAt: Date.now(),
    read:false
  };
  if(!n.title || !n.body) return;
  const arr = store.get('notices', []); arr.unshift(n); store.set('notices', arr);
  e.target.reset(); renderNotices(); renderNextEvent();
});
document.getElementById('showOnlyUnread').addEventListener('change', renderNotices);
document.getElementById('showOnlyPinned').addEventListener('change', renderNotices);

function renderSchedules(){
  const ul = document.getElementById('scheduleList');
  const items = store.get('schedules', []).slice().sort((a,b)=> new Date(a.at)-new Date(b.at));
  ul.innerHTML = items.map(s=>`<li class="item"><div><strong>${new Date(s.at).toLocaleString()}</strong> �� ${s.text}</div></li>`).join('');
}
document.getElementById('scheduleForm').addEventListener('submit', (e)=>{
  e.preventDefault();
  const date = document.getElementById('sDate').value, time = document.getElementById('sTime').value || '00:00';
  const text = document.getElementById('sText').value.trim();
  if(!date || !text) return;
  const at = new Date(`${date}T${time}`).toISOString();
  const arr = store.get('schedules', []); arr.push({id: crypto.randomUUID(), at, text}); store.set('schedules', arr);
  e.target.reset(); renderSchedules(); renderNextEvent();
});

function renderMoods(){
  const ul = document.getElementById('moodList');
  const items = store.get('moods', []);
  ul.innerHTML = items.map(m=>`<li class="item"><div>${m.emoji} ${m.text||''}</div><div class="meta">${new Date(m.at).toLocaleString()}</div></li>`).join('');
}
document.getElementById('moodForm').addEventListener('submit', (e)=>{
  e.preventDefault();
  const emoji = document.getElementById('moodEmoji').value;
  const text = document.getElementById('moodText').value.trim();
  const arr = store.get('moods', []); arr.unshift({emoji, text, at: Date.now()}); store.set('moods', arr);
  document.getElementById('moodText').value=''; renderMoods();
});

let deferredPrompt; const installBtn = document.getElementById('installBtn');
window.addEventListener('beforeinstallprompt', (e)=>{ e.preventDefault(); deferredPrompt=e; installBtn.hidden=false; });
installBtn.addEventListener('click', async()=>{ if(!deferredPrompt)return; deferredPrompt.prompt(); await deferredPrompt.userChoice; deferredPrompt=null; installBtn.hidden=true; });

(function showTipOnce(){
  const isIOS = /iphone|ipad|ipod/i.test(navigator.userAgent);
  const shown = store.get('__tipShown', false);
  if (isIOS && !shown) {
    const tip = document.getElementById('a2hsTip'); tip.hidden = false;
    document.getElementById('tipClose').addEventListener('click', ()=>{ tip.hidden = true; store.set('__tipShown', true); });
    setTimeout(()=>{ if(!tip.hidden){ tip.hidden = true; store.set('__tipShown', true);} }, 12000);
  }
})();

renderNextEvent(); renderNotices(); renderSchedules(); renderMoods();
