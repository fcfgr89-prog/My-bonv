export default {
  async fetch(request, env, ctx) {
    return new Response(renderHTML(), { headers: { "content-type": "text/html; charset=utf-8" } });
  },
};

function renderHTML() {
  return `<!doctype html>
<html lang="fa" dir="rtl">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1"/>
<title>پنل سازنده کانفیگ VLESS</title>
<style>
body{margin:0;font-family:sans-serif;background:#0b1020;color:#e8ecf1}
.wrap{max-width:900px;margin:auto;padding:20px}
.card{background:#111831;border:1px solid #1e2a4a;border-radius:12px;padding:20px}
input,select,button{width:100%;margin:4px 0;padding:8px;border-radius:6px;border:1px solid #2a3d6a;background:#0c1427;color:#e8ecf1}
button{cursor:pointer}
.output{background:#0a1327;padding:8px;border-radius:6px;font-family:monospace;word-wrap:anywhere}
.row{display:grid;grid-template-columns:1fr 1fr;gap:8px}
</style>
</head>
<body>
<div class="wrap">
  <div class="card">
    <h2>ساخت لینک و کانفیگ VLESS</h2>
    <label>UUID</label>
    <div class="row">
      <input id="uuid" placeholder="UUID کاربر">
      <button onclick="genUUID()">ساخت UUID</button>
    </div>
    <label>نام کانکشن</label>
    <input id="name" placeholder="مثال: vless-cf-ws">
    <label>Host (SNI)</label>
    <input id="host" placeholder="دامنه CDN یا Worker">
    <label>WS Path</label>
    <input id="path" value="/ws">
    <label>آدرس سرور (برای اتصال مستقیم)</label>
    <input id="server" placeholder="IP یا دامنه">
    <div class="row">
      <div>
        <label>پورت</label>
        <input id="port" value="443">
      </div>
      <div>
        <label>TLS</label>
        <select id="tls"><option value="tls">tls</option><option value="none">none</option></select>
      </div>
    </div>
    <button onclick="build()">بساز</button>
    <h3>لینک VLESS</h3>
    <div id="vlessURI" class="output"></div>
    <h3>V2Ray JSON</h3>
    <div id="v2rayJSON" class="output"></div>
  </div>
</div>
<script>
function genUUID(){
  const u = crypto.randomUUID ? crypto.randomUUID() : 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, c=>{
    const r = Math.random()*16|0, v = c==='x'?r:(r&0x3|0x8); return v.toString(16);
  });
  document.getElementById('uuid').value = u;
}
function val(id){return document.getElementById(id).value.trim();}
function set(id,txt){document.getElementById(id).textContent = txt;}
function build(){
  const uuid=val('uuid'), name=val('name')||'vless', host=val('host'), path=val('path')||'/ws', server=val('server')||host, port=val('port')||'443', tls=val('tls');
  const params=new URLSearchParams({encryption:'none',type:'ws',host:host,path:path});
  if(tls==='tls'){params.set('security','tls');params.set('sni',host);}
  const vless=`vless://${uuid}@${server}:${port}?${params}#${encodeURIComponent(name)}`;
  set('vlessURI',vless);
  const v2={tag:"vless-ws",protocol:"vless",settings:{vnext:[{address:server,port:+port,users:[{id:uuid,encryption:"none"}]}]},streamSettings:{network:"ws",security:tls,wsSettings:{path:path,headers:{Host:host}},...(tls==="tls"?{tlsSettings:{serverName:host}}:{})}};
  set('v2rayJSON',JSON.stringify(v2,null,2));
}
</script>
</body>
</html>`;
}
