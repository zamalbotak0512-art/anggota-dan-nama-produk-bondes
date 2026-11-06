<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Data Anggota & Produk</title>
<style>
:root{--ink:#172033;--muted:#667085;--line:#e7eaf0;--bg:#f5f7fb;--blue:#315efb;--blue2:#2448cb;--red:#d92d20}
*{box-sizing:border-box}body{margin:0;font-family:Inter,system-ui,Arial,sans-serif;background:var(--bg);color:var(--ink)}
header{background:linear-gradient(120deg,#172554,#315efb);color:white;padding:34px 22px}
.wrap{max-width:1100px;margin:auto;padding:0 18px}header h1{margin:0 0 8px;font-size:clamp(24px,4vw,34px)}header p{margin:0;color:#dbe5ff}
main{padding:24px 0 50px}.stats{display:grid;grid-template-columns:repeat(2,minmax(0,1fr));gap:14px;margin-bottom:18px}
.stat,.panel{background:white;border:1px solid var(--line);border-radius:16px;padding:18px;box-shadow:0 5px 20px #1d293910}
.stat span{display:block;color:var(--muted);font-size:13px}.stat strong{display:block;font-size:28px;margin-top:5px}
.panel{margin-bottom:18px}.panel h2{font-size:18px;margin:0 0 16px}
.formgrid{display:grid;grid-template-columns:repeat(3,minmax(0,1fr));gap:12px}
label{display:block;font-size:13px;font-weight:600;color:#475467;margin-bottom:6px}
input,select{width:100%;padding:11px 12px;border:1px solid #d0d5dd;border-radius:9px;font:inherit;background:white;color:var(--ink)}
input:focus,select:focus{outline:3px solid #315efb22;border-color:var(--blue)}
.actions{display:flex;gap:8px;align-items:end;flex-wrap:wrap}
button{border:0;border-radius:9px;padding:11px 15px;font:inherit;font-weight:650;cursor:pointer}
.primary{background:var(--blue);color:white}.primary:hover{background:var(--blue2)}.secondary{background:#eef2ff;color:#2448cb}.danger{background:#fff0ee;color:var(--red)}.small{padding:7px 10px;font-size:12px}
.toolbar{display:flex;justify-content:space-between;gap:12px;align-items:center;margin-bottom:14px;flex-wrap:wrap}.toolbar input{max-width:320px}
.tablewrap{overflow-x:auto}table{width:100%;border-collapse:collapse;min-width:650px}th,td{text-align:left;padding:12px 10px;border-bottom:1px solid var(--line);font-size:14px}th{color:var(--muted);font-size:12px;text-transform:uppercase;letter-spacing:.04em}tbody tr:last-child td{border-bottom:0}.empty{text-align:center;color:var(--muted);padding:28px}
.pill{display:inline-block;background:#eef2ff;color:#344fbd;border-radius:99px;padding:4px 9px;font-size:12px}
@media(max-width:700px){.formgrid{grid-template-columns:1fr}.stats{grid-template-columns:1fr 1fr}.toolbar input{max-width:none;flex:1}}
</style>
</head>
<body>
<header><div class="wrap"><h1>Data Anggota & Produk</h1><p>Setiap anggota dapat mengisi hingga 5 nama produk.</p></div></header>
<main class="wrap">
<section class="stats">
 <div class="stat"><span>Total Anggota</span><strong id="memberCount">0</strong></div>
 <div class="stat"><span>Total Produk</span><strong id="productCount">0</strong></div>
</section>
<section class="panel">
 <h2 id="formTitle">Tambah anggota dan 5 produk</h2>
 <form id="dataForm">
  <div class="formgrid">
   <div><label for="member">Nama anggota</label><input id="member" placeholder="Contoh: Andi Saputra" required maxlength="80"></div>
  </div>
  <div class="formgrid" style="margin-top:14px">
   <div><label for="product0">Produk 1</label><input id="product0" placeholder="Isi nama produk 1" maxlength="100"></div>
   <div><label for="product1">Produk 2</label><input id="product1" placeholder="Isi nama produk 2" maxlength="100"></div>
   <div><label for="product2">Produk 3</label><input id="product2" placeholder="Isi nama produk 3" maxlength="100"></div>
   <div><label for="product3">Produk 4</label><input id="product3" placeholder="Isi nama produk 4" maxlength="100"></div>
   <div><label for="product4">Produk 5</label><input id="product4" placeholder="Isi nama produk 5" maxlength="100"></div>
  </div>
  <div class="actions" style="margin-top:14px"><button class="primary" type="submit" id="saveBtn">+ Simpan data anggota</button><button class="secondary" type="button" id="cancelBtn" hidden>Batal</button></div>
 </form>
</section>
<section class="panel">
 <div class="toolbar"><h2 style="margin:0">Daftar anggota & produk</h2><div class="actions"><input id="search" type="search" placeholder="Cari anggota atau produk..."><button class="secondary" type="button" id="exportCsv">Ekspor CSV</button><button class="secondary" type="button" id="exportTxt">Ekspor TXT</button></div></div>
 <div class="tablewrap"><table><thead><tr><th>No.</th><th>Nama anggota</th><th>5 pilihan nama produk</th><th>Aksi</th></tr></thead><tbody id="rows"></tbody></table><div class="empty" id="empty">Belum ada data. Tambahkan nama anggota dan isi daftar produknya di atas.</div></div>
</section>
</main>
<script>
const KEY='data-anggota-produk-v1';
let records=JSON.parse(localStorage.getItem(KEY)||'[]'), editingId=null;
const $=id=>document.getElementById(id);
function productsFromRecord(r){
 if(Array.isArray(r.products)) return r.products;
 return r.product ? [r.product] : [];
}
function persist(){localStorage.setItem(KEY,JSON.stringify(records));render()}
function render(){
 const q=$('search').value.trim().toLowerCase();
 const filtered=records.filter(r=>r.member.toLowerCase().includes(q)||productsFromRecord(r).some(p=>p.toLowerCase().includes(q)));
 $('memberCount').textContent=records.length;
 $('productCount').textContent=new Set(records.flatMap(productsFromRecord).map(p=>p.toLowerCase())).size;
 $('rows').innerHTML=filtered.map((r,i)=>{
  const products=productsFromRecord(r);
  const productList=Array.from({length:5},(_,n)=>products[n]?`<span class="pill" style="margin:2px">${n+1}. ${escapeHtml(products[n])}</span>`:`<span style="color:#98a2b3;font-size:12px;margin:2px">${n+1}. —</span>`).join('');
  return `<tr><td>${i+1}</td><td>${escapeHtml(r.member)}</td><td style="min-width:300px">${productList}</td><td><button class="secondary small" onclick="editRecord('${r.id}')">Edit</button> <button class="danger small" onclick="deleteRecord('${r.id}')">Hapus</button></td></tr>`;
 }).join('');
 $('empty').hidden=filtered.length>0;
}
function escapeHtml(s){return String(s).replace(/[&<>"']/g,c=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]))}
$('dataForm').addEventListener('submit',e=>{
 e.preventDefault();
 const member=$('member').value.trim();
 const products=Array.from({length:5},(_,i)=>$('product'+i).value.trim());
 if(!member)return;
 if(editingId){records=records.map(r=>r.id===editingId?{id:r.id,member,products}:r)}
 else records.push({id:crypto.randomUUID(),member,products});
 resetForm();persist();
});
function editRecord(id){
 const r=records.find(x=>x.id===id);if(!r)return;
 editingId=id;$('member').value=r.member;
 const products=productsFromRecord(r);
 for(let i=0;i<5;i++)$('product'+i).value=products[i]||'';
 $('formTitle').textContent='Edit data anggota dan produk';
 $('saveBtn').textContent='Simpan perubahan';$('cancelBtn').hidden=false;$('member').focus();
}
function deleteRecord(id){if(confirm('Hapus data anggota dan semua produknya?')){records=records.filter(r=>r.id!==id);if(editingId===id)resetForm();persist()}}
function resetForm(){editingId=null;$('dataForm').reset();$('formTitle').textContent='Tambah anggota dan 5 produk';$('saveBtn').textContent='+ Simpan data anggota';$('cancelBtn').hidden=true}
$('cancelBtn').addEventListener('click',resetForm);$('search').addEventListener('input',render);
function exportRows(){
 const q=$('search').value.trim().toLowerCase();
 return records.filter(r=>r.member.toLowerCase().includes(q)||productsFromRecord(r).some(p=>p.toLowerCase().includes(q)));
}
function downloadFile(content,filename,type){
 const blob=new Blob([content],{type});
 const url=URL.createObjectURL(blob);const a=document.createElement('a');
 a.href=url;a.download=filename;document.body.appendChild(a);a.click();a.remove();URL.revokeObjectURL(url);
}
function csvCell(value){return '"'+String(value??'').replace(/"/g,'""')+'"'}
$('exportCsv').addEventListener('click',()=>{
 const data=exportRows();
 const header=['Nama Anggota',...Array.from({length:5},(_,i)=>'Produk '+(i+1))];
 const lines=[header,...data.map(r=>[r.member,...Array.from({length:5},(_,i)=>productsFromRecord(r)[i]||'')])];
 const csv='\uFEFF'+lines.map(row=>row.map(csvCell).join(',')).join('\r\n');
 downloadFile(csv,'daftar_anggota_dan_produk.csv','text/csv;charset=utf-8;');
});
$('exportTxt').addEventListener('click',()=>{
 const data=exportRows();
 const body=data.map((r,i)=>[
  `${i+1}. Nama Anggota: ${r.member}`,
  ...Array.from({length:5},(_,n)=>`   Produk ${n+1}: ${productsFromRecord(r)[n]||'-'}`)
 ].join('\n')).join('\n\n');
 downloadFile(body||'Belum ada data anggota.', 'daftar_anggota_dan_produk.txt','text/plain;charset=utf-8;');
});
render();
</script>
</body></html>
