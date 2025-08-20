export default {
  async fetch(request, env, ctx) {
    const url = new URL(request.url);
    if (url.pathname === "/health") return new Response("ok", { status: 200 });
    return new Response(renderHTML(), { headers: { "content-type": "text/html; charset=utf-8" } });
  },
};

function renderHTML() {
  return `<!doctype html>
<html lang="fa" dir="rtl">
<head>
<meta charset="utf-8"/>
<meta name="viewport" content="width=device-width, initial-scale=1"/>
<title>پنل سازنده کانفیگ VLESS</title>
<style>
:root{--bg:#0b1020;--card:#111831;--line:#1e2a4a;--txt:#e8ecf1;--muted:#9fb0cc;--accent:#6ea8fe;}
body{margin:0;font-family:system-ui,Segoe UI,Roboto,IRANSans,Arial,sans-serif;background:var(--bg);color:var(--txt)}
.wrap{max-width:1000px;margin:0 auto;padding:24px}
.card{background:var(--card);border:1px solid var(--line);border-radius:12px;padding:20px}
h1{margin:0 0 10px;font-size:20px}
p.muted{color:var(--muted);font-size:13px;margin:4px 0 16px}
.grid{display:grid;gap:12px;grid-template-columns:1fr}
@media(min-width:900px){.grid{grid-template-columns:1fr 1fr}}
label{font-size:13px;color:var(--muted);display:block;margin-bottom:6px}
input,select,button,textarea{width:100%;box-sizing:border-box;padding:10px 12px;border-radius:8px;border:1px solid var(--line);background:#0c1427;color:var(--txt)}
.row{display:grid;gap:10px;grid-template-columns:1fr 1fr}
small{color:var(--muted)}
.btn{background:#0c1427;border-color:#2a3d6a;cursor:pointer}
.btn-primary{background:var(--accent);color:#0b1020;border:1px solid #4e7ed8}
.output{white-space:pre-wrap;word-wrap:anywhere;font-family:ui-monospace,SFMono-Regular,Menlo,monospace;background:#0a1327;border:1px dashed #25407b;border-radius:8px;padding:10px}
.badges{display:flex;gap:8px;flex-wrap:wrap;margin-top:8px}
.badge{background:#152245;border:1px solid var(--line);border-radius:999px;padding:4px 10px;font-size:12px}
hr{border:0;border-top:1px solid var(--line);margin:16px 0}
</style>
</head>
<body>
  <div class="wrap">
    <div class="card">
      <h1>پنل سازنده کانفیگ VLESS</h1>
      <p class="muted">این پنل برای ساخت سریع لینک vless:// و خروجی JSON کلاینت‌هاست. Worker خودش سرور VLESS نیست.</p>

      <div class="grid">
        <div>
          <label>UUID کاربر</label>
          <div class="row">
            <input id="uuid" placeholder="مثال: 39ff3eba-3129-4d9c-88c8-ea29237a29c9">
            <button class="btn" onclick="genUUID()">ساخت UUID</button>
          </div>
        </div>

        <div>
          <label>نام نمایشی کانکشن</label>
          <input id="name" placeholder="مثلاً vless-cf-ws">
        </div>

        <div>
          <label>CDN Host (هدر Host/SNI)</label>
          <input id="host" placeholder="مثلاً your-worker.workers.dev یا cf-domain.example.com">
          <small>برای اتصال از طریق Worker، همین دامنه Worker را قرار بده.</small>
        </div>

        <div>
          <label>WS Path</label>
          <input id="path" value="/ws" placeholder="/path?ed=2560">
        </div>

        <div>
          <label>سرور/آی‌پی مقصد (اختیاری برای اتصال مستقیم CDN)</label>
          <input id="server" placeholder="مثلاً 104.24.xx.xx یا دامنه">
          <small>اگر خالی بگذاری و «اتصال از طریق Worker» فعال باشد، از همین دامنه Worker استفاده می‌شود.</small>
        </div>

        <div class="row">
          <div>
            <label>پورت</label>
            <input id="port" type="number" value="443">
          </div>
          <div>
            <label>TLS</label>
            <select id="tls">
              <option value="tls" selected>tls (HTTPS)</option>
              <option value="none">none (HTTP)</option>
            </select>
          </div>
        </div>

        <div class="row">
          <div>
            <label>اتصال از طریق همین Worker</label>
            <select id="viaWorker">
              <option value="yes" selected>بله (توصیه‌شده)</option>
              <option value="no">خیر (مستقیم به CDN IP)</option>
            </select>
            <small>برای workers.dev همیشه TLS/443 را انتخاب کن.</small>
          </div>
          <div>
            <label>SNI (در صورت TLS)</label>
            <input id="sni" placeholder="پیش‌فرض برابر Host">
          </div>
        </div>
      </div>

      <hr>

      <div class="row">
        <button class="btn-primary" onclick="buildAll()">ساخت کانفیگ‌ها</button>
        <button class="btn" onclick="fillWorkerDefaults()">پیش‌فرض مناسب Worker</button>
      </div>

      <div style="margin-top:16px">
        <div class="badges">
          <span class="badge" onclick="copyText('vlessURI')">کپی vless://</span>
          <span class="badge" onclick="copyText('v2rayJSON')">کپی V2Ray JSON</span>
          <span class="badge" onclick="copyText('singboxJSON')">کپی Sing-box JSON</span>
        </div>
      </div>

      <h3>لینک VLESS</h3>
      <div id="vlessURI" class="output"></div>

      <h3>V2Ray/Xray outbound JSON</h3>
      <div id="v2rayJSON" class="output"></div>

      <h3>Sing-box outbound</h3>
      <div id="singboxJSON" class="output"></div>

      <hr>
      <small class="muted">
        نکته: اگر «اتصال از طریق Worker» را انتخاب کنی، بهتر است TLS و پورت 443 استفاده شود و
        Host/SNI برابر دامنه Worker باشد. برای اتصال مستقیم به CDN IP می‌توانی TLS را بسته به تنظیماتت خاموش یا روشن کنی.
      </small>
    </div>
    <div style="text-align:center;color:#6e7ea6;margin-top:10px">/health ➜ تست سریع: <code id="wurl"></code></div>
  </div>

<script>
function genUUID(){
  const u = (self.crypto && crypto.randomUUID) ? crypto.randomUUID() : 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, c=>{
    const r = Math.random()*16|0, v = c === 'x' ? r : (r&0x3|0x8); return v.toString(16);
  });
  document.getElementById('uuid').value = u;
}

function fillWorkerDefaults(){
  const host = location.host;
  document.getElementById('host').value = host;
  document.getElementById('sni').value = host;
  document.getElementById('path').value = '/ws';
  document.getElementById('viaWorker').value = 'yes';
  document.getElementById('tls').value = 'tls';
  document.getElementById('port').value = '443';
}

function buildAll(){
  const uuid = val('uuid');
  const name = val('name') || 'vless-cf-ws';
  const host = val('host');
  const path = val('path') || '/ws';
  const viaWorker = val('viaWorker') === 'yes';
  const tls = val('tls'); // 'tls' | 'none'
  let sni = val('sni') || host;

  let server = val('server').trim();
  let port = val('port').trim();

  if (viaWorker) {
    server = location.host;        // اتصال از طریق Worker
    port = tls === 'tls' ? '443' : (port || '80');
    sni = host || server;          // SNI برابر host (یا خود worker)
  } else {
    // اتصال مستقیم به CDN/IP طبق ورودی کاربر
    if (!server) {
      alert('برای اتصال مستقیم، فیلد "سرور/آی‌پی مقصد" را پر کن یا اتصال از طریق Worker را انتخاب کن.');
      return;
    }
    if (!port) port = (tls === 'tls' ? '443' : '80');
  }

  const params = new URLSearchParams();
  params.set('encryption','none');
  params.set('type','ws');
  params.set('host', host || server);
  params.set('path', path);
  if (tls === 'tls') {
    params.set('security','tls');
    if (sni) params.set('sni', sni);
  }

  const vless = `vless://${uuid}@${server}:${port}?${params.toString()}#${encodeURIComponent(name)}`;
  setText('vlessURI', vless);

  const v2 = buildV2RayOutbound({ uuid, server, port: Number(port), hostHeader: host || server, path, tls: tls==='tls', sni });
  setText('v2rayJSON', JSON.stringify(v2, null, 2));

  const sb = buildSingboxOutbound({ uuid, server, port: Number(port), hostHeader: host || server, path, tls: tls==='tls', sni, name });
  setText('singboxJSON', JSON.stringify(sb, null, 2));
}

function buildV2RayOutbound({ uuid, server, port, hostHeader, path, tls, sni }){
  return {
    tag: "vless-ws",
    protocol: "vless",
    settings: {
      vnext: [{
        address: server,
        port: port,
        users: [{ id: uuid, encryption: "none" }]
      }]
    },
    streamSettings: {
      network: "ws",
      security: tls ? "tls" : "none",
      ...(tls ? { tlsSettings: { serverName: sni || hostHeader } } : {}),
      wsSettings: {
        path: path || "/ws",
        headers: { Host: hostHeader }
      }
    },
    mux: { enabled: true, concurrency: 8 }
  };
}

function buildSingboxOutbound({ uuid, server, port, hostHeader, path, tls, sni, name }){
  return {
    type: "vless",
    tag: name || "vless-ws",
    server,
    server_port: port,
    uuid,
    flow: "",
    packet_encoding: "",
    tls: tls ? { enabled: true, server_name: sni || hostHeader, insecure: false } : { enabled: false },
    transport: {
      type: "ws",
      path: path || "/ws",
      headers: { Host: hostHeader }
    }
  };
}

function val(id){ return (document.getElementById(id).value || "").trim(); }
function setText(id, t){ document.getElementById(id).textContent = t; }
async function copyText(id){
  const t = document.getElementById(id).textContent.trim();
  if (!t) return;
  try { await navigator.clipboard.writeText(t); alert('کپی شد'); } catch(e){ alert('کپی نشد: '+e); }
}

document.getElementById('wurl').textContent = location.origin + '/health';
</script>
</body>
</html>`;
}
