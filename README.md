

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Papan Ucapan Natal Interaktif</title>
    
    <link href="https://fonts.googleapis.com/css2?family=Mountains+of+Christmas:wght@700&family=Poppins:wght@400;600&display=swap" rel="stylesheet">

    <style>
        /* --- CSS STYLING --- */
        :root {
            --red: #D42426;
            --green: #2F5233;
            --gold: #F1D570;
            --cream: #F1F1F1;
        }

        body {
            margin: 0;
            padding: 0;
            background-color: var(--green);
            background-image: radial-gradient(circle, #406e45 1px, transparent 1px);
            background-size: 20px 20px;
            font-family: 'Poppins', sans-serif;
            overflow: hidden; /* Mencegah scroll berlebih */
            height: 100vh;
            width: 100vw;
            color: #333;
            cursor: crosshair; /* Indikator bisa klik */
        }

        /* Judul Halaman */
        h1 {
            font-family: 'Mountains of Christmas', cursive;
            color: var(--gold);
            text-align: center;
            font-size: 3rem;
            margin-top: 20px;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.5);
            pointer-events: none;
            z-index: 0;
        }

        /* Efek Salju Sederhana */
        .snowflake {
            position: absolute;
            top: -10px;
            color: #FFF;
            font-size: 1em;
            animation: fall linear infinite;
            z-index: 0;
            pointer-events: none;
        }

        @keyframes fall {
            to { transform: translateY(105vh); }
        }

        /* --- UI CONTROLS --- */
        #controls {
            position: fixed;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            background: rgba(255, 255, 255, 0.9);
            padding: 15px 25px;
            border-radius: 50px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.3);
            display: flex;
            gap: 15px;
            z-index: 1000;
            align-items: center;
            cursor: default;
        }

        .instruction {
            font-size: 0.9rem;
            font-weight: 600;
            color: var(--green);
        }

        /* --- KARTU UCAPAN (STICKY NOTES) --- */
        .card {
            position: absolute;
            width: 180px;
            padding: 15px;
            border-radius: 8px; /* Sedikit melengkung */
            box-shadow: 3px 3px 10px rgba(0,0,0,0.3);
            font-size: 0.85rem;
            animation: popIn 0.3s ease-out;
            cursor: pointer;
            transition: transform 0.2s;
            z-index: 10;
        }

        .card:hover {
            transform: scale(1.05);
            z-index: 20;
        }

        .card strong {
            display: block;
            margin-bottom: 5px;
            font-size: 1rem;
            border-bottom: 1px solid rgba(0,0,0,0.1);
            padding-bottom: 3px;
        }

        .card-red { background-color: var(--red); color: white; transform: rotate(-3deg); }
        .card-gold { background-color: var(--gold); color: var(--green); transform: rotate(2deg); }
        .card-white { background-color: white; color: var(--red); transform: rotate(1deg); }

        @keyframes popIn {
            from { transform: scale(0); }
            to { transform: scale(1) rotate(var(--rotation)); }
        }

        /* --- MODAL INPUT --- */
        #inputModal {
            display: none;
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: white;
            padding: 25px;
            border-radius: 15px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.5);
            z-index: 2000;
            width: 300px;
            text-align: center;
            border: 4px solid var(--red);
            cursor: default;
        }

        #inputModal h3 { margin-top: 0; color: var(--green); }
        
        input, textarea, select {
            width: 100%;
            padding: 10px;
            margin: 8px 0;
            border: 1px solid #ddd;
            border-radius: 5px;
            box-sizing: border-box;
            font-family: 'Poppins', sans-serif;
        }

        button.btn-save {
            background-color: var(--green);
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 20px;
            cursor: pointer;
            font-weight: bold;
            margin-top: 10px;
        }
        
        button.btn-cancel {
            background-color: #ccc;
            color: #333;
            border: none;
            padding: 10px 20px;
            border-radius: 20px;
            cursor: pointer;
            margin-top: 10px;
        }

        /* Overlay gelap saat modal muncul */
        #overlay {
            display: none;
            position: fixed;
            top: 0; left: 0; right: 0; bottom: 0;
            background: rgba(0,0,0,0.5);
            z-index: 1500;
        }

    </style>
</head>
<body>

    <h1>🎄 Wall of Christmas Wishes 🎄</h1>

    <div id="card-container"></div>

    <div id="controls">
        <span class="instruction">Klik di mana saja pada layar untuk menempelkan kartu ucapan!</span>
    </div>

    <div id="overlay"></div>
    <div id="inputModal">
        <h3>Tulis Ucapanmu 🎅</h3>
        <input type="text" id="senderName" placeholder="Nama Anda" maxlength="15">
        <textarea id="messageText" rows="3" placeholder="Ucapan Natal..." maxlength="80"></textarea>
        <select id="cardColor">
            <option value="card-red">Merah Festive</option>
            <option value="card-gold">Emas Mewah</option>
            <option value="card-white">Putih Salju</option>
        </select>
        <button class="btn-save" onclick="saveCard()">Tempel Kartu!</button>
        <button class="btn-cancel" onclick="closeModal()">Batal</button>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/9.22.0/firebase-app.js";
        import { getDatabase, ref, push, onChildAdded } from "https://www.gstatic.com/firebasejs/9.22.0/firebase-database.js";

        // 2. KONFIGURASI FIREBASE ANDA DI SINI
        // Ganti dengan config dari Firebase Console Anda
        const firebaseConfig = {
        apiKey: "AIzaSyDGX1vYxH-WxripA52ImncLe7NM1ozqOwk",
        authDomain: "kartu-ucapan-74469.firebaseapp.com",
        databaseURL: "https://kartu-ucapan-74469-default-rtdb.firebaseio.com",
        projectId: "kartu-ucapan-74469",
        storageBucket: "kartu-ucapan-74469.firebasestorage.app",
       messagingSenderId: "666565058284",
       appId: "1:666565058284:web:c156b770a95de851350fc9",
       measurementId: "G-GP8K39VW5W"
        
        };

        // Inisialisasi Firebase
        let app, db, cardsRef;
        
        try {
            app = initializeApp(firebaseConfig);
            db = getDatabase(app);
            cardsRef = ref(db, 'wishes'); // Nama folder di database
            console.log("Firebase Connected!");
        } catch (e) {
            console.error("Error Firebase. Pastikan Config sudah diisi!", e);
            alert("Harap isi konfigurasi Firebase di dalam kode source code agar fitur berjalan.");
        }

        // --- LOGIKA APLIKASI ---

        let tempClickPos = { x: 0, y: 0 };
        const modal = document.getElementById('inputModal');
        const overlay = document.getElementById('overlay');

        // Event Listener untuk klik Background
        document.body.addEventListener('click', (e) => {
            // Cegah modal muncul jika klik pada kontrol atau modal itu sendiri
            if (e.target.closest('#controls') || e.target.closest('#inputModal') || e.target.closest('.card')) return;

            tempClickPos.x = e.clientX;
            tempClickPos.y = e.clientY;
            
            openModal();
        });

        // Fungsi Global agar bisa dipanggil HTML (karena type="module")
        window.openModal = () => {
            modal.style.display = 'block';
            overlay.style.display = 'block';
            document.getElementById('senderName').focus();
        };

        window.closeModal = () => {
            modal.style.display = 'none';
            overlay.style.display = 'none';
            document.getElementById('senderName').value = '';
            document.getElementById('messageText').value = '';
        };

        window.saveCard = () => {
            const name = document.getElementById('senderName').value;
            const msg = document.getElementById('messageText').value;
            const color = document.getElementById('cardColor').value;

            if (!name || !msg) {
                alert("Mohon isi nama dan ucapan!");
                return;
            }

            // Kirim ke Firebase
            if(push && cardsRef) {
                push(cardsRef, {
                    name: name,
                    message: msg,
                    type: color,
                    x: tempClickPos.x,
                    y: tempClickPos.y,
                    timestamp: Date.now()
                });
            } else {
                alert("Firebase belum dikonfigurasi dengan benar.");
            }

            closeModal();
        };

        // --- MENAMPILKAN KARTU DARI FIREBASE (REAL-TIME) ---
        if(onChildAdded && cardsRef) {
            onChildAdded(cardsRef, (snapshot) => {
                const data = snapshot.val();
                renderCard(data);
            });
        }

        function renderCard(data) {
            const el = document.createElement('div');
            el.className = `card ${data.type}`;
            
            // Set posisi
            el.style.left = data.x + 'px';
            el.style.top = data.y + 'px';
            
            // Random sedikit rotasi agar terlihat alami
            const randomRot = (Math.random() * 10 - 5); 
            el.style.transform = `rotate(${randomRot}deg)`;

            el.innerHTML = `
                <strong>${sanitize(data.name)}</strong>
                <p>${sanitize(data.message)}</p>
            `;

            document.getElementById('card-container').appendChild(el);
        }

        // Mencegah XSS sederhana
        function sanitize(str) {
            const temp = document.createElement('div');
            temp.textContent = str;
            return temp.innerHTML;
        }

        // --- EFEK SALJU (Visual Only) ---
        function createSnowflake() {
            const snow = document.createElement('div');
            snow.classList.add('snowflake');
            snow.innerText = '❄';
            snow.style.left = Math.random() * 100 + 'vw';
            snow.style.animationDuration = Math.random() * 3 + 2 + 's';
            snow.style.opacity = Math.random();
            snow.style.fontSize = Math.random() * 10 + 10 + 'px';
            
            document.body.appendChild(snow);
            
            setTimeout(() => {
                snow.remove();
            }, 5000);
        }

        setInterval(createSnowflake, 200);

    </script>
</body>

