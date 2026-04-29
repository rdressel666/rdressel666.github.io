<!DOCTYPE html>
<html lang="it">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Italian Master v8.9</title>
    <style>
        html, body { height: 100%; margin: 0; padding: 0; background: #f0f2f5; overflow: hidden; }
        body { font-family: -apple-system, sans-serif; display: flex; flex-direction: column; align-items: center; }
        .card { background: white; width: 100%; max-width: 600px; height: 100%; display: flex; flex-direction: column; padding: 5px 10px; box-sizing: border-box; }

        /* THE COMPACT HEADER */
        .header-box { display: flex; align-items: baseline; justify-content: center; gap: 10px; margin: 0; }
        #pLabel { color: #d32f2f; font-size: 1.1rem; font-weight: bold; text-transform: uppercase; }
        h1 { margin: 0; font-size: 2.3rem; line-height: 1; color: #1a1a1a; }
        #mLabel { color: #2e7d32; font-weight: bold; font-size: 1.1rem; font-style: italic; margin: 2px 0; }
        
        input { width: 100%; padding: 12px; margin: 5px 0; border: 3px solid #ddd; border-radius: 10px; font-size: 1.6rem; text-align: center; outline: none; box-sizing: border-box; -webkit-appearance: none; }
        .btn-main { width: 100%; padding: 14px; background: #2e7d32; color: white; border: none; border-radius: 10px; font-size: 1.3rem; font-weight: bold; }
        .accent-bar { display: flex; justify-content: center; gap: 4px; margin: 4px 0; }
        .accent-btn { padding: 8px; border: 1px solid #ccc; border-radius: 8px; background: white; font-size: 1.3rem; min-width: 42px; }
        .feedback { font-weight: bold; min-height: 25px; font-size: 1rem; margin: 2px 0; display: flex; align-items: center; justify-content: center; }
        .top-row { display: flex; justify-content: space-between; align-items: center; margin-bottom: 5px; }
        .tab-btn { padding: 6px 10px; border: none; background: #ccc; border-radius: 6px; font-weight: bold; font-size: 0.85rem; }
        .active-tab { background: #2e7d32; color: white; }
    </style>
</head>
<body onclick="forceFocus()">
<div class="card">
    <div class="top-row">
        <div>
            <button id="t1" class="tab-btn active-tab" onclick="showDrill()">Drill</button>
            <button id="t2" class="tab-btn" onclick="showRef()">List</button>
        </div>
        <select id="tenseSelect" onchange="resetTask()" style="padding:4px; font-size: 0.85rem; border-radius: 6px;">
            <option value="presente">Pres</option>
            <option value="passato_prossimo">Pass.Pro</option>
            <option value="imperfetto">Imp</option>
            <option value="futuro">Fut</option>
            <option value="condizionale">Cond</option>
        </select>
    </div>

    <div id="drillBox">
        <div class="header-box">
            <div id="pLabel">...</div>
            <h1 id="vLabel">---</h1>
        </div>
        <div id="mLabel">---</div>
        <div id="feedback" class="feedback"></div>
        <input type="text" id="userInput" autocomplete="off" autocapitalize="none" spellcheck="false" placeholder="Scrivi...">
        <div class="accent-bar">
            <button class="accent-btn" onclick="addAcc('à')">à</button>
            <button class="accent-btn" onclick="addAcc('è')">è</button>
            <button class="accent-btn" onclick="addAcc('é')">é</button>
            <button class="accent-btn" onclick="addAcc('ì')">ì</button>
            <button class="accent-btn" onclick="addAcc('ò')">ò</button>
            <button class="accent-btn" onclick="addAcc('ù')">ù</button>
        </div>
        <button class="btn-main" id="mainBtn" onclick="doAction()">Verifica</button>
        <button onclick="resetTask()" style="background:none; border:none; color:#999; margin-top:5px; text-decoration:underline; font-size: 0.75rem;">Skip</button>
    </div>

    <div id="refBox" style="display:none;">
        <select id="vSelect" onchange="updateTable()" style="width:100%; padding:8px; border-radius: 8px;"></select>
        <div style="overflow-y:auto; margin-top:5px;"><table style="width:100%; border-collapse:collapse; font-size:0.8rem; text-align:left;">
            <thead><tr style="background:#eee;"><th>Sogg.</th><th>Pres</th><th>Pass.Pro</th><th>Imp</th></tr></thead>
            <tbody id="rt"></tbody>
        </table></div>
    </div>
</div>

<script>
    var verbs = [
        {inf: "Parlare", stem: "parl", p_part: "parlato", eng: "speak", eng_p: "spoke", group: "are", type: "reg"},
        {inf: "Vendere", stem: "vend", p_part: "venduto", eng: "sell", eng_p: "sold", group: "ere", type: "reg"},
        {inf: "Partire", stem: "part", p_part: "partito", eng: "leave", eng_p: "left", group: "ire", type: "reg"},
        {inf: "Capire", stem: "cap", p_part: "capito", eng: "understand", eng_p: "understood", group: "ire", type: "isc"},
        {inf: "Svegliarsi", stem: "svegli", p_part: "svegliato", eng: "wake up", eng_p: "woke up", group: "are", type: "refl"},
        {inf: "Cercare", stem: "cerc", p_part: "cercato", eng: "look for", eng_p: "looked for", group: "are", type: "reg_h"},
        {inf: "Pagare", stem: "pag", p_part: "pagato", eng: "pay", eng_p: "paid", group: "are", type: "reg_h"},
        {inf: "Mangiare", stem: "mangi", p_part: "mangiato", eng: "eat", eng_p: "ate", group: "are", type: "reg"},
        {inf: "Andare", eng: "go", eng_p: "went", group: "ir", type: "ir", presente: ["vado", "vai", "va", "andiamo", "andate", "vanno"], passato_prossimo: ["sono andato", "sei andato", "è andato", "siamo andati", "siete andati", "sono andati"], imperfetto: ["andavo", "andavi", "andava", "andavamo", "andavate", "andavano"], futuro: ["andrò", "andrai", "andrà", "andremo", "andrete", "andranno"], condizionale: ["andrei", "andresti", "andrebbe", "andremmo", "andreste", "andrebbero"]}
    ];

    var aux_avere = ["ho", "hai", "ha", "abbiamo", "avete", "hanno"];
    var pro = [{it:"Io",en:"I",r:"mi"},{it:"Tu",en:"You",r:"ti"},{it:"Lui/Lei",en:"He/She",r:"si"},{it:"Noi",en:"We",r:"ci"},{it:"Voi",en:"You (pl)",r:"vi"},{it:"Loro",en:"They",r:"si"}];
    var suf = {
        presente: {are:["o","i","a","iamo","ate","ano"], ere:["o","i","e","iamo","ete","ono"], ire:["o","i","e","iamo","ite","ono"], isc:["isco","isci","isce","iamo","ite","iscono"]},
        imperfetto: {are:["avo","avi","ava","avamo","avate","avano"], ere:["evo","evi","eva","evamo","evate","evano"], ire:["ivo","ivi","iva","ivamo","ivate","ivano"], isc:["ivo","ivi","iva","ivamo","ivate","ivano"]},
        futuro: {are:["erò","erai","erà","eremo","erete","eranno"], ere:["erò","erai","erà","eremo","erete","eranno"], ire:["irò","irai","irà","iremo","irete","iranno"], isc:["irò","irai","irà","iremo","irete","iranno"]},
        condizionale: {are:["erei","eresti","erebbe","eremmo","ereste","erebbero"], ere:["erei","eresti","erebbe","eremmo","ereste","erebbero"], ire:["irei","iresti","irebbe","iremmo","ireste","irebbero"], isc:["irei","iresti","irebbe","iremmo","ireste","irebbero"]}
    };

    var task = null, wait = false;
    var inputEl = document.getElementById('userInput');

    function forceFocus() { if(!wait) inputEl.focus(); }
    function cleanInf(v) { return (v.type === "refl") ? v.inf.substring(0, v.inf.length - 2) + "e" : v.inf; }

    function getConj(v, pi, tn) {
        if (v.type === "ir") return v[tn][pi];
        if (tn === 'passato_prossimo') return (v.type === "refl" ? pro[pi].r + " " : "") + aux_avere[pi] + " " + (v.p_part || "");
        var s = v.stem, gt = (tn === 'presente' && v.type === 'isc') ? 'isc' : v.group, suffix = suf[tn][gt][pi];
        var needsH = (v.type === "reg_h" && ((tn === 'presente' && (pi === 1 || pi === 3)) || tn === 'futuro' || tn === 'condizionale'));
        if (s.charAt(s.length - 1) === 'i' && suffix.charAt(0) === 'i') s = s.substring(0, s.length - 1);
        if (needsH) s += "h";
        return (v.type === "refl" ? pro[pi].r + " " : "") + s + suffix;
    }

    function resetTask() {
        wait = false;
        var tn = document.getElementById('tenseSelect').value;
        var v = verbs[Math.floor(Math.random() * verbs.length)];
        var pi = Math.floor(Math.random() * 6);
        task = { ans: getConj(v, pi, tn).toLowerCase() };
        document.getElementById('pLabel').innerText = pro[pi].it;
        document.getElementById('vLabel').innerText = cleanInf(v);
        var hint = pro[pi].en + " " + ((tn === 'presente') ? v.eng : (tn === 'passato_prossimo' ? (v.eng_p||"") : (tn === 'imperfetto' ? "used to "+v.eng : (tn === 'futuro' ? "will "+v.eng : "would "+v.eng))));
        document.getElementById('mLabel').innerText = '"' + hint + '"';
        inputEl.value = ""; inputEl.readOnly = false; inputEl.style.background = "white";
        document.getElementById('feedback').innerText = ""; document.getElementById('mainBtn').innerText = "Verifica";
        setTimeout(forceFocus, 50);
    }

    function doAction() {
        if (wait) { resetTask(); return; }
        var inp = inputEl.value.toLowerCase().trim();
        if (inp === task.ans) {
            document.getElementById('feedback').innerHTML = "<span style='color:green'>Ottimo!</span>";
            setTimeout(resetTask, 500);
        } else {
            document.getElementById('feedback').innerHTML = "No: <b style='color:#d32f2f'>" + task.ans + "</b>";
            inputEl.readOnly = true; inputEl.style.background = "#ffd6d6";
            document.getElementById('mainBtn').innerText = "Avanti";
            wait = true;
        }
    }

    function showDrill() { document.getElementById('drillBox').style.display = 'block'; document.getElementById('refBox').style.display = 'none'; document.getElementById('t1').className = "tab-btn active-tab"; document.getElementById('t2').className = "tab-btn"; forceFocus(); }
    function showRef() { document.getElementById('drillBox').style.display = 'none'; document.getElementById('refBox').style.display = 'block'; document.getElementById('t1').className = "tab-btn"; document.getElementById('t2').className = "tab-btn active-tab"; updateTable(); }

    function updateTable() {
        var v = verbs[document.getElementById('vSelect').value], h = "";
        for(var i=0; i<6; i++) { h += "<tr><td><b>" + pro[i].it + "</b></td>" + "<td>" + getConj(v,i,'presente') + "</td>" + "<td>" + getConj(v,i,'passato_prossimo') + "</td>" + "<td>" + getConj(v,i,'imperfetto') + "</td></tr>"; }
        document.getElementById('rt').innerHTML = h;
    }

    var vs = document.getElementById('vSelect');
    for(var i=0; i<verbs.length; i++) { var o = document.createElement('option'); o.value = i; o.innerText = cleanInf(verbs[i]); vs.appendChild(o); }
    document.addEventListener('keydown', function(e) { if(e.keyCode === 13) doAction(); });
    resetTask();
</script>
</body>
</html>
