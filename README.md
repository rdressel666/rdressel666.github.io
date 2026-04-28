<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Italian Master v6.7 - 20+ Verbs</title>
    <style>
        body { font-family: sans-serif; background: #f0f2f5; display: flex; justify-content: center; padding: 20px; }
        .card { background: white; padding: 30px; border-radius: 15px; box-shadow: 0 4px 12px rgba(0,0,0,0.1); text-align: center; width: 100%; max-width: 700px; }
        .btn-main { width: 100%; padding: 15px; background: #2e7d32; color: white; border: none; border-radius: 8px; font-size: 1.2rem; cursor: pointer; font-weight: bold; margin-top: 10px; }
        input { width: 90%; padding: 12px; margin: 15px 0; border: 2px solid #ddd; border-radius: 8px; font-size: 1.4rem; text-align: center; outline: none; }
        .feedback { margin-top: 15px; font-weight: bold; min-height: 50px; line-height: 1.4; }
        .hidden { display: none !important; }
        .tab-btn { padding: 10px 20px; cursor: pointer; border: none; background: #ccc; border-radius: 5px; margin: 0 5px; font-weight: bold; }
        .active-tab { background: #2e7d32; color: white; }
        .accent-btn { padding: 8px 12px; border: 1px solid #ccc; border-radius: 4px; background: white; cursor: pointer; font-size: 1.1rem; margin: 2px; }
        .ref-container { overflow-x: auto; margin-top: 20px; max-height: 400px; }
        table { width: 100%; border-collapse: collapse; font-size: 0.85rem; text-align: left; }
        tr:nth-child(even) { background-color: #f9f9f9; }
        td, th { padding: 8px; border-bottom: 1px solid #eee; }
    </style>
</head>
<body>

<div class="card">
    <div style="margin-bottom: 20px;">
        <button id="t1" class="tab-btn active-tab" onclick="showDrill()">Drill Mode</button>
        <button id="t2" class="tab-btn" onclick="showRef()">Reference List</button>
    </div>

    <div id="drillBox">
        <select id="tenseSelect" onchange="resetTask()" style="width:100%; padding:10px; margin-bottom:10px; border-radius: 5px;">
            <option value="presente">Presente (I do)</option>
            <option value="passato_prossimo">Passato Prossimo (I spoke/did)</option>
            <option value="imperfetto">Imperfetto (I was/used to do)</option>
            <option value="futuro">Futuro (I will do)</option>
            <option value="condizionale">Condizionale (I would do)</option>
        </select>
        
        <div id="pLabel" style="color:#666;">--</div>
        <h1 id="vLabel" style="margin:10px 0; font-size: 2.5rem;">---</h1>
        <div id="mLabel" style="color:#2e7d32; font-weight:bold; font-size:1.2rem; font-style: italic;">---</div>

        <div id="feedback" class="feedback"></div>
        <input type="text" id="userInput" autocomplete="off" placeholder="Type answer...">
        
        <div style="margin-bottom: 10px;">
            <button class="accent-btn" onclick="addAcc('à')">à</button>
            <button class="accent-btn" onclick="addAcc('è')">è</button>
            <button class="accent-btn" onclick="addAcc('é')">é</button>
            <button class="accent-btn" onclick="addAcc('ì')">ì</button>
            <button class="accent-btn" onclick="addAcc('ò')">ò</button>
            <button class="accent-btn" onclick="addAcc('ù')">ù</button>
        </div>

        <button class="btn-main" id="mainBtn" onclick="doAction()">Check Answer</button>
    </div>

    <div id="refBox" class="hidden">
        <select id="vSelect" onchange="updateTable()" style="width:100%; padding:10px; border-radius: 5px;"></select>
        <div class="ref-container">
            <table>
                <thead><tr style="background:#eee;"><th>Person</th><th>Pres</th><th>Pass.Pro</th><th>Imp</th><th>Fut</th><th>Cond</th></tr></thead>
                <tbody id="rt"></tbody>
            </table>
        </div>
    </div>
</div>

<script>
    var verbs = [
        // --- ORIGINAL SET ---
        {inf: "Parlare", stem: "parl", p_part: "parlato", eng: "speak", eng_p: "spoke", group: "are", type: "reg"},
        {inf: "Vendere", stem: "vend", p_part: "venduto", eng: "sell", eng_p: "sold", group: "ere", type: "reg"},
        {inf: "Partire", stem: "part", p_part: "partito", eng: "leave", eng_p: "left", group: "ire", type: "reg"},
        {inf: "Dormire", stem: "dorm", p_part: "dormito", eng: "sleep", eng_p: "slept", group: "ire", type: "reg"},
        {inf: "Capire", stem: "cap", p_part: "capito", eng: "understand", eng_p: "understood", group: "ire", type: "isc"},
        {inf: "Finire", stem: "fin", p_part: "finito", eng: "finish", eng_p: "finished", group: "ire", type: "isc"},
        {inf: "Svegliarsi", stem: "svegli", p_part: "svegliato", eng: "wake up", eng_p: "woke up", group: "are", type: "refl"},
        {inf: "Sentirsi", stem: "sent", p_part: "sentito", eng: "feel", eng_p: "felt", group: "ire", type: "refl"},
        {inf: "Cercare", stem: "cerc", p_part: "cercato", eng: "look for", eng_p: "looked for", group: "are", type: "reg_h"},
        
        // --- 10 NEW VERBS ---
        {inf: "Mangiare", stem: "mangi", p_part: "mangiato", eng: "eat", eng_p: "ate", group: "are", type: "reg"},
        {inf: "Lavarsi", stem: "lav", p_part: "lavato", eng: "wash oneself", eng_p: "washed up", group: "are", type: "refl"},
        {inf: "Chiamarsi", stem: "chiam", p_part: "chiamato", eng: "be named", eng_p: "was named", group: "are", type: "refl"},
        {inf: "Leggere", stem: "legg", p_part: "letto", eng: "read", eng_p: "read", group: "ere", type: "reg"},
        {inf: "Vedere", stem: "ved", p_part: "visto", eng: "see", eng_p: "saw", group: "ere", type: "reg"},
        {inf: "Prendere", stem: "prend", p_part: "preso", eng: "take", eng_p: "took", group: "ere", type: "reg"},
        {inf: "Uscire", stem: "usc", p_part: "uscito", eng: "go out", eng_p: "went out", group: "ire", type: "reg"},
        {inf: "Aprire", stem: "apr", p_part: "aperto", eng: "open", eng_p: "opened", group: "ire", type: "reg"},
        {inf: "Fare", eng: "do", eng_p: "did", group: "ir", type: "ir", 
            presente: ["faccio", "fai", "fa", "facciamo", "fate", "fanno"],
            passato_prossimo: ["ho fatto", "hai fatto", "ha fatto", "abbiamo fatto", "avete fatto", "hanno fatto"],
            imperfetto: ["facevo", "facevi", "faceva", "facevamo", "facevate", "facevano"],
            futuro: ["farò", "farai", "farà", "faremo", "farete", "faranno"],
            condizionale: ["farei", "faresti", "farebbe", "faremmo", "fareste", "farebbero"]},
        {inf: "Andare", eng: "go", eng_p: "went", group: "ir", type: "ir", 
            presente: ["vado", "vai", "va", "andiamo", "andate", "vanno"],
            passato_prossimo: ["sono andato", "sei andato", "è andato", "siamo andati", "siete andati", "sono andati"],
            imperfetto: ["andavo", "andavi", "andava", "andavamo", "andavate", "andavano"],
            futuro: ["andrò", "andrai", "andrà", "andremo", "andrete", "andranno"],
            condizionale: ["andrei", "andresti", "andrebbe", "andremmo", "andreste", "andrebbero"]},

        // --- CORE IRREGULARS ---
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
            condizionale: ["avrei", "avresti", "avrebbe", "avremmo", "avreste", "avrebbero"]}
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

    function getConj(v, pi, tn) {
        if (v.type === "ir") return v[tn][pi];
        if (tn === 'passato_prossimo') {
            var res = aux_avere[pi] + " " + v.p_part;
            return (v.type === "refl") ? pro[pi].r + " " + res : res;
        }
        var s = v.stem;
        var gt = (tn === 'presente' && v.type === 'isc') ? 'isc' : v.group;
        var suffix = suf[tn][gt][pi];
        if (s.endsWith('i') && suffix.startsWith('i')) s = s.slice(0, -1);
        if (v.type === "reg_h" && ((tn === 'presente' && (pi === 1 || pi === 3)) || tn === 'futuro' || tn === 'condizionale')) s += 'h';
        var word = s + suffix;
        return (v.type === "refl") ? pro[pi].r + " " + word : word;
    }

    function getTenseEng(v, pi, tn) {
        var p = pro[pi].en;
        if (tn === 'presente') return p + " " + v.eng + (p === "He/She" ? "s" : "");
        if (tn === 'passato_prossimo') return p + " " + v.eng_p;
        if (tn === 'imperfetto') return p + " used to " + v.eng;
        if (tn === 'futuro') return p + " will " + v.eng;
        if (tn === 'condizionale') return p + " would " + v.eng;
        return p + " " + v.eng;
    }

    function resetTask() {
        wait = false;
        var tn = document.getElementById('tenseSelect').value;
        var v = verbs[Math.floor(Math.random() * verbs.length)];
        var pi = Math.floor(Math.random() * 6);
        task = { ans: getConj(v, pi, tn).toLowerCase() };
        
        document.getElementById('pLabel').innerText = pro[pi].it + " (" + pro[pi].en + ")";
        document.getElementById('vLabel').innerText = v.inf.replace("si", "");
        document.getElementById('mLabel').innerText = '"' + getTenseEng(v, pi, tn) + '"';
        document.getElementById('userInput').value = "";
        document.getElementById('userInput').disabled = false;
        document.getElementById('userInput').style.background = "white";
        document.getElementById('feedback').innerText = "";
        document.getElementById('mainBtn').innerText = "Check Answer";
        document.getElementById('userInput').focus();
    }

    function doAction() {
        if (wait) { resetTask(); return; }
        var input = document.getElementById('userInput').value.trim().toLowerCase();
        if (input === task.ans) {
            document.getElementById('feedback').innerHTML = "<span style='color:green; font-size:1.5rem;'>Ottimo! ✅</span>";
            setTimeout(resetTask, 800);
        } else {
            document.getElementById('feedback').innerHTML = "Incorrect. Answer: <b style='color:#d32f2f; font-size:1.3rem;'>" + task.ans + "</b>";
            document.getElementById('userInput').disabled = true;
            document.getElementById('userInput').style.background = "#ffd6d6";
            document.getElementById('mainBtn').innerText = "Next Verb →";
            wait = true;
        }
    }

    function showDrill() {
        document.getElementById('drillBox').className = "";
        document.getElementById('refBox').className = "hidden";
        document.getElementById('t1').className = "tab-btn active-tab";
        document.getElementById('t2').className = "tab-btn";
    }

    function showRef() {
        document.getElementById('drillBox').className = "hidden";
        document.getElementById('refBox').className = "";
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
        var el = document.getElementById('userInput');
        if(!wait) { el.value += c; el.focus(); }
    }

    window.addEventListener('load', function() {
        var sel = document.getElementById('vSelect');
        for(var i=0; i<verbs.length; i++) {
            var o = document.createElement('option');
            o.value = i; o.innerText = verbs[i].inf.replace("si", "");
            sel.appendChild(o);
        }
        document.addEventListener('keydown', function(e) { if(e.key === 'Enter') doAction(); });
        resetTask();
    });
</script>

</body>
</html>
