<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>車両判定シミュレーター</title>
    <style>
        :root { --blue: #00479d; --red: #e60012; --bg: #f8f9fa; --gray: #444; }
        body { font-family: -apple-system, sans-serif; background: var(--bg); margin: 0; padding-bottom: 180px; -webkit-user-select: none; }
        .header { background: var(--blue); color: white; padding: 10px; text-align: center; border-bottom: 3px solid var(--red); position: sticky; top: 0; z-index: 1000; display: flex; justify-content: space-between; align-items: center; }
        .header h1 { margin: 0; font-size: 0.9rem; }
        .section-title { background: var(--gray); color: white; padding: 8px 15px; font-weight: bold; font-size: 0.8rem; position: sticky; top: 43px; z-index: 999; }
        .item { display: flex; justify-content: space-between; align-items: center; padding: 10px 15px; background: white; border-bottom: 1px solid #eee; min-height: 58px; }
        .item-n { font-size: 0.9rem; font-weight: bold; }
        .pts-tag { color: var(--red); font-weight: bold; margin-left: 5px; font-size: 0.8rem; }
        .ctrls { display: flex; align-items: center; gap: 10px; }
        .btn { width: 44px; height: 44px; background: #fff; border: 1px solid #ccc; font-size: 1.5rem; font-weight: bold; border-radius: 8px; box-shadow: 0 2px 0 #ddd; display: flex; align-items: center; justify-content: center; }
        .btn:active { background: #eee; box-shadow: none; transform: translateY(2px); }
        .qty-input { width: 45px; height: 40px; text-align: center; border: 1px solid #ccc; font-size: 1.2rem; font-weight: bold; border-radius: 6px; }
        .footer { position: fixed; bottom: 0; width: 100%; background: white; border-top: 4px solid var(--blue); z-index: 1001; padding-bottom: env(safe-area-inset-bottom); box-shadow: 0 -5px 15px rgba(0,0,0,0.1); }
        .price-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 5px; padding: 10px; border-bottom: 1px solid #eee; }
        .price-box { text-align: center; }
        .price-label { font-size: 0.65rem; color: #666; font-weight: bold; }
        .price-val { font-size: 1.3rem; font-weight: bold; color: var(--red); }
        .footer-main { display: flex; justify-content: space-around; align-items: center; padding: 10px; }
        .total-box { text-align: center; }
        .res-v { font-size: 1.6rem; font-weight: bold; color: var(--blue); }
        .reset-btn { background: #666; color: white; border: none; padding: 12px 20px; border-radius: 8px; font-weight: bold; font-size: 0.85rem; }
        #admin-panel { display: none; position: fixed; top: 10%; left: 5%; width: 90%; background: white; border: 2px solid var(--blue); border-radius: 12px; z-index: 2000; padding: 20px; box-sizing: border-box; box-shadow: 0 0 100px rgba(0,0,0,0.5); }
        .admin-table { width: 100%; border-collapse: collapse; }
        .admin-table input { width: 100%; padding: 8px; border: 1px solid #ccc; border-radius: 4px; text-align: right; }
    </style>
</head>
<body>

<div class="header">
    <h1>車両判定システム</h1>
    <button onclick="openAdmin()" style="font-size: 0.7rem; background:none; color:white; border:1px solid white; border-radius:4px; padding:4px 8px;">設定</button>
</div>

<div id="main-content">
    <div class="section-title">リビング・キッチン</div>
    <div id="list-living"></div>
    <div class="section-title">家具・家電</div>
    <div id="list-others"></div>
    <div class="section-title">資材</div>
    <div id="list-shizai"></div>
    <div class="section-title">フリー入力</div>
    <div id="list-free"></div>
</div>

<div class="footer">
    <div class="price-grid">
        <div class="price-box"><div class="price-label">【通常料金】</div><span id="price-standard" class="price-val">0</span>円～</div>
        <div class="price-box"><div class="price-label">【指定料金】</div><span id="price-reserved" class="price-val">0</span>円～</div>
    </div>
    <div class="footer-main">
        <div class="total-box"><small>合計</small><br><span id="total-pts" class="res-v">0</span>P</div>
        <div class="total-box"><small>推奨車両</small><br><span id="truck-res" class="res-v" style="color:var(--red); font-size:1.1rem;">---</span></div>
        <button class="reset-btn" onclick="location.reload()">クリア</button>
    </div>
</div>

<div id="admin-panel">
    <h3 style="margin-top:0">単価設定</h3>
    <table class="admin-table">
        <tr><th>車両</th><th>通常</th><th>指定</th></tr>
        <tr><td>2T</td><td><input type="number" id="edit-2t-std"></td><td><input type="number" id="edit-2t-res"></td></tr>
        <tr><td>2TL</td><td><input type="number" id="edit-2tl-std"></td><td><input type="number" id="edit-2tl-res"></td></tr>
        <tr><td>3T</td><td><input type="number" id="edit-3t-std"></td><td><input type="number" id="edit-3t-res"></td></tr>
        <tr><td>2台口〜</td><td><input type="number" id="edit-mt-std"></td><td><input type="number" id="edit-mt-res"></td></tr>
    </table>
    <button onclick="saveAdmin()" style="width:100%; margin-top:20px; padding:15px; background:var(--blue); color:white; border:none; border-radius:8px; font-weight:bold;">保存</button>
</div>

<script>
// --- データ定義 ---
const dataLiving = [
    {n:"ソファー 1人掛け", p:9}, {n:"ソファー 2人掛け", p:22}, {n:"ソファー 3人掛け", p:50}, {n:"冷蔵庫 5D", p:25}, {n:"食器棚 特大", p:42}
];
const dataOthers = [
    {n:"洋服タンス", p:29}, {n:"和タンス", p:18}, {n:"洗濯機 ドラム", p:12}, {n:"ベッド 引出付", p:18}, {n:"自転車", p:15}, {n:"ダンボールA", p:1}, {n:"ダンボールB", p:2}
];
const dataShizai = [
    {n:"オリコン", p:1}, {n:"ハンガーBOX", p:7}, {n:"大型定数", p:15}
];

let PRICE_TABLE = JSON.parse(localStorage.getItem('sys_prices_v3')) || {
    "2T車": [15000, 18000], "2T車L": [20000, 24000], "3T車": [30000, 35000], "2台口以上": [45000, 52000]
};

let counter = 0;
function render(list, targetId) {
    const target = document.getElementById(targetId);
    if(!target) return;
    list.forEach(item => {
        counter++;
        const id = 'q' + counter;
        const div = document.createElement('div');
        div.className = 'item';
        div.innerHTML = `<div><span class="item-n">${item.n}</span><span class="pts-tag">${item.p}P</span></div>
            <div class="ctrls"><button class="btn" onclick="chg('${id}',-1)">－</button>
            <input type="number" id="${id}" class="qty-input" value="0" data-pts="${item.p}" oninput="calc()">
            <button class="btn" onclick="chg('${id}',1)">＋</button></div>`;
        target.appendChild(div);
    });
}

function chg(id, d) {
    const el = document.getElementById(id);
    el.value = Math.max(0, (parseInt(el.value) || 0) + d);
    calc();
}

function calc() {
    let total = 0;
    document.querySelectorAll('.qty-input[data-pts]').forEach(el => total += (parseInt(el.value)||0) * parseInt(el.dataset.pts));
    for(let i=1; i<=3; i++) {
        const fq = document.getElementById('fq'+i);
        const fp = document.getElementById('fp'+i);
        if(fq && fp) total += (parseInt(fq.value)||0) * (parseInt(fp.value)||0);
    }
    document.getElementById('total-pts').innerText = total.toLocaleString();

    let truck = "---";
    if (total > 0 && total <= 200) truck = "2T車";
    else if (total > 200 && total <= 260) truck = "2T車L";
    else if (total > 260 && total <= 360) truck = "3T車";
    else if (total > 360) truck = "2台口以上";

    document.getElementById('truck-res').innerText = truck;
    if(truck !== "---") {
        document.getElementById('price-standard').innerText = PRICE_TABLE[truck][0].toLocaleString();
        document.getElementById('price-reserved').innerText = PRICE_TABLE[truck][1].toLocaleString();
    }
}

function openAdmin() {
    if(prompt("パスワード") === "1234") {
        document.getElementById('edit-2t-std').value = PRICE_TABLE["2T車"][0];
        document.getElementById('edit-2t-res').value = PRICE_TABLE["2T車"][1];
        document.getElementById('edit-2tl-std').value = PRICE_TABLE["2T車L"][0];
        document.getElementById('edit-2tl-res').value = PRICE_TABLE["2T車L"][1];
        document.getElementById('edit-3t-std').value = PRICE_TABLE["3T車"][0];
        document.getElementById('edit-3t-res').value = PRICE_TABLE["3T車"][1];
        document.getElementById('edit-mt-std').value = PRICE_TABLE["2台口以上"][0];
        document.getElementById('edit-mt-res').value = PRICE_TABLE["2台口以上"][1];
        document.getElementById('admin-panel').style.display = 'block';
    }
}

function saveAdmin() {
    PRICE_TABLE["2T車"] = [Number(document.getElementById('edit-2t-std').value), Number(document.getElementById('edit-2t-res').value)];
    PRICE_TABLE["2T車L"] = [Number(document.getElementById('edit-2tl-std').value), Number(document.getElementById('edit-2tl-res').value)];
    PRICE_TABLE["3T車"] = [Number(document.getElementById('edit-3t-std').value), Number(document.getElementById('edit-3t-res').value)];
    PRICE_TABLE["2台口以上"] = [Number(document.getElementById('edit-mt-std').value), Number(document.getElementById('edit-mt-res').value)];
    localStorage.setItem('sys_prices_v3', JSON.stringify(PRICE_TABLE));
    document.getElementById('admin-panel').style.display = 'none';
    calc();
}

// 実行
render(dataLiving, 'list-living');
render(dataOthers, 'list-others');
render(dataShizai, 'list-shizai');

for(let i=1; i<=3; i++) {
    const div = document.createElement('div');
    div.className = 'item';
    div.innerHTML = `<input type="text" placeholder="品名" style="width:70px"><input type="number" id="fp${i}" placeholder="P" style="width:30px; margin:0 5px" oninput="calc()"><div class="ctrls"><button class="btn" onclick="chg('fq${i}',-1)">－</button><input type="number" id="fq${i}" class="qty-input" value="0" oninput="calc()"><button class="btn" onclick="chg('fq${i}',1)">＋</button></div>`;
    document.getElementById('list-free').appendChild(div);
}
</script>
</body>
</html>
