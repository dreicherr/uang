# uang
Catatan kuangan saya dan partner periode Juli-Desember 2026
<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<title>Kas Harian</title>
<style>
  :root{
    --bg:#EEF1EF; --card:#FFFFFF; --ink:#1B2B29; --ink-soft:#5B6B68;
    --teal:#2F6F63; --teal-dark:#1F4E45; --gold:#C99A2E; --red:#B84C3C;
    --line:#D8DFDC; --green-bg:#E4EFE9; --red-bg:#F7E7E3; --gold-bg:#FBF1DC;
  }
  *{box-sizing:border-box;}
  body{
    margin:0; background:var(--bg); color:var(--ink);
    font-family:'Segoe UI',-apple-system,BlinkMacSystemFont,sans-serif;
    padding:20px;
  }
  .wrap{max-width:900px;margin:0 auto;}
  h1{
    font-family:Georgia,'Iowan Old Style',serif; font-size:28px; margin:0 0 2px 0;
    color:var(--teal-dark); letter-spacing:.2px;
  }
  .sub{color:var(--ink-soft); font-size:13px; margin-bottom:18px;}
  .mono{font-family:'SFMono-Regular',Menlo,Consolas,monospace;}
  .tabs{display:flex; gap:6px; margin-bottom:16px; flex-wrap:wrap;}
  .tab{
    padding:8px 14px; border-radius:8px; cursor:pointer; font-size:13px;
    background:var(--card); border:1px solid var(--line); color:var(--ink-soft);
  }
  .tab.active{background:var(--teal-dark); color:#fff; border-color:var(--teal-dark);}
  .card{
    background:var(--card); border-radius:14px; padding:20px; margin-bottom:16px;
    border:1px solid var(--line);
  }
  .receipt{
    background:var(--card); border:1px dashed var(--teal); border-radius:10px;
    padding:18px 22px; text-align:center; margin-bottom:16px;
  }
  .receipt .label{font-size:12px; letter-spacing:1.5px; color:var(--ink-soft); text-transform:uppercase;}
  .receipt .amount{font-family:'SFMono-Regular',Menlo,monospace; font-size:34px; font-weight:700; color:var(--teal-dark); margin:4px 0;}
  .grid2{display:grid; grid-template-columns:1fr 1fr; gap:14px;}
  .grid3{display:grid; grid-template-columns:1fr 1fr 1fr; gap:10px;}
  @media(max-width:640px){.grid2,.grid3{grid-template-columns:1fr;}}
  label{font-size:12px; color:var(--ink-soft); display:block; margin-bottom:3px;}
  input, select{
    width:100%; padding:8px 10px; border-radius:7px; border:1px solid var(--line);
    font-size:14px; font-family:inherit; margin-bottom:10px; background:#fcfcfb;
  }
  button{
    background:var(--teal-dark); color:#fff; border:none; padding:10px 18px;
    border-radius:8px; cursor:pointer; font-size:14px; font-weight:600;
  }
  button.ghost{background:transparent; color:var(--red); border:1px solid var(--red); padding:5px 10px; font-size:12px; font-weight:400;}
  button.small{padding:5px 10px; font-size:12px;}
  table{width:100%; border-collapse:collapse; font-size:13px;}
  th,td{padding:7px 6px; text-align:left; border-bottom:1px solid var(--line);}
  th{color:var(--ink-soft); font-weight:600; font-size:11px; text-transform:uppercase;}
  td.num{font-family:'SFMono-Regular',Menlo,monospace; text-align:right;}
  .section-title{font-family:Georgia,serif; font-size:16px; color:var(--teal-dark); margin:0 0 12px 0; border-bottom:2px solid var(--teal); padding-bottom:6px;}
  .bar-bg{background:#E2E6E4; border-radius:6px; height:10px; overflow:hidden; margin:6px 0 2px 0;}
  .bar-fill{background:var(--teal); height:100%; border-radius:6px;}
  .bar-fill.over{background:var(--gold);}
  .pill{display:inline-block; padding:2px 9px; border-radius:20px; font-size:11px; font-weight:600;}
  .pill.ok{background:var(--green-bg); color:var(--teal-dark);}
  .pill.warn{background:var(--gold-bg); color:#8a6d1c;}
  .pill.bad{background:var(--red-bg); color:var(--red);}
  .month-select{display:flex; gap:8px; align-items:center; margin-bottom:14px;}
  .month-select select{width:auto; margin:0;}
  .note{font-size:12px; color:var(--ink-soft); font-style:italic; margin-top:6px;}
  .stat-row{display:flex; justify-content:space-between; font-size:13px; padding:5px 0;}
  .stat-row b{font-family:'SFMono-Regular',Menlo,monospace;}
  .empty{color:var(--ink-soft); font-size:13px; padding:14px 0; text-align:center;}
</style>
</head>
<body>
<div class="wrap">
  <h1>📒 Kas Harian</h1>
  <div class="sub">Pencatatan otomatis · Juli–November 2026</div>
  <div class="tabs">
    <div class="tab active" data-tab="dashboard">Dashboard</div>
    <div class="tab" data-tab="input">Input Harian</div>
    <div class="tab" data-tab="hutang">Hutang Teman</div>
    <div class="tab" data-tab="paylater">PayLater &amp; Gadai</div>
    <div class="tab" data-tab="target">Target Bulanan</div>
  </div>

  <div id="dashboard"></div>
  <div id="input" style="display:none"></div>
  <div id="hutang" style="display:none"></div>
  <div id="paylater" style="display:none"></div>
  <div id="target" style="display:none"></div>
</div>

<script>
const CUR = n => 'Rp' + Math.round(n).toLocaleString('id-ID');
const MONTHS = ['Juli','Agustus','September','Oktober','November'];
const MONTH_KEY = m => ['2026-07','2026-08','2026-09','2026-10','2026-11'][MONTHS.indexOf(m)];

let state = {
  saldoAwal: 19000,
  entries: [],
  friendDebts: [
    {name:"Divka",awal:300000,bayar:0},{name:"Edwin",awal:300000,bayar:0},
    {name:"Ka Selpi BME",awal:250000,bayar:0},{name:"Ulul",awal:225000,bayar:0},
    {name:"Lek Sruni",awal:200000,bayar:0},{name:"Bludi",awal:100000,bayar:100000},
    {name:"Kantri",awal:100000,bayar:0},{name:"Ican",awal:60000,bayar:30000},
    {name:"Alan",awal:50000,bayar:0},{name:"Zaki",awal:50000,bayar:0},
    {name:"Cunda",awal:50000,bayar:0},{name:"Yosua",awal:50000,bayar:30000},
  ],
  paylater: [
    {label:"Juni",jumlah:638000,due:"1 Jul",status:"Lunas"},
    {label:"Juli",jumlah:640000,due:"1 Agu",status:"Belum"},
    {label:"Agustus",jumlah:563000,due:"1 Sep",status:"Belum"},
    {label:"September",jumlah:563000,due:"1 Okt",status:"Belum"},
    {label:"Oktober",jumlah:563000,due:"1 Nov",status:"Belum (rencana prepay KIP)"},
    {label:"November",jumlah:539000,due:"1 Des",status:"Belum (rencana prepay KIP)"},
    {label:"Desember",jumlah:539000,due:"1 Jan",status:"Belum (rencana prepay KIP)"},
    {label:"Januari",jumlah:297000,due:"1 Feb",status:"Belum (rencana prepay KIP)"},
  ],
  gadai: {pokok:1725000, perpanjangan:190000, status:"Aktif (diperpanjang)"},
  targets: {
    Juli:      {kosPartner:500000, kosDia:700000, kuota:130000, gadai:190000, paylater:640000, hutang:0, makan:50000, kebutuhan:150000, motor:100000, bensin:100000, reserve:0, ortu:750000},
    Agustus:   {kosPartner:500000, kosDia:700000, kuota:130000, gadai:190000, paylater:563000, hutang:0, makan:150000, kebutuhan:180000, motor:100000, bensin:120000, reserve:0, ortu:750000},
    September: {kosPartner:500000, kosDia:700000, kuota:130000, gadai:190000, paylater:563000, hutang:0, makan:200000, kebutuhan:200000, motor:100000, bensin:150000, reserve:0, ortu:1505000},
    Oktober:   {kosPartner:500000, kosDia:0,      kuota:130000, gadai:0,      paylater:0,      hutang:0, makan:200000, kebutuhan:200000, motor:100000, bensin:150000, reserve:1920000, ortu:1505000},
    November:  {kosPartner:500000, kosDia:0,      kuota:130000, gadai:0,      paylater:0,      hutang:0, makan:200000, kebutuhan:200000, motor:100000, bensin:150000, reserve:1918000, ortu:1505000},
  }
};

async function loadState(){
  try{
    const s = await window.storage.get('kas-state');
    if(s && s.value) state = JSON.parse(s.value);
  }catch(e){ /* belum ada data tersimpan, pakai default */ }
}
async function saveState(){
  try{ await window.storage.set('kas-state', JSON.stringify(state)); }
  catch(e){ console.error('Gagal menyimpan', e); }
}

function todayStr(){
  const d = new Date();
  return d.toISOString().slice(0,10);
}
function hariFromDate(dstr){
  const days=['Minggu','Senin','Selasa','Rabu','Kamis','Jumat','Sabtu'];
  return days[new Date(dstr).getDay()];
}
function monthLabelFromDate(dstr){
  const m = dstr.slice(0,7);
  const idx = ['2026-07','2026-08','2026-09','2026-10','2026-11'].indexOf(m);
  return idx>=0 ? MONTHS[idx] : null;
}

function runningSaldo(){
  let s = state.saldoAwal;
  const list = [...state.entries].sort((a,b)=>a.tanggal.localeCompare(b.tanggal));
  for(const e of list){
    const masuk = e.ortu+e.spf+e.lain;
    const keluar = e.kos+e.kuota+e.gadai+e.paylater+e.hutang+e.makan+e.kebutuhan+e.motor+e.bensin;
    s += masuk-keluar;
  }
  return s;
}

function sisaHutangTeman(){
  return state.friendDebts.reduce((a,d)=>a+(d.awal-d.bayar),0);
}
function sisaPaylater(){
  return state.paylater.filter(p=>p.status!=='Lunas').reduce((a,p)=>a+p.jumlah,0);
}

function weekOfMonth(dstr){
  const d = new Date(dstr);
  return Math.ceil(d.getDate()/7);
}

function computeTarget(month){
  const t = state.targets[month];
  const total = t.kosPartner+t.kosDia+t.kuota+t.gadai+t.paylater+t.hutang+t.makan+t.kebutuhan+t.motor+t.bensin+t.reserve;
  const kebutuhanSpF = Math.max(0, total - t.ortu);
  const weekly = kebutuhanSpF/4.33;
  const daily = weekly/5;
  return {total, kebutuhanSpF, weekly, daily};
}

function currentMonthEntries(month){
  const key = MONTH_KEY(month);
  return state.entries.filter(e=>e.tanggal.startsWith(key));
}

// ---------------- RENDER: DASHBOARD ----------------
function renderDashboard(){
  const el = document.getElementById('dashboard');
  const saldo = runningSaldo();
  const today = todayStr();
  let curMonth = monthLabelFromDate(today) || 'Juli';
  const entriesThisMonth = currentMonthEntries(curMonth);
  const spfThisMonth = entriesThisMonth.reduce((a,e)=>a+e.spf,0);
  const targ = computeTarget(curMonth);
  const spfThisWeek = state.entries.filter(e=>{
    const d = new Date(e.tanggal); const now = new Date();
    const diff = (now-d)/(1000*3600*24);
    return diff>=0 && diff<7;
  }).reduce((a,e)=>a+e.spf,0);

  const debtT = sisaHutangTeman();
  const debtP = sisaPaylater();
  const pct = Math.min(100, Math.round(saldo/4000000*100));

  el.innerHTML = `
    <div class="receipt">
      <div class="label">Saldo Saat Ini</div>
      <div class="amount mono">${CUR(saldo)}</div>
      <div class="note">Target akhir November: Rp4.000.000 (setelah semua hutang lunas)</div>
      <div class="bar-bg"><div class="bar-fill ${pct>=100?'over':''}" style="width:${pct}%"></div></div>
      <div class="note">${pct}% menuju target cadangan</div>
    </div>
    <div class="grid2">
      <div class="card">
        <div class="section-title">ShopeeFood — ${curMonth}</div>
        <div class="stat-row"><span>Target mingguan</span><b>${CUR(targ.weekly)}</b></div>
        <div class="stat-row"><span>Target harian (5 hr/mgg)</span><b>${CUR(targ.daily)}</b></div>
        <div class="stat-row"><span>Realisasi minggu ini</span><b class="mono">${CUR(spfThisWeek)}</b></div>
        <div class="stat-row"><span>Realisasi bulan ini</span><b class="mono">${CUR(spfThisMonth)}</b></div>
      </div>
      <div class="card">
        <div class="section-title">Sisa Kewajiban</div>
        <div class="stat-row"><span>Hutang teman</span><b class="mono">${CUR(debtT)}</b></div>
        <div class="stat-row"><span>Shopee PayLater</span><b class="mono">${CUR(debtP)}</b></div>
        <div class="stat-row"><span>Gadai laptop</span><b class="mono">${state.gadai.status==='Ditebus'?'Rp0 (lunas)':CUR(state.gadai.pokok)}</b></div>
        <div class="stat-row"><span><b>Total kewajiban</b></span><b class="mono">${CUR(debtT+debtP+(state.gadai.status==='Ditebus'?0:state.gadai.pokok))}</b></div>
      </div>
    </div>
    <div class="card">
      <div class="section-title">Catatan Terakhir</div>
      ${renderRecentTable()}
    </div>
  `;
}

function renderRecentTable(){
  const list = [...state.entries].sort((a,b)=>b.tanggal.localeCompare(a.tanggal)).slice(0,7);
  if(list.length===0) return '<div class="empty">Belum ada catatan. Isi di tab "Input Harian".</div>';
  let rows = list.map(e=>{
    const masuk = e.ortu+e.spf+e.lain;
    const keluar = e.kos+e.kuota+e.gadai+e.paylater+e.hutang+e.makan+e.kebutuhan+e.motor+e.bensin;
    return `<tr><td>${e.tanggal}</td><td>${e.hari}</td><td class="num">${CUR(masuk)}</td><td class="num">${CUR(keluar)}</td>
      <td><button class="ghost small" onclick="deleteEntry('${e.id}')">Hapus</button></td></tr>`;
  }).join('');
  return `<table><tr><th>Tanggal</th><th>Hari</th><th>Masuk</th><th>Keluar</th><th></th></tr>${rows}</table>`;
}

async function deleteEntry(id){
  state.entries = state.entries.filter(e=>e.id!==id);
  await saveState();
  renderAll();
}

// ---------------- RENDER: INPUT HARIAN ----------------
function renderInput(){
  const el = document.getElementById('input');
  el.innerHTML = `
    <div class="card">
      <div class="section-title">Tambah Catatan Harian</div>
      <div class="grid3">
        <div><label>Tanggal</label><input type="date" id="f_tgl" value="${todayStr()}"></div>
        <div><label>Ortu (masuk)</label><input type="number" id="f_ortu" value="0"></div>
        <div><label>SpF Bersih (masuk)</label><input type="number" id="f_spf" value="0"></div>
        <div><label>Lain-lain (masuk)</label><input type="number" id="f_lain" value="0"></div>
        <div><label>Kos (keluar)</label><input type="number" id="f_kos" value="0"></div>
        <div><label>Kuota (keluar)</label><input type="number" id="f_kuota" value="0"></div>
        <div><label>Gadai (keluar)</label><input type="number" id="f_gadai" value="0"></div>
        <div><label>PayLater (keluar)</label><input type="number" id="f_paylater" value="0"></div>
        <div><label>Hutang Teman (keluar)</label><input type="number" id="f_hutang" value="0"></div>
        <div><label>Makan (keluar)</label><input type="number" id="f_makan" value="0"></div>
        <div><label>Kebutuhan (keluar)</label><input type="number" id="f_kebutuhan" value="0"></div>
        <div><label>Motor (keluar)</label><input type="number" id="f_motor" value="0"></div>
        <div><label>Bensin (keluar)</label><input type="number" id="f_bensin" value="0"></div>
      </div>
      <button onclick="submitEntry()">Simpan Catatan</button>
      <div id="save_msg" class="note"></div>
    </div>
    <div class="card">
      <div class="section-title">Semua Catatan</div>
      <div style="max-height:340px; overflow:auto;">${renderFullTable()}</div>
    </div>
  `;
}

function renderFullTable(){
  const list = [...state.entries].sort((a,b)=>a.tanggal.localeCompare(b.tanggal));
  if(list.length===0) return '<div class="empty">Belum ada catatan.</div>';
  let saldo = state.saldoAwal;
  let rows = list.map(e=>{
    const masuk = e.ortu+e.spf+e.lain;
    const keluar = e.kos+e.kuota+e.gadai+e.paylater+e.hutang+e.makan+e.kebutuhan+e.motor+e.bensin;
    saldo += masuk-keluar;
    return `<tr><td>${e.tanggal}</td><td>${e.hari}</td><td class="num">${CUR(masuk)}</td><td class="num">${CUR(keluar)}</td><td class="num">${CUR(saldo)}</td></tr>`;
  }).join('');
  return `<table><tr><th>Tanggal</th><th>Hari</th><th>Masuk</th><th>Keluar</th><th>Saldo</th></tr>${rows}</table>`;
}

async function submitEntry(){
  const val = id => Number(document.getElementById(id).value)||0;
  const tgl = document.getElementById('f_tgl').value;
  if(!tgl){ document.getElementById('save_msg').innerText='Isi tanggal dulu.'; return; }
  const entry = {
    id: 'e'+Date.now(),
    tanggal: tgl, hari: hariFromDate(tgl),
    ortu: val('f_ortu'), spf: val('f_spf'), lain: val('f_lain'),
    kos: val('f_kos'), kuota: val('f_kuota'), gadai: val('f_gadai'),
    paylater: val('f_paylater'), hutang: val('f_hutang'), makan: val('f_makan'),
    kebutuhan: val('f_kebutuhan'), motor: val('f_motor'), bensin: val('f_bensin'),
  };
  state.entries.push(entry);
  await saveState();
  document.getElementById('save_msg').innerText = 'Tersimpan ✓';
  renderAll();
  renderInput();
}

// ---------------- RENDER: HUTANG TEMAN ----------------
function renderHutang(){
  const el = document.getElementById('hutang');
  let rows = state.friendDebts.map((d,i)=>{
    const sisa = d.awal-d.bayar;
    return `<tr>
      <td>${d.name}</td>
      <td class="num">${CUR(d.awal)}</td>
      <td class="num"><input type="number" style="width:90px;margin:0" value="${d.bayar}" onchange="updateBayar(${i}, this.value)"></td>
      <td class="num">${CUR(sisa)}</td>
      <td>${sisa<=0?'<span class="pill ok">LUNAS</span>':'<span class="pill warn">Sisa</span>'}</td>
    </tr>`;
  }).join('');
  el.innerHTML = `
    <div class="card">
      <div class="section-title">Hutang Teman</div>
      <table><tr><th>Nama</th><th>Awal</th><th>Sudah Dibayar</th><th>Sisa</th><th>Status</th></tr>${rows}</table>
      <div class="stat-row" style="margin-top:10px;border-top:2px solid var(--line);padding-top:10px;">
        <span><b>Total Sisa</b></span><b class="mono">${CUR(sisaHutangTeman())}</b>
      </div>
      <div class="note">Rencana: seluruh sisa dilunasi Oktober memakai dana KIP Kuliah.</div>
    </div>
  `;
}
async function updateBayar(i, v){
  state.friendDebts[i].bayar = Number(v)||0;
  await saveState();
  renderAll();
}

// ---------------- RENDER: PAYLATER & GADAI ----------------
function renderPaylater(){
  const el = document.getElementById('paylater');
  let rows = state.paylater.map((p,i)=>`
    <tr><td>${p.label}</td><td class="num">${CUR(p.jumlah)}</td><td>${p.due}</td>
    <td><select onchange="updatePLStatus(${i}, this.value)">
      <option ${p.status==='Belum'?'selected':''}>Belum</option>
      <option ${p.status==='Lunas'?'selected':''}>Lunas</option>
      <option value="Belum (rencana prepay KIP)" ${p.status.includes('KIP')?'selected':''}>Belum (rencana prepay KIP)</option>
    </select></td></tr>`).join('');
  el.innerHTML = `
    <div class="card">
      <div class="section-title">Shopee PayLater</div>
      <table><tr><th>Tagihan</th><th>Jumlah</th><th>Jatuh Tempo</th><th>Status</th></tr>${rows}</table>
      <div class="stat-row" style="margin-top:10px;border-top:2px solid var(--line);padding-top:10px;">
        <span><b>Total Belum Lunas</b></span><b class="mono">${CUR(sisaPaylater())}</b>
      </div>
    </div>
    <div class="card">
      <div class="section-title">Gadai Laptop</div>
      <div class="stat-row"><span>Pokok Gadai</span><b class="mono">${CUR(state.gadai.pokok)}</b></div>
      <div class="stat-row"><span>Biaya Perpanjangan/Bulan</span><b class="mono">${CUR(state.gadai.perpanjangan)}</b></div>
      <div class="stat-row"><span>Status</span><b>${state.gadai.status}</b></div>
      ${state.gadai.status!=='Ditebus' ? '<button onclick="redeemGadai()">Tandai Sudah Ditebus (Oktober)</button>' : '<span class="pill ok">LUNAS DITEBUS</span>'}
    </div>
  `;
}
async function updatePLStatus(i,v){ state.paylater[i].status=v; await saveState(); renderAll(); }
async function redeemGadai(){ state.gadai.status='Ditebus'; await saveState(); renderAll(); }

// ---------------- RENDER: TARGET BULANAN ----------------
function renderTarget(){
  const el = document.getElementById('target');
  let cards = MONTHS.map(m=>{
    const t = computeTarget(m);
    const a = state.targets[m];
    return `<div class="card">
      <div class="section-title">${m}</div>
      <div class="grid3">
        <div><label>Kos Partner</label><input type="number" value="${a.kosPartner}" onchange="updateTarget('${m}','kosPartner',this.value)"></div>
        <div><label>Kos Anda (sinking/KIP)</label><input type="number" value="${a.kosDia}" onchange="updateTarget('${m}','kosDia',this.value)"></div>
        <div><label>Kuota</label><input type="number" value="${a.kuota}" onchange="updateTarget('${m}','kuota',this.value)"></div>
        <div><label>Gadai</label><input type="number" value="${a.gadai}" onchange="updateTarget('${m}','gadai',this.value)"></div>
        <div><label>PayLater</label><input type="number" value="${a.paylater}" onchange="updateTarget('${m}','paylater',this.value)"></div>
        <div><label>Hutang Teman</label><input type="number" value="${a.hutang}" onchange="updateTarget('${m}','hutang',this.value)"></div>
        <div><label>Makan</label><input type="number" value="${a.makan}" onchange="updateTarget('${m}','makan',this.value)"></div>
        <div><label>Kebutuhan</label><input type="number" value="${a.kebutuhan}" onchange="updateTarget('${m}','kebutuhan',this.value)"></div>
        <div><label>Motor</label><input type="number" value="${a.motor}" onchange="updateTarget('${m}','motor',this.value)"></div>
        <div><label>Bensin</label><input type="number" value="${a.bensin}" onchange="updateTarget('${m}','bensin',this.value)"></div>
        <div><label>Target Tabungan Cadangan</label><input type="number" value="${a.reserve}" onchange="updateTarget('${m}','reserve',this.value)"></div>
        <div><label>Estimasi Ortu</label><input type="number" value="${a.ortu}" onchange="updateTarget('${m}','ortu',this.value)"></div>
      </div>
      <div class="stat-row"><span>Total Kebutuhan</span><b class="mono">${CUR(t.total)}</b></div>
      <div class="stat-row"><span>Perlu dari ShopeeFood</span><b class="mono">${CUR(t.kebutuhanSpF)}</b></div>
      <div class="stat-row"><span>Target Mingguan</span><b class="mono">${CUR(t.weekly)}</b></div>
      <div class="stat-row"><span><b>Target Harian (5 hr/mgg)</b></span><b class="mono">${CUR(t.daily)}</b></div>
    </div>`;
  }).join('');
  el.innerHTML = cards;
}
async function updateTarget(m,key,v){
  state.targets[m][key] = Number(v)||0;
  await saveState();
  renderAll();
}

// ---------------- TABS ----------------
document.querySelectorAll('.tab').forEach(tab=>{
  tab.addEventListener('click', ()=>{
    document.querySelectorAll('.tab').forEach(t=>t.classList.remove('active'));
    tab.classList.add('active');
    ['dashboard','input','hutang','paylater','target'].forEach(id=>{
      document.getElementById(id).style.display = (id===tab.dataset.tab)?'block':'none';
    });
  });
});

function renderAll(){
  renderDashboard();
  renderHutang();
  renderPaylater();
  renderTarget();
}

(async function init(){
  await loadState();
  renderAll();
  renderInput();
})();
</script>
</body>
</html>
