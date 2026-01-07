<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Kasir RM JOYO - Jajanan & Hapus Menu</title>
<style>
body{margin:0;font-family:Arial,sans-serif;background:#f2f2f2;color:#000;transition:0.3s;padding-top:24px;}
body.dark{background:#121212;color:#eee;}
.page{display:none;padding:15px;}
.page.active{display:block;}
button{width:100%;padding:10px;margin:5px 0;border:none;border-radius:10px;background:#c62828;color:white;font-size:16px;cursor:pointer;}
body.dark button{background:#b71c1c;}
input, select{width:100%;padding:8px;margin:5px 0;border:1px solid #ccc;border-radius:6px;font-size:14px;}
body.dark input, body.dark select{background:#333;color:#eee;border-color:#888;}
.kasir,.riwayat-container,.rekap-container,.login-container{width:90%;max-width:700px;margin:15px auto;background:#fff;padding:15px;border-radius:14px;box-shadow:0 6px 20px rgba(0,0,0,.15);}
body.dark .kasir,body.dark .riwayat-container,body.dark .rekap-container,body.dark .login-container{background:#1e1e1e;color:#eee;}
.header{text-align:center;margin-bottom:10px;}
.kategori-title{margin:15px 0 5px 0;font-weight:bold;font-size:18px;color:#c62828;border-bottom:2px solid #c62828;display:inline-block;}
.menu-item{padding:8px 12px;margin-bottom:5px;background:#f9f9f9;border-radius:8px;display:flex;justify-content:space-between;align-items:center;cursor:pointer;border:1px solid #eee;}
body.dark .menu-item{background:#2a2a2a;border-color:#444;}
.menu-info{flex:2;display:flex;justify-content:space-between;margin-right:15px;}
.del-menu{color:#bbb;font-size:14px;padding:5px;cursor:pointer;}
.del-menu:hover{color:red;}
.order-row {display:flex; justify-content:space-between; align-items:center; padding:8px 0; border-bottom:1px solid #eee;}
.hapus-btn {color:#d32f2f; font-weight:bold; font-size:20px; cursor:pointer; padding:0 10px;}
.marquee-container {position: fixed;top:0; left:0; width:100%;overflow:hidden; white-space:nowrap;font-size:0.8em; font-weight:bold;z-index:1000;}
.marquee-text{display:inline-block;padding-left:100%;animation: scroll 15s linear infinite;}
@keyframes scroll {0% { transform: translateX(0%);} 100% { transform: translateX(-100%);}}
</style>
</head>
<body>

<div class="marquee-container">
  <div class="marquee-text" id="runningText">Loading...</div>
</div>

<div id="pageLogin" class="page active">
  <div class="login-container">
    <div class="header"><h2>LOGIN KASIR</h2></div>
    <input type="text" id="username" placeholder="Username">
    <input type="password" id="password" placeholder="Password">
    <button onclick="login()">Masuk</button>
  </div>
</div>

<div id="pageKasir" class="page">
  <div class="kasir">
    <div class="header"><h2 id="displayNamaToko">RM JOYO</h2><div id="waktu"></div></div>
    <div id="menuContainer"></div>
    <hr>
    <h3 style="text-align:center;">Daftar Pesanan</h3>
    <div id="list"></div>
    <div style="text-align:center; font-weight:bold; font-size:20px; margin:15px 0;">Total: Rp <span id="total">0</span></div>
    <button onclick="bayar()">BAYAR & SIMPAN NOTA</button>
    <button style="background:#444;" onclick="showPage('pageRiwayat')">⚙ Update Nama & Menu</button>
    <button style="background:#444;" onclick="showPage('pageRekap')">📋 Lihat Rekap Nota</button>
  </div>
</div>

<div id="pageRiwayat" class="page">
  <div class="riwayat-container">
    <div class="header"><h3>⚙ Pengaturan Identitas</h3></div>
    <div style="background:#f9f9f9; padding:10px; border-radius:10px; margin-bottom:15px; color:#333; border:1px solid #ddd;">
      <input type="text" id="inputNamaToko" placeholder="Nama Toko">
      <input type="text" id="inputNamaKasir" placeholder="Nama Kasir">
      <button onclick="updateIdentitas()" style="background:#0288d1;">SIMPAN PERUBAHAN NAMA</button>
    </div>
    <hr>
    <div class="header"><h3>Tambah Menu Baru</h3></div>
    <div style="display:flex; gap:5px; margin-bottom:10px; flex-wrap:wrap;">
      <input type="text" id="namaBaru" placeholder="Nama Menu" style="flex:2; min-width:150px;">
      <input type="number" id="hargaBaru" placeholder="Harga" style="flex:1;">
      <select id="kategoriBaru" style="flex:1;">
        <option value="Makanan">Makanan</option>
        <option value="Minuman">Minuman</option>
        <option value="Jajanan">Jajanan</option>
      </select>
      <button onclick="tambahMenuBaru()" style="background:#2e7d32;">Tambah</button>
    </div>
    <button id="darkModeBtn" onclick="toggleDark()">Dark Mode: OFF</button>
    <button onclick="resetRiwayat()" style="background:#d32f2f;">Hapus Semua Transaksi</button>
    <button onclick="showPage('pageKasir')">Kembali ke Kasir</button>
    <div id="riwayat" style="margin-top:15px; border-top:1px solid #ccc;"></div>
  </div>
</div>

<div id="pageRekap" class="page">
  <div class="rekap-container">
    <div class="header"><h2>Rekap Per Nota</h2></div>
    <button onclick="showPage('pageKasir')">Kembali ke Kasir</button>
    <div id="rekapDiv"></div>
  </div>
</div>

<audio id="suaraKlik" src="https://www.soundjay.com/buttons/sounds/button-16.mp3" preload="auto"></audio>
<audio id="suaraBayar" src="https://www.soundjay.com/buttons/sounds/button-3.mp3" preload="auto"></audio>

<script>
let menuData = JSON.parse(localStorage.getItem("menuRM")) || [
  {nama:"Nasi Goreng",harga:15000,kategori:"Makanan"},
  {nama:"Teh Manis",harga:5000,kategori:"Minuman"},
  {nama:"Krupuk",harga:2000,kategori:"Jajanan"}
];
let identitas = JSON.parse(localStorage.getItem("identitasRM")) || { toko: "RM JOYO", kasir: "KOES" };
let pesananData = [], darkMode = JSON.parse(localStorage.getItem("darkMode")) || false;

const playSfx = (id) => { const a = document.getElementById(id); if(a){a.currentTime=0; a.play();} };

function applyIdentitas() {
    document.getElementById("displayNamaToko").innerText = identitas.toko;
    document.getElementById("inputNamaToko").value = identitas.toko;
    document.getElementById("inputNamaKasir").value = identitas.kasir;
}
applyIdentitas();

function updateIdentitas() {
    identitas.toko = document.getElementById("inputNamaToko").value || "RM JOYO";
    identitas.kasir = document.getElementById("inputNamaKasir").value || "KOES";
    localStorage.setItem("identitasRM", JSON.stringify(identitas));
    applyIdentitas();
    alert("Identitas Berhasil Diperbarui!");
}

function showPage(id){
  document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));
  document.getElementById(id).classList.add('active');
  if(id === 'pageRiwayat') loadRiwayat();
  if(id === 'pageRekap') loadRekap();
}

function login(){
  const u = document.getElementById("username").value;
  const p = document.getElementById("password").value;
  if((u==="kasir" && p==="3/3/2026 000") || (u==="777" && p==="000") || (u==="bos" && p==="777")){
    showPage("pageKasir");
    renderMenu();
  } else { alert("Login Gagal!"); }
}

function toggleDark(){
  darkMode = !darkMode; localStorage.setItem("darkMode", darkMode);
  document.body.classList.toggle("dark");
  document.getElementById("darkModeBtn").innerText = `Dark Mode: ${darkMode?'ON':'OFF'}`;
}

function updateWaktu(){
  const d = new Date();
  document.getElementById("waktu").innerText = d.toLocaleString('id-ID');
  const jam = d.getHours();
  let salam = jam < 11 ? "PAGI" : jam < 15 ? "SIANG" : jam < 18 ? "SORE" : "MALAM";
  document.getElementById("runningText").innerText = `${salam}, ${identitas.kasir.toUpperCase()} @ ${identitas.toko.toUpperCase()}... SEMANGAT SEMOGA SUKSES!`;
}
setInterval(updateWaktu, 1000);

function renderMenu(){
  const container = document.getElementById("menuContainer");
  container.innerHTML = "";
  ["Makanan","Minuman","Jajanan"].forEach(kat => {
    const list = menuData.filter(m => m.kategori === kat);
    if(list.length > 0){
      container.innerHTML += `<div class="kategori-title">${kat}</div>`;
      list.forEach((m, indexInMenu) => {
        const item = document.createElement("div");
        item.className = "menu-item";
        item.innerHTML = `
            <div class="menu-info" onclick="tambahItem('${m.nama}', ${m.harga})">
                <span>${m.nama}</span>
                <span>Rp ${m.harga.toLocaleString()}</span>
            </div>
            <div class="del-menu" onclick="hapusMenuMaster('${m.nama}')">✖</div>
        `;
        container.appendChild(item);
      });
    }
  });
}

function hapusMenuMaster(nama){
    if(confirm(`Hapus menu "${nama}" secara permanen?`)){
        menuData = menuData.filter(m => m.nama !== nama);
        localStorage.setItem("menuRM", JSON.stringify(menuData));
        renderMenu();
    }
}

function tambahItem(nama, harga){
  let ada = pesananData.find(x => x.nama === nama);
  if(ada) { ada.jumlah++; } else { pesananData.push({nama, harga, jumlah:1}); }
  renderPesanan();
  playSfx("suaraKlik");
}

function renderPesanan(){
  const list = document.getElementById("list");
  list.innerHTML = "";
  let total = 0;
  pesananData.forEach((p, i) => {
    total += p.harga * p.jumlah;
    const row = document.createElement("div");
    row.className = "order-row";
    row.innerHTML = `
      <div style="flex:2;"><b>${p.nama}</b><br><small>${p.jumlah} x ${p.harga.toLocaleString()}</small></div>
      <div style="flex:1; text-align:right;">Rp ${(p.harga*p.jumlah).toLocaleString()}</div>
      <div class="hapus-btn" onclick="hapusSatu(${i})">✖</div>
    `;
    list.appendChild(row);
  });
  document.getElementById("total").innerText = total.toLocaleString();
}

function hapusSatu(index){
  pesananData.splice(index, 1);
  renderPesanan();
  playSfx("suaraKlik");
}

function bayar(){
  if(pesananData.length === 0) return alert("Pilih menu dulu!");
  const total = pesananData.reduce((a,b) => a + (b.harga*b.jumlah), 0);
  const riwayat = JSON.parse(localStorage.getItem("riwayatRM")) || [];
  riwayat.push({waktu: new Date().toISOString(), pesanan: [...pesananData], total, kasir: identitas.kasir});
  localStorage.setItem("riwayatRM", JSON.stringify(riwayat));
  pesananData = [];
  renderPesanan();
  playSfx("suaraBayar");
  alert("Nota Berhasil Disimpan!");
}

function loadRekap(){
  const riwayat = JSON.parse(localStorage.getItem("riwayatRM")) || [];
  const div = document.getElementById("rekapDiv");
  div.innerHTML = "";
  let gt = 0;
  riwayat.slice().reverse().forEach((n, i) => {
    gt += n.total;
    let items = n.pesanan.map(p => `<li>${p.nama} x${p.jumlah}</li>`).join("");
    div.innerHTML += `<div style="background:#fff; color:#000; padding:10px; margin-bottom:10px; border-radius:8px; border-left:4px solid #c62828;">
      <b>Nota #${riwayat.length-i}</b> - ${new Date(n.waktu).toLocaleTimeString()}<br>
      <small>Kasir: ${n.kasir}</small>
      <ul style="margin:5px 0;">${items}</ul>
      <b>Total: Rp ${n.total.toLocaleString()}</b>
    </div>`;
  });
  div.innerHTML = `<h3 style="text-align:center;">TOTAL OMZET: Rp ${gt.toLocaleString()}</h3>` + div.innerHTML;
}

function loadRiwayat(){
  const riwayat = JSON.parse(localStorage.getItem("riwayatRM")) || [];
  const div = document.getElementById("riwayat");
  div.innerHTML = "<h4>Laporan Terjual Per Item:</h4>";
  let rekap = {};
  riwayat.forEach(t => { t.pesanan.forEach(p => { rekap[p.nama] = (rekap[p.nama] || 0) + p.jumlah; }); });
  for(let m in rekap) div.innerHTML += `<div>${m}: <b>${rekap[m]} terjual</b></div>`;
}

function resetRiwayat(){ if(confirm("Hapus semua riwayat?")) { localStorage.removeItem("riwayatRM"); loadRiwayat(); loadRekap(); } }

function tambahMenuBaru(){
  const n = document.getElementById("namaBaru").value;
  const h = parseInt(document.getElementById("hargaBaru").value);
  if(!n || !h) return alert("Nama & Harga harus diisi!");
  menuData.push({nama:n, harga:h, kategori:document.getElementById("kategoriBaru").value});
  localStorage.setItem("menuRM", JSON.stringify(menuData));
  renderMenu();
  document.getElementById("namaBaru").value = "";
  document.getElementById("hargaBaru").value = "";
}
</script>
</body>
</html>
