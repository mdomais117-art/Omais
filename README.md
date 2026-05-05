<!DOCTYPE html>
<html lang="hi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Worker App - Malegaon</title>
    
    <script src="https://www.gstatic.com/firebasejs/9.15.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.15.0/firebase-database-compat.js"></script>

    <style>
        :root { --primary: #0d6efd; --success: #198754; --warning: #ffc107; --danger: #dc3545; --bg: #f0f2f5; }
        body { font-family: 'Segoe UI', sans-serif; background: var(--bg); margin: 0; padding-bottom: 60px; overflow-x: hidden; }
        
        /* PWA Install Banner */
        #installBanner { background: var(--primary); color: white; padding: 10px; text-align: center; font-weight: bold; font-size: 14px; display: none; }
        
        /* Header */
        .header { background: #1e293b; color: white; padding: 12px 15px; display: flex; justify-content: space-between; align-items: center; position: sticky; top: 0; z-index: 1000; }
        .header h1 { margin: 0; font-size: 18px; }
        
        .container { padding: 12px; max-width: 650px; margin: auto; }
        
        /* Worker Card */
        .worker-card { background: #fff; border-radius: 12px; padding: 12px; margin-bottom: 12px; display: flex; align-items: center; box-shadow: 0 2px 8px rgba(0,0,0,0.1); border-left: 5px solid var(--success); position: relative;}
        .worker-card.busy { border-left-color: var(--danger); opacity: 0.8; }
        .profile-img { width: 70px; height: 70px; border-radius: 8px; object-fit: cover; margin-right: 12px; background: #eee; }
        .worker-info { flex-grow: 1; }
        .worker-info b { font-size: 16px; color: #1e293b; }
        .loc-badge { background: #e2e8f0; font-size: 10px; padding: 2px 6px; border-radius: 4px; font-weight: bold; }
        .loc-special { background: #cce5ff; color: #004085; }
        
        /* Side Menus */
        .side-menu { position: fixed; top: 0; width: 85%; max-width: 380px; height: 100%; background: white; z-index: 2000; transition: 0.3s; padding: 15px; box-sizing: border-box; overflow-y: auto; box-shadow: 0 0 20px rgba(0,0,0,0.3); }
        .left-menu { left: -400px; border-right: 4px solid var(--primary); }
        .right-menu { right: -400px; border-left: 4px solid var(--success); }
        .open-left { left: 0; }
        .open-right { right: 0; }
        .overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.5); display: none; z-index: 1500; }

        /* Form Inputs */
        input, select, textarea { width: 100%; padding: 10px; margin-bottom: 10px; border: 1px solid #ddd; border-radius: 6px; font-size: 14px; box-sizing: border-box; }
        button { cursor: pointer; border: none; border-radius: 6px; font-weight: bold; padding: 10px; }
        .admin-btn { background: var(--primary); color: white; width: 100%; margin-bottom: 10px; }
        
        /* Rate Table */
        .rate-table { width: 100%; font-size: 10px; border-collapse: collapse; margin-top: 10px; }
        .rate-table th { background: #f8fafc; padding: 6px 2px; border: 1px solid #e2e8f0; }
        .rate-table td { padding: 4px 2px; border: 1px solid #e2e8f0; text-align: center; }
        .rate-in { width: 42px; padding: 4px 2px; border: 1px solid #cbd5e1; text-align: center; border-radius: 4px; font-size: 10px; }
        
        /* Photo Preview */
        .photo-preview { width: 100%; height: 120px; background: #f1f5f9; border: 2px dashed #cbd5e1; border-radius: 8px; margin-bottom: 10px; display: flex; align-items: center; justify-content: center; overflow: hidden; }
        .photo-preview img { width: 100%; height: 100%; object-fit: cover; }

        /* Chat System */
        .chat-btn { position: fixed; bottom: 20px; right: 20px; background: var(--primary); color: white; width: 60px; height: 60px; border-radius: 50%; font-size: 24px; box-shadow: 0 4px 10px rgba(0,0,0,0.3); z-index: 900; }
        #chatPage { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: var(--bg); z-index: 3000; display: none; flex-direction: column; }
        .chat-header { background: #1e293b; color: white; padding: 15px; display: flex; justify-content: space-between; align-items: center; }
        #chatBox { flex-grow: 1; padding: 15px; overflow-y: auto; display: flex; flex-direction: column; gap: 10px; }
        .msg { max-width: 80%; padding: 10px; border-radius: 8px; background: white; box-shadow: 0 1px 3px rgba(0,0,0,0.1); }
        .msg.me { align-self: flex-end; background: #dcf8c6; }
        .msg-name { font-size: 10px; color: #555; font-weight: bold; margin-bottom: 4px; }
        .chat-input-area { background: white; padding: 10px; display: flex; gap: 10px; align-items: center; }
    </style>
</head>
<body>

<div id="installBanner">⬇️ Install Worker App for better experience! <button onclick="this.parentElement.style.display='none'" style="padding:2px 5px;">X</button></div>

<div class="header">
    <button onclick="toggleMenu('left')" style="background:none; color:white; font-size:22px;">⚙️</button>
    <h1>Worker App</h1>
    <button onclick="toggleMenu('right')" style="background:none; color:white; font-size:22px;">👤</button>
</div>

<div class="overlay" id="overlay" onclick="closeMenus()"></div>

<div class="side-menu left-menu" id="leftMenu">
    <h3>⚙️ Admin / Setting</h3>
    <div id="adminLoginArea">
        <input type="password" id="adminPass" placeholder="Admin Password">
        <button onclick="checkAdmin()" class="admin-btn">Login</button>
    </div>

    <div id="adminPanel" style="display:none;">
        <button onclick="logoutAdmin()" style="background:var(--danger); color:white; width:100%; margin-bottom:10px;">Logout</button>
        <textarea id="adminNote" placeholder="Naya Notice yahan likhein..."></textarea>
        <button onclick="saveNote()" class="admin-btn">Post Update</button>
        
        <hr>
        <b>📁 Categories</b>
        <div style="display:flex; gap:5px; margin-bottom:10px;">
            <input type="text" id="newCat" placeholder="Nayi Category">
            <button onclick="addCat()" style="background:var(--success); color:white;">Add</button>
        </div>
        <div id="catEditor" style="font-size:12px;"></div>
    </div>

    <div style="background:#fff; padding:10px; border-radius:8px; margin-top:15px; border:1px solid #ddd;">
        <h4>📒 Judai Rate Chart</h4>
        <table class="rate-table">
            <thead>
                <tr><th>Taag</th><th>Cott</th><th>C-Ang</th><th>PC</th><th>P-Ang</th></tr>
            </thead>
            <tbody id="rateBody"></tbody>
        </table>
        <button id="saveRateBtn" onclick="saveRates()" class="admin-btn" style="display:none; margin-top:10px;">Save Rates</button>
    </div>
</div>

<div class="side-menu right-menu" id="rightMenu">
    <h3>📝 My Profile</h3>
    
    <div class="photo-preview" onclick="document.getElementById('photoInput').click()">
        <span id="photoText">📷 Select from Gallery</span>
        <img id="previewImg" style="display:none;">
    </div>
    <input type="file" id="photoInput" accept="image/*" style="display:none;" onchange="previewPhoto(this)">
    
    <input type="text" id="wName" placeholder="Aapka Naam">
    <input type="number" id="wPhone" placeholder="Mobile Number">
    <select id="wLoc"></select>
    <select id="wCat"></select>
    
    <button onclick="saveProfile()" style="background:var(--success); color:white; width:100%; padding:14px; margin-bottom:15px;">SAVE PROFILE</button>
    
    <div id="myStatusArea" style="display:none; background:#f8fafc; padding:10px; border-radius:8px; border:1px solid #ddd;">
        <b>🔴 Set Busy Timer:</b>
        <div style="display:flex; gap:5px; margin-top:5px;">
            <select id="busyHours">
                <option value="2.5">Quick (2h 30m)</option>
                <option value="1">1 Hour</option><option value="5">5 Hours</option>
                <option value="12">12 Hours</option><option value="24">24 Hours</option>
                <option value="999">Holiday (Manual)</option>
            </select>
            <button onclick="setBusy()" style="background:var(--danger); color:white;">Set</button>
        </div>
        <button onclick="setFree()" style="background:var(--success); color:white; width:100%; margin-top:10px;">🟢 I am FREE Now</button>
    </div>
</div>

<div class="container">
    <div id="announcementArea"></div>
    <select id="catFilter" onchange="fetchWorkers()" style="font-weight:bold; font-size:16px; border:2px solid var(--primary); background:#fff;"></select>
    <div id="workerList" style="margin-top:15px;">Loading...</div>
</div>

<button class="chat-btn" onclick="openChat()">💬</button>

<div id="chatPage">
    <div class="chat-header">
        <h3 style="margin:0;">Community Chat</h3>
        <button onclick="closeChat()" style="background:none; color:white; font-size:18px;">❌</button>
    </div>
    <div id="chatBox"></div>
    <div class="chat-input-area">
        <input type="file" id="chatPhoto" accept="image/*" style="display:none;" onchange="sendChatPhoto(this)">
        <button onclick="document.getElementById('chatPhoto').click()" style="background:#e2e8f0; padding:10px 15px;">📷</button>
        <input type="text" id="chatMsg" placeholder="Message likhein..." style="margin:0;">
        <button onclick="sendChatMsg()" style="background:var(--primary); color:white;">Send</button>
    </div>
</div>

<script>
    // 1. FIREBASE SETUP
    const firebaseConfig = { apiKey: "AIzaSyAi-eed_fIWaj3GQVleMzD7Fz4ppS_voxQ", databaseURL: "https://gathni-app-default-rtdb.firebaseio.com" };
    firebase.initializeApp(firebaseConfig);
    const db = firebase.database();

    // 2. LOCATIONS ARRAY
    const locations = ["All Over Malegaon", "Madni Nagar", "Highway-Touch Area (Agra Road)", "Nayapura", "Islampura", "Azad Nagar", "Gulshanabad", "Diamond Nagar", "Golden Nagar", "Motipura", "Pawar Wadi", "Jafar Nagar", "Ramzanpura", "Ayesha Nagar", "Zaitun Pura", "Millat Nagar", "Chandanpuri Gate", "Dyane", "Malde", "Soygaon", "Daregaon", "Al-Hilal Colony", "Shanti Nagar", "Sardar Nagar", "Qidwai Nagar", "Rehmat Nagar", "Hashim Nagar", "Bilal Nagar", "Farooqui Nagar", "Mehfooz Nagar", "Baig Plot", "Ghaffar File", "Sulemani Chowk", "Tipu Sultan Chowk", "Ansar Nagar", "Faisal Nagar", "Iqbal Road", "Ismail Nagar", "Mausam Pul Area", "Girna Colony (Industrial)", "Madina Chowk", "Aman Chowk", "Khadda Chowk", "Noor Hospital Area", "Old Agra Road Units", "Kusumba Road Units", "Mausam River Side", "Sahyog Nagar", "Rehmani Nagar", "Qubool Nagar", "Ward No 1 to 12 (Textile Zones)", "Malegaon Industrial Estate", "Sangameshwar", "Shivaji Putla Area", "Hazrat Bilal Nagar", "Hyder Ali Road"];
    
    let locHtml = "";
    locations.forEach(l => locHtml += `<option value="${l}">${l}</option>`);
    document.getElementById('wLoc').innerHTML = locHtml;

    // 3. GLOBAL VARIABLES
    let base64Photo = "";
    let myWorkerId = localStorage.getItem('my_worker_id');
    let isAdmin = localStorage.getItem('admin_auth') === 'true';

    // PWA Sim
    setTimeout(() => { document.getElementById('installBanner').style.display = 'block'; }, 2000);

    // BUILD RATE TABLE (2000-3400)
    let rBody = "";
    for(let t=2000; t<=3400; t+=100) {
        rBody += `<tr><td><b>${t}</b></td>
            <td><input type="number" id="c${t}" class="rate-in" disabled></td>
            <td><input type="number" id="ca${t}" class="rate-in" disabled></td>
            <td><input type="number" id="p${t}" class="rate-in" disabled></td>
            <td><input type="number" id="pa${t}" class="rate-in" disabled></td></tr>`;
    }
    document.getElementById('rateBody').innerHTML = rBody;

    // 4. ADMIN LOGIC
    function checkAdmin() {
        if(document.getElementById('adminPass').value === 'malegaongatni.2026') {
            localStorage.setItem('admin_auth', 'true'); isAdmin = true; location.reload();
        } else { alert("Wrong Password!"); }
    }
    function logoutAdmin() { localStorage.removeItem('admin_auth'); location.reload(); }
    
    if(isAdmin) {
        document.getElementById('adminLoginArea').style.display = 'none';
        document.getElementById('adminPanel').style.display = 'block';
        document.getElementById('saveRateBtn').style.display = 'block';
        document.querySelectorAll('.rate-in').forEach(i => i.disabled = false);
    }

    // 5. MENU & PHOTO LOGIC
    function toggleMenu(s) { 
        document.getElementById(s+'Menu').classList.add('open-'+s); 
        document.getElementById('overlay').style.display = 'block'; 
        if(s==='left') loadRates();
        if(s==='right' && myWorkerId) {
            document.getElementById('myStatusArea').style.display = 'block';
            loadMyProfile();
        }
    }
    function closeMenus() { 
        document.getElementById('leftMenu').classList.remove('open-left'); 
        document.getElementById('rightMenu').classList.remove('open-right'); 
        document.getElementById('overlay').style.display = 'none'; 
    }
    
    function previewPhoto(input) {
        if (input.files && input.files[0]) {
            let r = new FileReader();
            r.onload = function(e) {
                document.getElementById('previewImg').src = e.target.result;
                document.getElementById('previewImg').style.display = 'block';
                document.getElementById('photoText').style.display = 'none';
                base64Photo = e.target.result; 
            };
            r.readAsDataURL(input.files[0]);
        }
    }

    // 6. CATEGORIES & RATES
    db.ref('categories').on('value', snap => {
        let cats = snap.val() || { "default": "Gathni / Taag Jodne" };
        let opt = "", ed = "";
        for(let id in cats) {
            opt += `<option value="${cats[id]}">${cats[id]}</option>`;
            ed += `<div style="display:flex; justify-content:space-between; border-bottom:1px solid #eee; padding:5px 0;">${cats[id]} <button onclick="delCat('${id}')" style="color:red; background:none; padding:0;">X</button></div>`;
        }
        document.getElementById('wCat').innerHTML = opt;
        document.getElementById('catFilter').innerHTML = "<option value='All'>All Categories</option>" + opt;
        document.getElementById('catEditor').innerHTML = ed;
        fetchWorkers();
    });
    function addCat() { let v = document.getElementById('newCat').value; if(v) db.ref('categories').push(v).then(()=>document.getElementById('newCat').value=""); }
    function delCat(id) { if(confirm("Delete Category?")) db.ref('categories/'+id).remove(); }

    function saveRates() {
        let r = {};
        for(let t=2000; t<=3400; t+=100) { r['c'+t]=document.getElementById('c'+t).value; r['ca'+t]=document.getElementById('ca'+t).value; r['p'+t]=document.getElementById('p'+t).value; r['pa'+t]=document.getElementById('pa'+t).value; }
        db.ref('rates').set(r).then(()=>alert("Saved!"));
    }
    function loadRates() {
        db.ref('rates').once('value', snap => {
            let d = snap.val(); if(!d) return;
            for(let t=2000; t<=3400; t+=100) { document.getElementById('c'+t).value=d['c'+t]||""; document.getElementById('ca'+t).value=d['ca'+t]||""; document.getElementById('p'+t).value=d['p'+t]||""; document.getElementById('pa'+t).value=d['pa'+t]||""; }
        });
    }
    function saveNote() { db.ref('note').set(document.getElementById('adminNote').value); }
    db.ref('note').on('value', snap => {
        let v = snap.val();
        document.getElementById('announcementArea').innerHTML = v ? `<div style="background:#fff3cd; padding:10px; border-radius:6px; margin-bottom:10px;">📢 ${v}</div>` : "";
    });

    // 7. PROFILE LOGIC
    function saveProfile() {
        let data = {
            name: document.getElementById('wName').value, phone: document.getElementById('wPhone').value,
            loc: document.getElementById('wLoc').value, cat: document.getElementById('wCat').value,
            photo: base64Photo, calls: 0, busyUntil: 0
        };
        if(!data.name || !data.phone) return alert("Naam aur Mobile required hai!");
        
        if(myWorkerId) { db.ref('workers/'+myWorkerId).update(data).then(()=>{ alert("Updated!"); closeMenus(); }); }
        else {
            let ref = db.ref('workers').push();
            ref.set(data).then(()=>{ localStorage.setItem('my_worker_id', ref.key); alert("Profile Created!"); location.reload(); });
        }
    }
    function loadMyProfile() {
        db.ref('workers/'+myWorkerId).once('value', snap => {
            let d = snap.val(); if(!d) return;
            document.getElementById('wName').value = d.name; document.getElementById('wPhone').value = d.phone;
            document.getElementById('wLoc').value = d.loc; document.getElementById('wCat').value = d.cat;
            if(d.photo) { document.getElementById('previewImg').src = d.photo; document.getElementById('previewImg').style.display = 'block'; document.getElementById('photoText').style.display='none'; base64Photo = d.photo; }
        });
    }
    function deleteMyProfile() {
        if(confirm("Profile delete karni hai?")) {
            db.ref('workers/'+myWorkerId).remove().then(()=>{ localStorage.removeItem('my_worker_id'); location.reload(); });
        }
    }

    // 8. STATUS & SORTING LOGIC
    function setBusy() {
        let hrs = parseFloat(document.getElementById('busyHours').value);
        let until = hrs === 999 ? 9999999999999 : Date.now() + (hrs * 3600000);
        db.ref('workers/'+myWorkerId).update({busyUntil: until, calls: 0}).then(()=>{ alert("Status: BUSY"); closeMenus(); });
    }
    function setFree() { db.ref('workers/'+myWorkerId).update({busyUntil: 0, calls: 0}).then(()=>{ alert("Status: FREE"); closeMenus(); }); }
    
    function makeCall(id, currentCalls) {
        let newCount = (currentCalls || 0) + 1;
        let update = { calls: newCount };
        if(newCount >= 3) { update.busyUntil = Date.now() + (24 * 3600000); update.calls = 0; alert("Is number par bahut call gaye hain. Ye 24 ghante ke liye busy set ho gaya hai."); }
        db.ref('workers/'+id).update(update);
        window.location.href = `tel:${event.target.dataset.phone}`;
    }

    function fetchWorkers() {
        let filter = document.getElementById('catFilter').value;
        db.ref('workers').on('value', snap => {
            let list = document.getElementById('workerList'); list.innerHTML = "";
            let data = snap.val(); if(!data) return;
            
            let workersArray = [];
            for(let id in data) { let w = data[id]; w.id = id; workersArray.push(w); }
            
            // Sort: Free top, Busy bottom
            workersArray.sort((a, b) => {
                let aBusy = a.busyUntil > Date.now(); let bBusy = b.busyUntil > Date.now();
                return (aBusy === bBusy) ? 0 : aBusy ? 1 : -1;
            });

            workersArray.forEach(w => {
                if(filter === 'All' || w.cat === filter) {
                    let isBusy = w.busyUntil > Date.now();
                    let locName = w.loc.length > 15 ? w.loc.substring(0,15)+'...' : w.loc;
                    let isSpecialLoc = w.loc === 'All Over Malegaon' || w.loc.includes('Highway');
                    
                    let card = `<div class="worker-card ${isBusy?'busy':''}">
                        <img src="${w.photo || 'https://cdn-icons-png.flaticon.com/512/3135/3135715.png'}" class="profile-img">
                        <div class="worker-info">
                            <b>${w.name}</b><br>
                            <span class="loc-badge ${isSpecialLoc?'loc-special':''}">${locName}</span>
                            <div style="font-size:11px; color:${isBusy?'red':'green'}; margin-top:4px;"><b>${isBusy?'🔴 BUSY':'🟢 FREE'}</b></div>
                        </div>
                        <div style="display:flex; flex-direction:column; gap:5px;">`;
                            if(!isBusy) card += `<button data-phone="${w.phone}" onclick="makeCall('${w.id}', ${w.calls})" style="background:var(--success); color:white; padding:6px 12px; font-size:12px;">📞 Call</button>`;
                            if(w.id === myWorkerId) card += `<button onclick="toggleMenu('right')" style="background:var(--warning); padding:4px; font-size:10px;">Edit</button> <button onclick="deleteMyProfile()" style="background:var(--danger); color:white; padding:4px; font-size:10px;">Delete</button>`;
                    card += `</div></div>`;
                    list.innerHTML += card;
                }
            });
        });
    }

    // 9. COMMUNITY CHAT
    function openChat() { document.getElementById('chatPage').style.display = 'flex'; loadChat(); }
    function closeChat() { document.getElementById('chatPage').style.display = 'none'; }
    
    function loadChat() {
        db.ref('chat').on('value', snap => {
            let box = document.getElementById('chatBox'); box.innerHTML = "";
            let d = snap.val(); if(!d) return;
            for(let key in d) {
                let m = d[key];
                let isMe = m.senderId === myWorkerId;
                box.innerHTML += `<div class="msg ${isMe?'me':''}">
                    <div class="msg-name">${m.name}</div>
                    ${m.img ? `<img src="${m.img}" style="width:100%; border-radius:4px; margin-bottom:5px;">` : ''}
                    ${m.text ? `<div>${m.text}</div>` : ''}
                </div>`;
            }
            box.scrollTop = box.scrollHeight;
        });
    }
    
    function sendChatPhoto(input) {
        if(input.files && input.files[0]) {
            let r = new FileReader();
            r.onload = function(e) { postChatMsg("", e.target.result); };
            r.readAsDataURL(input.files[0]);
        }
    }
    function sendChatMsg() {
        let txt = document.getElementById('chatMsg').value;
        if(txt) { postChatMsg(txt, ""); document.getElementById('chatMsg').value = ""; }
    }
    function postChatMsg(txt, imgStr) {
        let senderName = "Guest";
        if(myWorkerId) senderName = document.getElementById('wName').value || "Worker";
        db.ref('chat').push({ senderId: myWorkerId || 'guest', name: senderName, text: txt, img: imgStr, time: Date.now() });
    }

</script>
</body>
</html>
