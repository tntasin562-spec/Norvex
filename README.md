# Norvex
WEAR YOUR IDIENTITY
from pathlib import Path
import zipfile, textwrap

base = Path("/mnt/data/norvez_app")
base.mkdir(exist_ok=True)

html = r'''<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>NØRVEZ — Clothing Store</title>
<style>
:root{--bg:#0b0b0c;--card:#151518;--text:#f7f7f7;--muted:#a7a7ad;--line:#29292e;--accent:#d9d9dd}
*{box-sizing:border-box}body{margin:0;font-family:Arial,Helvetica,sans-serif;background:var(--bg);color:var(--text)}
header{position:sticky;top:0;z-index:5;background:#0b0b0ceF;border-bottom:1px solid var(--line);backdrop-filter:blur(10px)}
.nav{max-width:1200px;margin:auto;padding:18px 22px;display:flex;align-items:center;justify-content:space-between}.logo{font-size:24px;font-weight:900;letter-spacing:5px}.nav button,.btn{border:1px solid var(--line);background:var(--card);color:var(--text);padding:10px 15px;border-radius:10px;cursor:pointer}
.hero{max-width:1200px;margin:24px auto;padding:55px 22px;border:1px solid var(--line);border-radius:22px;background:linear-gradient(120deg,#18181c,#0d0d0f);min-height:270px;display:flex;align-items:end}
.hero h1{font-size:clamp(38px,7vw,78px);margin:0 0 8px}.hero p{color:var(--muted);font-size:17px}
main{max-width:1200px;margin:auto;padding:0 22px 60px}.cats{display:flex;gap:10px;overflow:auto;padding:8px 0 24px}.cats button{white-space:nowrap}
.grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(220px,1fr));gap:18px}.card{background:var(--card);border:1px solid var(--line);border-radius:17px;overflow:hidden}.pic{height:245px;background:#222;display:flex;align-items:center;justify-content:center;color:#777;font-size:14px}.pic img{width:100%;height:100%;object-fit:cover}.info{padding:15px}.info h3{margin:0 0 7px}.meta{color:var(--muted);font-size:13px}.price{font-weight:800;font-size:18px;margin:10px 0}.actions{display:flex;gap:8px}.actions button{flex:1}
.admin{display:none;max-width:1200px;margin:auto;padding:25px 22px 70px}.admin.active{display:block}.admin h2{font-size:30px}.form{display:grid;grid-template-columns:repeat(auto-fit,minmax(210px,1fr));gap:12px;background:var(--card);padding:18px;border:1px solid var(--line);border-radius:17px}.form input,.form select,.form textarea{width:100%;padding:12px;border-radius:10px;border:1px solid var(--line);background:#0e0e10;color:white}.form textarea{min-height:90px}.wide{grid-column:1/-1}.table{margin-top:18px;overflow:auto}.row{min-width:720px;display:grid;grid-template-columns:2fr 1fr 1fr 1fr 100px;gap:10px;padding:13px;border-bottom:1px solid var(--line);align-items:center}.small{font-size:12px;color:var(--muted)}
footer{text-align:center;color:#777;border-top:1px solid var(--line);padding:25px}
.modal{display:none;position:fixed;inset:0;background:#000b;z-index:20;align-items:center;justify-content:center;padding:20px}.modal.show{display:flex}.box{max-width:520px;width:100%;background:#151518;border:1px solid var(--line);border-radius:18px;padding:20px}.box img{width:100%;max-height:330px;object-fit:cover;border-radius:12px}.close{float:right;background:none;border:0;color:white;font-size:25px}
</style>
</head>
<body>
<header><div class="nav"><div class="logo" id="brand">NØRVEZ</div><div><button onclick="toggleAdmin()">Admin Panel</button></div></div></header>

<section class="hero"><div><div class="small">PREMIUM CLOTHING BRAND</div><h1 id="heroTitle">WEAR YOUR IDENTITY.</h1><p id="heroText">Men • Women • T-Shirts • Pants • Saree • New Collection</p></div></section>

<main id="shop">
<h2>Shop Collection</h2>
<div class="cats" id="cats"></div>
<div class="grid" id="products"></div>
</main>

<section class="admin" id="admin">
<h2>NØRVEZ Admin Dashboard</h2>
<p class="small">এই demo-তে তুমি product add, edit, delete, price, stock, category এবং brand settings পরিবর্তন করতে পারবে।</p>
<div class="form">
<input id="pname" placeholder="Product name">
<input id="price" type="number" placeholder="Price (৳)">
<select id="category"><option>T-Shirts</option><option>Pants</option><option>Men</option><option>Women</option><option>Saree</option><option>New Arrivals</option></select>
<input id="stock" type="number" placeholder="Stock">
<input id="image" class="wide" placeholder="Image URL (optional)">
<textarea id="desc" class="wide" placeholder="Product description"></textarea>
<button class="btn wide" onclick="addProduct()">+ Add Product</button>
</div>
<h3>Products</h3><div class="table" id="adminList"></div>
<h3>Store Design</h3>
<div class="form">
<input id="brandInput" placeholder="Brand name">
<input id="heroInput" class="wide" placeholder="Hero headline">
<input id="heroTextInput" class="wide" placeholder="Hero subtext">
<button class="btn wide" onclick="saveSettings()">Save Store Settings</button>
</div>
</section>

<footer>© 2026 NØRVEZ — Premium Clothing</footer>

<div class="modal" id="modal"><div class="box"><button class="close" onclick="closeModal()">×</button><div id="modalBody"></div></div></div>

<script>
const defaultProducts=[
{id:1,name:"NØRVEZ Essential Tee",price:850,category:"T-Shirts",stock:25,image:"",desc:"Premium everyday T-shirt."},
{id:2,name:"NØRVEZ Classic Pants",price:1450,category:"Pants",stock:18,image:"",desc:"Clean modern fit."},
{id:3,name:"NØRVEZ Women's Collection",price:1250,category:"Women",stock:14,image:"",desc:"Modern women's fashion."},
{id:4,name:"NØRVEZ Premium Saree",price:2450,category:"Saree",stock:10,image:"",desc:"Elegant premium saree collection."}
];
let products=JSON.parse(localStorage.getItem("norvez_products")||"null")||defaultProducts;
let settings=JSON.parse(localStorage.getItem("norvez_settings")||"null")||{brand:"NØRVEZ",hero:"WEAR YOUR IDENTITY.",text:"Men • Women • T-Shirts • Pants • Saree • New Collection"};
let filter="All";

function money(n){return "৳"+Number(n).toLocaleString("en-BD")}
function save(){localStorage.setItem("norvez_products",JSON.stringify(products))}
function renderCats(){
 const cats=["All",...new Set(products.map(p=>p.category))];
 document.getElementById("cats").innerHTML=cats.map(c=>`<button class="btn" onclick="setFilter('${c.replaceAll("'","\\'")}')">${c}</button>`).join("");
}
function setFilter(c){filter=c;render()}
function render(){
 renderCats();
 const list=filter==="All"?products:products.filter(p=>p.category===filter);
 document.getElementById("products").innerHTML=list.map(p=>`
 <article class="card">
 <div class="pic">${p.image?`<img src="${escapeHtml(p.image)}" alt="">`:"NØRVEZ"}</div>
 <div class="info"><div class="meta">${escapeHtml(p.category)} • Stock ${p.stock}</div><h3>${escapeHtml(p.name)}</h3>
 <div class="price">${money(p.price)}</div><div class="actions"><button class="btn" onclick="details(${p.id})">View</button><button class="btn" onclick="alert('Cart feature is ready for the next version.')">Add to Cart</button></div></div></article>`).join("");
 renderAdmin();
}
function renderAdmin(){
 document.getElementById("adminList").innerHTML=`<div class="row small"><b>PRODUCT</b><b>PRICE</b><b>CATEGORY</b><b>STOCK</b><b>ACTION</b></div>`+
 products.map(p=>`<div class="row"><div><b>${escapeHtml(p.name)}</b><div class="small">${escapeHtml(p.desc||"")}</div></div><div>${money(p.price)}</div><div>${escapeHtml(p.category)}</div><div>${p.stock}</div><div><button class="btn" onclick="removeProduct(${p.id})">Delete</button></div></div>`).join("");
}
function addProduct(){
 const name=document.getElementById("pname").value.trim(); if(!name)return alert("Product name দিন");
 products.unshift({id:Date.now(),name,price:+document.getElementById("price").value||0,category:document.getElementById("category").value,stock:+document.getElementById("stock").value||0,image:document.getElementById("image").value.trim(),desc:document.getElementById("desc").value.trim()});
 save();["pname","price","stock","image","desc"].forEach(id=>document.getElementById(id).value="");render();
}
function removeProduct(id){products=products.filter(p=>p.id!==id);save();render()}
function details(id){const p=products.find(x=>x.id===id);document.getElementById("modalBody").innerHTML=`${p.image?`<img src="${escapeHtml(p.image)}">`:""}<h2>${escapeHtml(p.name)}</h2><p class="meta">${escapeHtml(p.category)} • Stock ${p.stock}</p><h2>${money(p.price)}</h2><p>${escapeHtml(p.desc||"")}</p><button class="btn" onclick="alert('Added to cart')">Add to Cart</button>`;document.getElementById("modal").classList.add("show")}
function closeModal(){document.getElementById("modal").classList.remove("show")}
function toggleAdmin(){document.getElementById("admin").classList.toggle("active");window.scrollTo({top:document.getElementById("admin").classList.contains("active")?document.body.scrollHeight:0,behavior:"smooth"})}
function saveSettings(){
 settings.brand=document.getElementById("brandInput").value.trim()||settings.brand;
 settings.hero=document.getElementById("heroInput").value.trim()||settings.hero;
 settings.text=document.getElementById("heroTextInput").value.trim()||settings.text;
 localStorage.setItem("norvez_settings",JSON.stringify(settings));applySettings();alert("Store settings saved.");
}
function applySettings(){document.getElementById("brand").textContent=settings.brand;document.getElementById("heroTitle").textContent=settings.hero;document.getElementById("heroText").textContent=settings.text;document.getElementById("brandInput").value=settings.brand;document.getElementById("heroInput").value=settings.hero;document.getElementById("heroTextInput").value=settings.text}
function escapeHtml(s){return String(s??"").replace(/[&<>"']/g,m=>({"&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;","'":"&#039;"}[m]))}
applySettings();render();
</script>
</body>
</html>'''

(base/"index.html").write_text(html, encoding="utf-8")
readme = """NØRVEZ Clothing App — Demo

How to run:
1. Extract the ZIP.
2. Open index.html in Microsoft Edge or Chrome.
3. Click "Admin Panel".
4. Add products, prices, categories, stock and image URLs.
5. Store settings are saved in the browser using localStorage.

This is a frontend prototype, not a production ecommerce backend.
For a full Android/iOS store, the next version should add:
- Firebase/Supabase database
- Admin authentication
- Customer accounts
- Cart and checkout
- Payment gateway
- Order management
- Delivery integration
- Push notifications
- Cloud image storage
"""
(base/"README.txt").write_text(readme, encoding="utf-8")

zip_path=Path("/mnt/data/NORVEZ_Clothing_App_Demo.zip")
with zipfile.ZipFile(zip_path,"w",zipfile.ZIP_DEFLATED) as z:
    for f in base.iterdir():
        z.write(f, f.name)

print(f"Created: {zip_path}")
