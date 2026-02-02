<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>NJFREGI レジシステム申込フォーム</title>

<style>
*{box-sizing:border-box}
body{
  margin:0; font-family:-apple-system,BlinkMacSystemFont,"Hiragino Kaku Gothic ProN","Yu Gothic",Meiryo,sans-serif;
  background:#f3f3f3; color:#111;
}
header{padding:14px 10px 6px;text-align:center}
.logoText{font-size:22px;font-weight:900;letter-spacing:1px}
.logoSub{font-size:13px;font-weight:700;opacity:.7;margin-top:2px}

.container{max-width:520px;margin:0 auto;padding:0 12px 40px}

.stepper{display:flex;gap:6px;margin:10px 0 14px}
.dot{flex:1;text-align:center;opacity:.35}
.dot span{
  display:inline-flex;align-items:center;justify-content:center;
  width:28px;height:28px;border-radius:99px;
  border:2px solid #111;font-weight:800;font-size:14px
}
.dot p{margin:4px 0 0;font-size:11px;font-weight:700}
.dot.on{opacity:1}
.dot.on span{background:#111;color:#fff}

.card{
  background:#fff;border-radius:18px;padding:16px;
  box-shadow:0 6px 20px rgba(0,0,0,.06);margin-bottom:14px
}

h2{margin:0 0 12px;font-size:18px;font-weight:900}
label{font-size:13px;font-weight:700;margin-top:12px;display:block}
input{
  width:100%;padding:12px;font-size:16px;margin-top:6px;
  border-radius:12px;border:1px solid #ddd
}

.row{
  display:flex;align-items:center;justify-content:space-between;
  padding:12px;border:1px solid #e6e6e6;border-radius:14px;margin:10px 0
}
.left .name{font-weight:800}
.left .price{font-size:12px;opacity:.7;margin-top:3px}
.qty{
  width:90px;font-size:18px;padding:8px;border-radius:12px;
  border:1px solid #ddd;text-align:center
}

.totalBox{
  background:#f6f6f6;border-radius:14px;padding:12px;margin-top:14px
}
.totalBox div{display:flex;justify-content:space-between;font-weight:800;margin:6px 0}

.btn{
  width:100%;padding:14px;font-size:16px;border-radius:16px;
  border:none;font-weight:900;margin-top:14px
}
.primary{background:#111;color:#fff}
.sub{background:#e5e5e5}

.confirm-big{
  font-size:18px;line-height:1.7;background:#f6f6f6;
  padding:16px;border-radius:16px;border:1px solid #e2e2e2
}
.confirm-big b{font-size:19px}

.muted{font-size:13px;opacity:.7;margin-top:10px}
.err{color:#c00;font-weight:700}
.ok{color:#0a7a0a;font-weight:800}
</style>
</head>

<body>

<header>
  <div class="logoText">NJFREGI</div>
  <div class="logoSub">レジシステム申込フォーム</div>
</header>

<div class="container">

<!-- stepper -->
<div class="stepper" id="stepper">
  <div class="dot on"><span>1</span><p>確認</p></div>
  <div class="dot"><span>2</span><p>契約者</p></div>
  <div class="dot"><span>3</span><p>設置</p></div>
  <div class="dot"><span>4</span><p>数量</p></div>
  <div class="dot"><span>5</span><p>確認</p></div>
</div>

<!-- Step 1 -->
<section class="step">
  <div class="card">
    <h2>重要事項確認</h2>
    <label><input type="checkbox" class="check"> 契約期間2年・自動更新を理解しました</label>
    <label><input type="checkbox" class="check"> 最低利用24ヶ月を理解しました</label>
    <button class="btn primary" onclick="nextStep()">次へ</button>
  </div>
</section>

<!-- Step 2 -->
<section class="step" style="display:none">
  <div class="card">
    <h2>ご契約者</h2>
    <label>会社名 / 店舗名</label><input id="company">
    <label>担当者</label><input id="person">
    <label>電話</label><input id="tel">
    <button class="btn primary" onclick="nextStep()">次へ</button>
  </div>
</section>

<!-- Step 3 -->
<section class="step" style="display:none">
  <div class="card">
    <h2>設置先</h2>
    <label>店舗名</label><input id="install">
    <label>住所</label><input id="addr">
    <button class="btn primary" onclick="nextStep()">次へ</button>
  </div>
</section>

<!-- Step 4 -->
<section class="step" style="display:none">
  <div class="card">
    <h2>設備数量入力</h2>

    <div id="deviceList"></div>

    <div class="totalBox">
      <div><span>税別合計</span><span id="sumEx">0 円</span></div>
      <div><span>消費税</span><span id="tax">0 円</span></div>
      <div><span>税込合計</span><span id="sumIn">0 円</span></div>
    </div>

    <button class="btn primary" onclick="buildConfirm()">確認へ</button>
  </div>
</section>

<!-- Step 5 -->
<section class="step" style="display:none">
  <div class="card">
    <h2>申込内容確認</h2>
    <div id="confirmView" class="confirm-big"></div>
    <button class="btn primary" id="submitBtn">送信する</button>
    <div id="msg" class="muted"></div>
  </div>
</section>

</div>

<script>
// ===== CONFIG =====
const GAS_URL = "【这里粘贴你的 GAS WebApp URL（/exec）】";
const API_KEY = "NJFREGI-CHANGE-ME";

// ===== FIXED DATA =====
const TAX = 0.10;
const devices = [
  {name:"タブレット", unit:35000, qty:0},
  {name:"プリンター", unit:29000, qty:0}
];

let step = 0;

// ===== STEP CONTROL =====
function nextStep(){
  if(step===0){
    if(![...document.querySelectorAll(".check")].every(c=>c.checked)){
      alert("すべて確認してください"); return;
    }
  }
  setStep(step+1);
}
function setStep(n){
  step = n;
  document.querySelectorAll(".step").forEach((s,i)=>s.style.display=i===n?"block":"none");
  document.querySelectorAll(".dot").forEach((d,i)=>d.classList.toggle("on",i===n));
}

// ===== DEVICE RENDER =====
function renderDevices(){
  const box=document.getElementById("deviceList");
  box.innerHTML=devices.map((d,i)=>`
    <div class="row">
      <div class="left">
        <div class="name">${d.name}</div>
        <div class="price">単価 ${d.unit.toLocaleString()} 円</div>
      </div>
      <input class="qty" type="number" min="0" value="${d.qty}"
        oninput="devices[${i}].qty=Math.max(0,parseInt(this.value||0));calc();">
    </div>
  `).join("");
}
function calc(){
  let ex=devices.reduce((s,d)=>s+d.qty*d.unit,0);
  document.getElementById("sumEx").innerText=ex.toLocaleString()+" 円";
  document.getElementById("tax").innerText=Math.round(ex*TAX).toLocaleString()+" 円";
  document.getElementById("sumIn").innerText=Math.round(ex*(1+TAX)).toLocaleString()+" 円";
}

// ===== CONFIRM =====
function buildConfirm(){
  let ex=devices.reduce((s,d)=>s+d.qty*d.unit,0);
  document.getElementById("confirmView").innerHTML=`
    <b>会社名：</b>${company.value}<br>
    <b>担当者：</b>${person.value}<br>
    <b>電話：</b>${tel.value}<br><br>
    <b>設備：</b><br>
    ${devices.map(d=>`${d.name} × ${d.qty}`).join("<br>")}<br><br>
    <b>税込合計：</b>${Math.round(ex*(1+TAX)).toLocaleString()} 円
  `;
  setStep(4);
}

// ===== SUBMIT =====
submitBtn.onclick=async()=>{
  msg.textContent="送信中…";
  try{
    const res=await fetch(GAS_URL,{
      method:"POST",
      headers:{"Content-Type":"application/json"},
      body:JSON.stringify({
        apiKey:API_KEY,
        company:company.value,
        person:person.value,
        tel:tel.value,
        install:install.value,
        addr:addr.value,
        devices:devices,
        tax:TAX
      })
    });
    const j=await res.json();
    if(!j.ok) throw j.error;
    msg.innerHTML="<span class='ok'>送信完了。PDF・Excelをメール送信しました。</span>";
  }catch(e){
    msg.innerHTML="<span class='err'>送信失敗："+e+"</span>";
  }
};

// init
renderDevices(); calc();
</script>

</body>
</html>
