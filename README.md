<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8">
<title>Spotify Overlay</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Pixelify+Sans:wght@400;500;600;700&family=VT323&display=swap" rel="stylesheet">
<style>
  :root{--accent:#1ec8d8;--panel:#14151a;--panel2:#1c1e25;--edge:#0a0b0e;--ink:#eafbfd;--muted:#7e8c90}
  *{margin:0;padding:0;box-sizing:border-box}
  html,body{background:transparent;font-family:'Pixelify Sans',system-ui,sans-serif;color:var(--ink);overflow:hidden}
  #wrap{display:inline-block;transform-origin:top left}
  .panel{width:430px;background:var(--panel);border:4px solid var(--edge);
    box-shadow:inset 2px 2px 0 rgba(255,255,255,.07),inset -2px -2px 0 rgba(0,0,0,.55),0 10px 30px rgba(0,0,0,.5),0 0 26px rgba(30,200,216,.12);
    padding:16px 16px 14px}
  .nowlabel{display:flex;align-items:center;gap:9px;margin-bottom:12px}
  .nowlabel .disc{width:20px;height:20px;animation:spin 3.5s linear infinite}
  @keyframes spin{to{transform:rotate(360deg)}}
  .nowlabel span{font-size:15px;font-weight:600;letter-spacing:.16em;text-transform:uppercase;color:var(--accent);text-shadow:0 0 10px rgba(30,200,216,.5)}
  .row{display:flex;gap:14px;align-items:center}
  .art{width:84px;height:84px;flex:none;background:var(--panel2);border:3px solid var(--edge);
    box-shadow:inset 2px 2px 0 rgba(255,255,255,.06),inset -2px -2px 0 rgba(0,0,0,.5)}
  .art img{width:100%;height:100%;object-fit:cover;display:block}
  .art.pixel img,.nart.pixel img{image-rendering:pixelated}
  .meta{min-width:0;flex:1}
  .title{font-size:25px;font-weight:600;line-height:1.05;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;text-shadow:0 0 12px rgba(30,200,216,.18)}
  .artist{font-size:17px;color:var(--muted);white-space:nowrap;overflow:hidden;text-overflow:ellipsis;margin-top:3px}
  .barwrap{margin-top:14px}
  .bar{height:14px;background:var(--panel2);border:3px solid var(--edge);box-shadow:inset 2px 2px 0 rgba(0,0,0,.5);overflow:hidden}
  .fill{height:100%;width:0%;background:repeating-linear-gradient(90deg,var(--accent) 0 8px,color-mix(in srgb,var(--accent) 78%,#000) 8px 11px);box-shadow:0 0 12px rgba(30,200,216,.6);transition:width .25s linear}
  .times{display:flex;justify-content:space-between;margin-top:5px;font-family:'VT323',monospace;font-size:20px;color:var(--muted)}
  .times .cur{color:var(--accent)}
  .next{margin-top:15px;border-top:2px solid rgba(255,255,255,.08);padding-top:12px}
  .nexthd{font-size:13px;letter-spacing:.18em;text-transform:uppercase;color:var(--muted);margin-bottom:9px}
  .nitem{display:flex;gap:11px;align-items:center;margin-bottom:9px}
  .nitem:last-child{margin-bottom:0}
  .nart{width:40px;height:40px;flex:none;background:var(--panel2);border:2px solid var(--edge)}
  .nart img{width:100%;height:100%;object-fit:cover;display:block}
  .nmeta{min-width:0}
  .ntitle{font-size:17px;color:#cfe9ec;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;max-width:300px}
  .nartist{font-size:14px;color:var(--muted);white-space:nowrap;overflow:hidden;text-overflow:ellipsis;max-width:300px}
  .req{color:var(--accent)}
  .idle{font-size:18px;color:var(--muted);padding:6px 2px}

  #toasts{position:fixed;top:8px;right:8px;display:flex;flex-direction:column;gap:8px;align-items:flex-end;z-index:20}
  .toast{display:flex;gap:10px;align-items:center;background:var(--panel);border:3px solid var(--edge);
    box-shadow:inset 2px 2px 0 rgba(255,255,255,.07),inset -2px -2px 0 rgba(0,0,0,.5),0 0 22px rgba(30,200,216,.22);
    padding:10px 14px;transform:translateX(130%);transition:transform .45s cubic-bezier(.2,.8,.2,1)}
  .toast.show{transform:translateX(0)}
  .toast .disc{width:22px;height:22px;flex:none;animation:spin 3.5s linear infinite}
  .thd{font-size:13px;color:var(--accent);text-transform:uppercase;letter-spacing:.14em}
  .ttxt{font-size:18px;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;max-width:320px}

  #setup{display:none;position:fixed;inset:0;background:rgba(5,6,8,.92);z-index:50;padding:34px;font-family:system-ui,sans-serif}
  #setup.show{display:block}
  #setup h2{font-family:'Pixelify Sans';color:var(--accent);font-size:24px;margin-bottom:6px}
  #setup .st{font-size:14px;color:var(--muted);margin:4px 0 16px}
  #setup .grp{background:var(--panel);border:3px solid var(--edge);padding:16px;margin-bottom:14px;max-width:560px}
  #setup .gh{font-family:'Pixelify Sans';font-size:18px;margin-bottom:8px}
  #setup button{font-family:'Pixelify Sans';font-size:16px;background:rgba(30,200,216,.16);border:2px solid var(--accent);color:#dffafd;padding:9px 14px;cursor:pointer;margin-right:8px}
  #setup button:hover{background:rgba(30,200,216,.3)}
  #setup input{width:100%;margin-top:9px;background:#0d1416;border:2px solid rgba(255,255,255,.18);color:#fff;padding:8px;font-family:'VT323';font-size:18px}
  #setup .ok{color:#7fe3a0}#setup .no{color:#e38a8a}
  #setup .tok{margin-top:9px;font-family:'VT323';font-size:14px;color:#9fb;word-break:break-all;background:#0d1416;border:1px solid rgba(30,200,216,.3);padding:8px;display:none}
  #setup .hint{font-size:12px;color:#566;margin-top:14px}
</style>
</head>
<body>
<div id="wrap"><div class="panel" id="panel"></div></div>
<div id="toasts"></div>
<div id="setup"></div>

<script>
/* ============================================================
   HIER ANPASSEN
   ============================================================ */
const CONFIG = {
  // --- Spotify ---
  SP_CLIENT_ID:  "907e3a50218b4aaca088911fd3bc6803",
  SP_REFRESH:    "",                 // optional fest eintragen (sonst per Setup speichern)
  // --- Twitch ---
  TW_CLIENT_ID:  "DEINE_TWITCH_CLIENT_ID",
  REWARD_TITLE:  "Song Request",     // exakter Name deiner Channel-Points-Belohnung (mit Text-Eingabe!)
  // --- gemeinsam ---
  REDIRECT_URI:  "https://deepezzz.github.io/", // Pages-URL, in BEIDEN Apps als Redirect eintragen

  // --- Look / Verhalten ---
  accent:   "#1ec8d8",
  showNext: true,
  nextCount: 3,
  showProgress: true,
  showToast: true,      // kurzer Hinweis wenn ein Request reinkommt
  pixelArt: true,
  scale:    1,
  pollMs:   2500
};
/* ============================================================ */

document.getElementById("wrap").style.transform="scale("+CONFIG.scale+")";
document.documentElement.style.setProperty("--accent",CONFIG.accent);
const SP_SCOPE="user-read-currently-playing user-read-playback-state user-modify-playback-state";
const TW_SCOPE="channel:read:redemptions";

/* ---------- helpers ---------- */
function rand(n){const a=new Uint8Array(n);crypto.getRandomValues(a);return Array.from(a,b=>("0"+b.toString(16)).slice(-2)).join("");}
async function challenge(v){const d=await crypto.subtle.digest("SHA-256",new TextEncoder().encode(v));
  return btoa(String.fromCharCode(...new Uint8Array(d))).replace(/\+/g,"-").replace(/\//g,"_").replace(/=+$/,"");}
function lsGet(k){try{return localStorage.getItem(k);}catch(e){return null;}}
function lsSet(k,v){try{localStorage.setItem(k,v);}catch(e){}}
function esc(s){return (s||"").replace(/[&<>]/g,c=>({"&":"&amp;","<":"&lt;",">":"&gt;"}[c]));}
function fmt(ms){ms=Math.max(0,ms);const s=Math.floor(ms/1000);return Math.floor(s/60)+":"+("0"+(s%60)).slice(-2);}
function arts(a){return (a||[]).map(x=>x.name).join(", ");}
function cover(it,big){const im=it&&it.album&&it.album.images;if(!im||!im.length)return "";return big?im[0].url:(im[im.length-1].url||im[0].url);}

/* ---------- Spotify auth ---------- */
async function spLogin(){
  const v=rand(48);lsSet("sv_verifier",v);
  const p=new URLSearchParams({client_id:CONFIG.SP_CLIENT_ID,response_type:"code",redirect_uri:CONFIG.REDIRECT_URI,
    scope:SP_SCOPE,code_challenge_method:"S256",code_challenge:await challenge(v),state:"spotify"});
  location.href="https://accounts.spotify.com/authorize?"+p.toString();
}
async function spHandleCode(){
  const q=new URLSearchParams(location.search);const code=q.get("code");
  if(!code||q.get("state")!=="spotify")return;
  const body=new URLSearchParams({grant_type:"authorization_code",code,redirect_uri:CONFIG.REDIRECT_URI,client_id:CONFIG.SP_CLIENT_ID,code_verifier:lsGet("sv_verifier")||""});
  try{const r=await fetch("https://accounts.spotify.com/api/token",{method:"POST",headers:{"Content-Type":"application/x-www-form-urlencoded"},body});
    const j=await r.json();if(j.refresh_token){lsSet("sv_refresh",j.refresh_token);spSet(j.access_token,j.expires_in);window._sptok=j.refresh_token;}}catch(e){}
  history.replaceState({},"",CONFIG.REDIRECT_URI);
}
let spTok=null,spExp=0;
function spSet(t,e){spTok=t;spExp=Date.now()+(e||3600)*1000;}
function spRefresh(){return CONFIG.SP_REFRESH||lsGet("sv_refresh");}
async function spEnsure(){
  if(spTok&&Date.now()<spExp-12000)return spTok;
  const rt=spRefresh();if(!rt)return null;
  const body=new URLSearchParams({grant_type:"refresh_token",refresh_token:rt,client_id:CONFIG.SP_CLIENT_ID});
  try{const r=await fetch("https://accounts.spotify.com/api/token",{method:"POST",headers:{"Content-Type":"application/x-www-form-urlencoded"},body});
    const j=await r.json();if(j.access_token){spSet(j.access_token,j.expires_in);if(j.refresh_token)lsSet("sv_refresh",j.refresh_token);return spTok;}}catch(e){}
  return null;
}
async function spApi(path,opt){
  const t=await spEnsure();if(!t)return null;
  try{const r=await fetch("https://api.spotify.com/v1/"+path,Object.assign({headers:{Authorization:"Bearer "+t}},opt||{}));
    if(r.status===204)return{empty:true};if(!r.ok)return null;
    const txt=await r.text();return txt?JSON.parse(txt):{ok:true};}catch(e){return null;}
}

/* ---------- Twitch auth (implicit) ---------- */
let twTok=null,broadcasterId=null;
function twLogin(){
  const p=new URLSearchParams({client_id:CONFIG.TW_CLIENT_ID,redirect_uri:CONFIG.REDIRECT_URI,response_type:"token",scope:TW_SCOPE});
  location.href="https://id.twitch.tv/oauth2/authorize?"+p.toString();
}
function twHandleToken(){
  const h=new URLSearchParams(location.hash.slice(1));
  if(h.get("access_token")){twTok=h.get("access_token");lsSet("tw_token",twTok);history.replaceState({},"",CONFIG.REDIRECT_URI);}
  else twTok=lsGet("tw_token");
}
async function twApi(path,opt){
  if(!twTok)return null;
  try{const r=await fetch("https://api.twitch.tv/helix/"+path,Object.assign({headers:{Authorization:"Bearer "+twTok,"Client-Id":CONFIG.TW_CLIENT_ID}},opt||{}));
    if(r.status===401){twTok=null;lsSet("tw_token","");return null;}
    if(!r.ok)return null;return await r.json();}catch(e){return null;}
}
async function twInit(){
  if(!twTok)return false;
  const u=await twApi("users");
  if(u&&u.data&&u.data[0]){broadcasterId=u.data[0].id;startEventSub();return true;}
  return false;
}

/* ---------- EventSub ---------- */
let ws=null,sessionId=null;
function startEventSub(url){
  try{ws&&ws.close();}catch(e){}
  ws=new WebSocket(url||"wss://eventsub.wss.twitch.tv/ws");
  ws.onmessage=async(e)=>{
    let m;try{m=JSON.parse(e.data);}catch(_){return;}
    const ty=m.metadata&&m.metadata.message_type;
    if(ty==="session_welcome"){sessionId=m.payload.session.id;await subscribe();}
    else if(ty==="notification"){handleEvent(m.payload);}
    else if(ty==="session_reconnect"){startEventSub(m.payload.session.reconnect_url);}
  };
  ws.onclose=()=>{setTimeout(()=>{if(twTok)startEventSub();},4000);};
}
async function subscribe(){
  if(!broadcasterId||!sessionId)return;
  await twApi("eventsub/subscriptions",{method:"POST",headers:{Authorization:"Bearer "+twTok,"Client-Id":CONFIG.TW_CLIENT_ID,"Content-Type":"application/json"},
    body:JSON.stringify({type:"channel.channel_points_custom_reward_redemption.add",version:"1",condition:{broadcaster_user_id:broadcasterId},transport:{method:"websocket",session_id:sessionId}})});
}
function handleEvent(payload){
  if(!payload.subscription||payload.subscription.type!=="channel.channel_points_custom_reward_redemption.add")return;
  const ev=payload.event;
  if(CONFIG.REWARD_TITLE && ev.reward && ev.reward.title!==CONFIG.REWARD_TITLE)return;
  requestSong(ev.user_input||"", ev.user_name||ev.user_login||"jemand");
}

/* ---------- song request -> Spotify ---------- */
const requestBy={};
async function requestSong(text,who){
  if(!text.trim())return;
  const j=await spApi("search?type=track&limit=1&q="+encodeURIComponent(text));
  const tr=j&&j.tracks&&j.tracks.items&&j.tracks.items[0];
  if(!tr)return;
  requestBy[tr.uri]=who;
  await spApi("me/player/queue?uri="+encodeURIComponent(tr.uri),{method:"POST"});
  if(CONFIG.showToast)toast(who,tr.name);
}

/* ---------- state + fetch ---------- */
let cur=null,queue=[];
async function fetchPlayer(){const p=await spApi("me/player");
  if(p&&!p.empty&&p.item)cur={item:p.item,progress:p.progress_ms||0,dur:p.item.duration_ms,playing:!!p.is_playing,at:Date.now()};else cur=null;}
async function fetchQueue(){if(!CONFIG.showNext)return;const q=await spApi("me/player/queue");if(q&&q.queue)queue=q.queue.slice(0,CONFIG.nextCount);}

/* ---------- render ---------- */
const DISC='<svg class="disc" viewBox="0 0 24 24"><circle cx="12" cy="12" r="11" fill="#0c0d11"/><circle cx="12" cy="12" r="11" fill="none" stroke="'+CONFIG.accent+'" stroke-width="1.4"/><circle cx="12" cy="12" r="6.5" fill="none" stroke="'+CONFIG.accent+'" stroke-width="1.2" opacity=".6"/><circle cx="12" cy="12" r="2.4" fill="'+CONFIG.accent+'"/></svg>';
const panel=document.getElementById("panel");
function render(){
  if(!spRefresh()){panel.innerHTML='<div class="nowlabel">'+DISC+'<span>Spotify</span></div><div class="idle">Nicht verbunden \u2013 Taste S</div>';return;}
  if(!cur){panel.innerHTML='<div class="nowlabel">'+DISC+'<span>Now Playing</span></div><div class="idle">Gerade l\u00e4uft nichts \u2026</div>';return;}
  const live=cur.playing?Math.min(cur.dur,cur.progress+(Date.now()-cur.at)):cur.progress;
  const pct=cur.dur?(live/cur.dur*100):0;const pix=CONFIG.pixelArt?" pixel":"";
  let h='<div class="nowlabel">'+DISC+'<span>'+(cur.playing?"Now Playing":"Pausiert")+'</span></div>';
  h+='<div class="row"><div class="art'+pix+'">'+(cover(cur.item,true)?'<img src="'+cover(cur.item,true)+'">':'')+'</div><div class="meta"><div class="title">'+esc(cur.item.name)+'</div><div class="artist">'+esc(arts(cur.item.artists))+'</div></div></div>';
  if(CONFIG.showProgress)h+='<div class="barwrap"><div class="bar"><div class="fill" style="width:'+pct+'%"></div></div><div class="times"><span class="cur">'+fmt(live)+'</span><span>'+fmt(cur.dur)+'</span></div></div>';
  if(CONFIG.showNext&&queue.length){
    h+='<div class="next"><div class="nexthd">Up Next</div>';
    queue.forEach(t=>{const by=requestBy[t.uri];
      h+='<div class="nitem"><div class="nart'+pix+'">'+(cover(t,false)?'<img src="'+cover(t,false)+'">':'')+'</div><div class="nmeta"><div class="ntitle">'+esc(t.name)+'</div><div class="nartist">'+esc(arts(t.artists))+(by?' <span class="req">\u2022 von '+esc(by)+'</span>':'')+'</div></div></div>';});
    h+='</div>';
  }
  panel.innerHTML=h;
}
function toast(who,song){
  const el=document.createElement("div");el.className="toast";
  el.innerHTML=DISC+'<div><div class="thd">Song Request</div><div class="ttxt">'+esc(who)+': '+esc(song)+'</div></div>';
  document.getElementById("toasts").appendChild(el);
  requestAnimationFrame(()=>el.classList.add("show"));
  setTimeout(()=>{el.classList.remove("show");setTimeout(()=>el.remove(),550);},5000);
}

/* ---------- setup overlay (Taste S) ---------- */
const setup=document.getElementById("setup");
function renderSetup(){
  const sp=spRefresh()?'<span class="ok">verbunden</span>':'<span class="no">nicht verbunden</span>';
  const tw=twTok?'<span class="ok">verbunden'+(broadcasterId?'':' (init\u2026)'):'<span class="no">nicht verbunden';
  setup.innerHTML='<h2>Overlay Setup</h2><div class="st">Mit S schlie\u00dfen. Vor dem Live-Schalten einrichten.</div>'+
    '<div class="grp"><div class="gh">Spotify '+sp+'</div>'+
      '<button id="spc">Connect Spotify</button>'+
      '<input id="spi" placeholder="Refresh-Token einf\u00fcgen (optional)">'+
      '<div class="tok" id="spt"></div></div>'+
    '<div class="grp"><div class="gh">Twitch '+tw+'</span></div>'+
      '<button id="twc">Connect Twitch</button>'+
      '<div class="hint">Belohnung muss exakt "'+esc(CONFIG.REWARD_TITLE)+'" hei\u00dfen und Viewer-Texteingabe haben.</div></div>'+
    '<div class="hint">Channel Points = Affiliate n\u00f6tig. Song in Queue legen = Spotify Premium n\u00f6tig.</div>';
  document.getElementById("spc").onclick=spLogin;
  document.getElementById("twc").onclick=twLogin;
  document.getElementById("spi").onchange=function(){if(this.value.trim()){lsSet("sv_refresh",this.value.trim());location.reload();}};
  if(window._sptok){const o=document.getElementById("spt");o.style.display="block";o.textContent="Token: "+window._sptok;}
}
document.addEventListener("keydown",e=>{if((e.key||"").toLowerCase()==="s"){setup.classList.toggle("show");if(setup.classList.contains("show"))renderSetup();}});

/* ---------- boot ---------- */
(async()=>{
  await spHandleCode();
  twHandleToken();
  if(twTok)twInit();
  if(!spRefresh()){setup.classList.add("show");renderSetup();}
  render();
  if(spRefresh()){await fetchPlayer();await fetchQueue();render();}
  setInterval(async()=>{if(spRefresh())await fetchPlayer();},CONFIG.pollMs);
  setInterval(async()=>{if(spRefresh())await fetchQueue();},CONFIG.pollMs*2.4);
  setInterval(render,250);
})();
</script>
</body>
</html>
