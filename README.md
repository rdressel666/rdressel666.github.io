<!DOCTYPE html>
<html lang="it">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Italian Master v8.5</title>
    <style>
        body { font-family: -apple-system, sans-serif; background: #f0f2f5; display: flex; justify-content: center; padding: 0; margin: 0; overflow: hidden; height: 100vh; }
        .card { background: white; padding: 10px; border-radius: 0; text-align: center; width: 100%; max-width: 500px; height: 100%; }
        
        /* Tighter vertical stack */
        h1 { margin: 0; font-size: 2.2rem; line-height: 1; color: #1a1a1a; }
        #pLabel { color: #666; font-size: 1rem; margin-top: 2px; }
        #mLabel { color: #2e7d32; font-weight: bold; font-size: 1.1rem; font-style: italic; margin: 0; }
        
        /* Shorter interactive elements */
        .btn-main { width: 100%; padding: 12px; background: #2e7d32; color: white; border: none; border-radius: 8px; font-size: 1.2rem; cursor: pointer; font-weight: bold; margin-top: 5px; }
        input { width: 90%; padding: 10px; margin: 4px 0; border: 2px solid #ddd; border-radius: 10px; font-size: 1.4rem; text-align: center; outline: none; -webkit-appearance: none; }
        
        .feedback { font-weight: bold; min-height: 25px; line-height: 1; font-size: 1rem; margin: 2px 0; }
        .hidden { display: none !important; }
        
        .tab-btn { padding: 6px 12px; cursor: pointer; border: none; background: #ccc; border-radius: 6px; margin: 0 5px; font-weight: bold; font-size: 0.9rem; }
        .active-tab { background: #2e7d32; color: white; }
        
        /* Compact Accents */
        .accent-bar { display: flex; justify-content: center; gap: 4px; margin-bottom: 4px; }
        .accent-btn { padding: 8px; border: 1px solid #ccc; border-radius: 6px; background: white; cursor: pointer; font-size: 1.2rem; min-width: 40px; }

        #skipBtn { background: none; border: none; color: #999; margin-top: 4px; text-decoration: underline; font-size: 0.8rem; padding: 5px; }
    </style>
</head>
<body>

<div class="card">
    <div style="margin-bottom: 5px;">
        <button id="t1" class="tab-btn active-tab" onclick="showDrill()">Drill</button>
        <button id="t2" class="tab-btn" onclick="showRef()">List</button>
    </div>

    <div id="drillBox">
        <select id="tenseSelect" onchange="resetTask()" style="width:100%; padding:6px; margin-bottom:4px; font-size: 0.9rem; border-radius: 6px;">
            <option value="presente">Presente</option>
            <option value="passato_prossimo">Passato Prossimo</option>
            <option value="imperfetto">Imperfetto</option>
            <option value="futuro">Futuro</option>
            <option value="condizionale">Condizionale</option>
        </select>
        
        <div id="pLabel">...</div>
        <h1 id="vLabel">---</h1>
        <div id="mLabel">---</div>

        <div id="feedback" class="feedback"></div>
        <input type="text" id="userInput" autocomplete="off" autocapitalize="none" spellcheck="false" placeholder="Scrivi qui...">
        
        <div class="accent-bar">
            <button class="accent-btn" onclick="addAcc('à')">à</button>
            <button class="accent-btn" onclick="addAcc('è')">è</button>
            <button class="accent-btn" onclick="addAcc('é')">é</button>
            <button class="accent-btn" onclick="addAcc('ì')">ì</button>
            <button class="accent-btn" onclick="addAcc('ò')">ò</button>
            <button class="accent-btn" onclick="addAcc('ù')">ù</button>
        </div>

        <button class="btn-main" id="mainBtn" onclick="doAction()">Verifica</button>
        <button id="skipBtn" onclick="resetTask()">Skip</button>
    </div>

    <div id="refBox" style="display:none;">
        <select id="vSelect" onchange="updateTable()" style="width:100%; padding:8px; border-radius: 8px;"></select>
        <div style="overflow-x:auto; margin-top:5px; height: 60vh;">
            <table style="width:100%; border-collapse:collapse; font-size:0.75rem; text-align:left;">
                <thead><tr style="background:#eee;"><th>Sogg.</th><th>Pres</th><th>Pass.Pro</th><th>Imp</th><th>Fut</th><th>Cond</th></tr></thead>
                <tbody id="rt"></tbody>
            </table>
        </div>
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
        {inf: "Bere", eng: "drink", eng_p: "drank", group: "ir", type: "ir",
            presente: ["bevo", "bevi", "beve", "beviamo", "bevete", "bevono"],
            passato_prossimo: ["ho bevuto", "hai bevuto", "ha bevuto", "abbiamo bevuto", "avete bevuto", "hanno bevuto"],
            imperfetto: ["bevevo", "bevevi", "beveva", "bevevamo", "bevevate", "bevevano"],
            futuro: ["berrò", "berrai", "berrà", "berremo", "berrete", "berranno"],
            condizionale: ["berrei", "berresti", "berrebbe", "berremmo", "berreste", "berrebbero"]},
        {inf: "Andare", eng: "go", eng_p: "went", group: "ir", type: "ir", 
            presente: ["vado", "vai", "va", "andiamo", "andate", "vanno"],
            passato_prossimo: ["sono andato", "sei andato", "è andato", "siamo andati", "siete andati", "sono andati"],
            imperfetto: ["andavo", "andavi", "andava", "andavamo", "andavate", "andavano"],
            futuro: ["andrò", "andrai", "andrà", "andremo", "andrete", "andranno"],
            condizionale: ["andrei", "andresti", "andrebbe", "andremmo", "andreste", "andrebbero"]},
        {inf: "Essere", eng: "be", eng_p: "was", group: "ir", type: "ir", 
            presente: ["sono", "sei", "è", "siamo", "siete", "sono"],
            passato_prossimo: ["sono stato", "sei stato", "è stato", "siamo stati", "siete stati", "sono stati"],
            imperfetto: ["ero", "eri", "era", "eravamo", "eravate", "erano"],
            futuro: ["sarò", "sarai", "sarà", "saremo", "sarete", "saranno"],
            condizionale: ["sarei", "saresti", "sarebbe", "saremmo", "sareste", "sarebbero"]},
        {inf: "Avere", eng: "have", eng_p: "had", group: "ir", type: "ir",
            presente: ["ho", "hai", "ha", "abbiamo", "avete", "hanno"],
            passato_prossimo: ["ho avuto", "hai avuto", "ha avuto", "abbiamo avuto", "avete avuto", "hanno avuto"],
            imperfetto: ["avevo", "avevi", "aveva", "avevamo", "avevate", "avevano"],
            futuro: ["avrò", "avrai", "avrà", "avremo", "avrete", "avranno"],
            condizionale: ["avrei", "avresti", "avrebbe", "avremmo", "avreste", "avrebbero"]},
        {inf: "Fare", eng: "do", eng_p: "did", group: "ir", type: "ir", 
            presente: ["faccio", "fai", "fa", "facciamo", "fate", "fanno"],
            passato_prossimo: ["ho fatto", "hai fatto", "ha fatto", "abbiamo fatto", "avete fatto", "hanno fatto"],
            imperfetto: ["facevo", "facevi", "faceva", "facevamo", "facevate", "facevano"],
            futuro: ["farò", "farai", "farà", "faremo", "farete", "faranno"],
            condizionale: ["farei", "faresti", "farebbe", "faremmo", "fareste", "farebbero"]}
    ];

    var aux_avere = ["ho", "hai", "ha", "abbiamo", "avete", "hanno"];
    var pro = [
        {it: "Io", en: "I", r: "mi"}, {it: "Tu", en: "You", r: "ti"},
        {it: "Lui/Lei", en: "He/She", r: "si"}, {it: "Noi", en: "We", r: "ci"},
        {it: "Voi", en: "You (pl)", r: "vi"}, {it: "Loro", en: "They", r: "si"}
    ];

    var suf = {
        presente: {are:["o","i","a","iamo","ate","ano"], ere:["o","i","e","iamo","ete","ono"], ire:["o","i","e","iamo","ite","ono"], isc:["isco","isci","isce","iamo","ite","iscono"]},
        imperfetto: {are:["avo","avi","ava","avamo","avate","avano"], ere:["evo","evi","eva","evamo","evate","evano"], ire:["ivo","ivi","iva","ivamo","ivate","ivano"], isc:["ivo","ivi","iva","ivamo","ivate","ivano"]},
        futuro: {are:["erò","erai","erà","eremo","erete","eranno"], ere:["erò","erai","erà","eremo","erete","eranno"], ire:["irò","irai","irà","iremo","irete","iranno"], isc:["irò","irai","irà","iremo","irete","iranno"]},
        condizionale: {are:["erei","eresti","erebbe","eremmo","ereste","erebbero"], ere:["erei","eresti","erebbe","eremmo","ereste","erebbero"], ire:["irei","iresti","irebbe","iremmo","ireste","irebbero"], isc:["irei","iresti","irebbe","iremmo","ireste","irebbero"]}
    };

    var task = null;
    var wait = false;
    var inputEl = document.getElementById('userInput');

    function cleanInf(v) {
        if (v.type === "ir") return v.inf;
        return (v.type === "refl") ? v.inf.substring(0, v.inf.length - 2) + "e" : v.inf;
    }

    function getConj(v, pi, tn) {
        if (v.type === "ir") return v[tn][pi];
        if (tn === 'passato_prossimo') {
            var res = aux_avere[pi] + " " + (v.p_part || "");
            return (v.type === "refl") ? pro[pi].r + " " + res : res;
        }
        var s = v.stem;
        var gt = (tn === 'presente' && v.type === 'isc') ? 'isc' : v.group;
        var suffix = suf[tn][gt][pi];
        var needsH = (v.type === "reg_h" && ((tn === 'presente' && (pi === 1 || pi === 3)) || tn === 'futuro' || tn === 'condizionale'));
        if (s.charAt(s.length - 1) === 'i' && suffix.charAt(0) === 'i') s = s.substring(0, s.length - 1);
        if (needsH) s += "h";
        var word = s + suffix;
        return (v.type === "refl") ? pro[pi].r + " " + word : word;
    }

    function resetTask() {
        wait = false;
        var tn = document.getElementById('tenseSelect').value;
        var v = verbs[Math.floor(Math.random() * verbs.length)];
        var pi = Math.floor(Math.random() * 6);
        task = { ans: getConj(v, pi, tn).toLowerCase() };
        document.getElementById('pLabel').innerText = pro[pi].it + " (" + pro[pi].en + ")";
        document.getElementById('vLabel').innerText = cleanInf(v);
        var p = pro[pi].en;
        var hint = p + " ";
        if (tn === 'presente') hint += v.eng + (p === "He/She" ? "s" : "");
        else if (tn === 'passato_prossimo') hint += (v.eng_p || "");
        else if (tn === 'imperfetto') hint += "used to " + v.eng;
        else if (tn === 'futuro') hint += "will " + v.eng;
        else hint += "would " + v.eng;
        document.getElementById('mLabel').innerText = '"' + hint + '"';
        
        inputEl.value = "";
        inputEl.readOnly = false;
        inputEl.style.background = "white";
        document.getElementById('feedback').innerText = "";
        document.getElementById('mainBtn').innerText = "Verifica";
        
        // Re-focus instantly
        setTimeout(function(){ inputEl.focus(); }, 10);
    }

    function doAction() {
        if (wait) { resetTask(); return; }
        var inp = inputEl.value.toLowerCase().trim();
        if (inp === task.ans) {
            document.getElementById('feedback').innerHTML = "<span style='color:green'>Ottimo!</span>";
            setTimeout(resetTask, 500);
        } else {
            document.getElementById('feedback').innerHTML = "No: <b style='color:#d32f2f'>" + task.ans + "</b>";
            inputEl.readOnly = true;
            inputEl.style.background = "#ffd6d6";
            document.getElementById('mainBtn').innerText = "Avanti";
            wait = true;
        }
    }

    function showDrill() {
        document.getElementById('drillBox').style.display = 'block';
        document.getElementById('refBox').style.display = 'none';
        document.getElementById('t1').className = "tab-btn active-tab";
        document.getElementById('t2').className = "tab-btn";
        inputEl.focus();
    }

    function showRef() {
        document.getElementById('drillBox').style.display = 'none';
        document.getElementById('refBox').style.display = 'block';
        document.getElementById('t1').className = "tab-btn";
        document.getElementById('t2').className = "tab-btn active-tab";
        updateTable();
    }

    function updateTable() {
        var vi = document.getElementById('vSelect').value;
        var v = verbs[vi];
        var h = "";
        for(var i=0; i<6; i++) {
            h += "<tr><td><b>" + pro[i].it + "</b></td>" +
                 "<td>" + getConj(v,i,'presente') + "</td>" +
                 "<td>" + getConj(v,i,'passato_prossimo') + "</td>" +
                 "<td>" + getConj(v,i,'imperfetto') + "</td>" +
                 "<td>" + getConj(v,i,'futuro') + "</td>" +
                 "<td>" + getConj(v,i,'condizionale') + "</td></tr>";
        }
        document.getElementById('rt').innerHTML = h;
    }

    function addAcc(c) {
        if(!wait) { 
            inputEl.value += c; 
            inputEl.focus(); 
        }
    }

    var vs = document.getElementById('vSelect');
    for(var i=0; i<verbs.length; i++) {
        var o = document.createElement('option');
        o.value = i; o.innerText = cleanInf(verbs[i]); vs.appendChild(o);
    }
    document.addEventListener('keydown', function(e) { if(e.keyCode === 13) doAction(); });
    resetTask();
</script>

</body>
</html>
