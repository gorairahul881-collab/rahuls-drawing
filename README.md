<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Rahul's Drawing</title>
<style>
body{margin:0;font-family:Arial;text-align:center;background:#fdf6f0}
.btn{padding:10px 20px;font-size:1em;background:#ff4757;color:white;border:none;border-radius:10px;cursor:pointer;transition:.3s;margin:5px}
.btn:hover{background:#ff6b81}
#welcome,#portfolio,#order,#costConfirmation,#finalConfirmation{padding:30px 20px}
#welcome{height:100vh;display:flex;flex-direction:column;justify-content:center;align-items:center;background:linear-gradient(to right,#ffecd2,#fcb69f)}
#welcome h1{font-size:3em;margin-bottom:30px}
#portfolio,#order,#costConfirmation,#finalConfirmation{display:none}
.logo{width:150px;height:150px;border-radius:50%;object-fit:cover;margin:0 auto 20px auto;border:5px solid #333}
.gallery{display:flex;flex-wrap:wrap;justify-content:center;gap:20px;margin-bottom:30px}
.gallery-item{text-align:center}
.gallery img{width:200px;height:200px;object-fit:cover;border-radius:15px;box-shadow:0 5px 15px rgba(0,0,0,.2);cursor:pointer;transition:.3s;z-index:1;position:relative}
.gallery img:hover{transform:scale(1.05)}
#lightbox{display:none;position:fixed;top:0;left:0;width:100%;height:100%;background:rgba(0,0,0,.9);justify-content:center;align-items:center;flex-direction:column;z-index:10000}
#lightbox img{max-width:90%;max-height:80%;border-radius:15px;box-shadow:0 5px 20px rgba(0,0,0,.5)}
#lightbox .close-btn,#lightbox .nav-btn{margin-top:10px;padding:10px 20px;font-size:1em;background:#333;color:#fff;cursor:pointer}
#order input,#order textarea,#order select{padding:10px;margin:5px;width:80%;border-radius:5px;border:1px solid #ccc}
#uploadPreview{width:200px;height:200px;object-fit:cover;margin-top:10px;border-radius:10px;display:none}
#orderQueue{text-align:left;max-width:600px;margin:20px auto}
#orderQueue li{margin:5px 0;background:#fff;padding:5px;border-radius:5px;box-shadow:0 2px 5px rgba(0,0,0,0.2)}
#customerCheck input{padding:5px;width:60%}
</style>
</head>
<body>

<!-- Welcome -->
<div id="welcome">
  <h1>Welcome to my page</h1>
  <p>Looking for a perfect gift for yourself or your loved ones? Whether it's for a friend or someone special, you can find something unique here!</p>
  <button class="btn" onclick="showPortfolio()">Visit Page</button>
</div>

<!-- Portfolio -->
<div id="portfolio">
  <img src="logo.jpg" class="logo" alt="Logo">
  <div class="gallery" id="gallery"></div>
  <button class="btn" onclick="showOrderForm()">Place Your Order</button>

  <h3>Admin Orders Queue</h3>
  <ul id="orderQueue"></ul>

  <h3>Customer Order Check</h3>
  <div id="customerCheck">
    <input type="text" id="checkName" placeholder="Enter your name">
    <button class="btn" onclick="viewMyOrder()">Check My Order</button>
    <p id="customerOrderList"></p>
  </div>
</div>

<!-- Order -->
<div id="order">
  <h2>Place Your Order</h2>
  <form id="orderForm" onsubmit="submitOrder(event)">
    <input type="text" placeholder="Your Name" id="name" required><br>
    <input type="email" placeholder="Email" id="email" required><br>
    <select id="countryCode" required></select><br>
    <input type="text" id="phoneNumber" placeholder="Phone Number" required><br>
    <textarea id="details" placeholder="Drawing Details" required></textarea><br>
    <label>Upload Photo:</label><br>
    <input type="file" id="uploadPhoto" accept="image/*" onchange="previewImage(event)"><br>
    <img id="uploadPreview"><br>
    <button type="submit" class="btn">Submit Order</button>
    <button type="button" class="btn" onclick="backToPortfolio()">Back</button>
  </form>
</div>

<!-- Cost Confirmation -->
<div id="costConfirmation">
  <h2>Order Cost</h2>
  <p>Your custom drawing will cost <strong>₹250</strong>.</p>
  <button class="btn" onclick="confirmOrderFinal()">Confirm Order</button>
  <button class="btn" onclick="backToOrder()">Back</button>
</div>

<!-- Final Confirmation -->
<div id="finalConfirmation">
  <h2>Thank you for your order!</h2>
  <p>Your order has been received! Please wait, I will contact you soon.</p>
  <p>Order Number: <span id="currentOrderNumber"></span></p>
  <button class="btn" onclick="backToPortfolio()">Back</button>
</div>

<!-- Lightbox -->
<div id="lightbox" onclick="closeLightbox(event)">
  <img id="lightbox-img" alt="Lightbox">
  <div>
    <button class="nav-btn" onclick="prevImage(event)">Previous</button>
    <button class="nav-btn" onclick="nextImage(event)">Next</button>
    <button class="close-btn" onclick="closeLightbox(event)">Close</button>
  </div>
</div>

<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.4/firebase-app.js";
import { getDatabase, ref, get, set, onValue } from "https://www.gstatic.com/firebasejs/10.12.4/firebase-database.js";

// Firebase setup
const firebaseConfig = {
  apiKey: "AIzaSyCIveVNuBXKCs0xKJtVN2do12HnsolTTS8",
  authDomain: "rahul-s-drawing.firebaseapp.com",
  databaseURL: "https://rahul-s-drawing-default-rtdb.firebaseio.com",
  projectId: "rahul-s-drawing",
  storageBucket: "rahul-s-drawing.appspot.com",
  messagingSenderId: "504436503015",
  appId: "1:504436503015:web:98765af24be87807304d42",
  measurementId: "G-7D0642YPJB"
};
const app = initializeApp(firebaseConfig);
const db = getDatabase(app);

// Variables
let orderCount=0, tempOrderData=null, orders=[];
let galleryImages=[], currentLightboxIndex=0;

// Gallery
const gallery=document.getElementById('gallery');
for(let i=1;i<=14;i++){
  galleryImages.push("drawing"+i+".jpg");
  const div=document.createElement('div');
  div.className='gallery-item';
  div.innerHTML=`
    <img src="drawing${i}.jpg" onclick="openLightboxIndex(${i-1})"><br>
    <button class="btn like-btn" onclick="toggleLike('like${i}')">Like (<span id="like${i}">0</span>)</button>
  `;
  gallery.appendChild(div);

  const likeRef = ref(db, "likes/like"+i);
  onValue(likeRef,(snapshot)=>{
    document.getElementById("like"+i).innerText = snapshot.val() || 0;
  });
}

// Country codes
const countryCodes=[
  {name:"India", code:"+91"},
  {name:"USA", code:"+1"},
  {name:"UK", code:"+44"},
  {name:"Australia", code:"+61"},
  {name:"Canada", code:"+1"},
  {name:"Japan", code:"+81"},
  {name:"Germany", code:"+49"},
  {name:"France", code:"+33"},
  {name:"China", code:"+86"},
  {name:"Brazil", code:"+55"}
];
const select=document.getElementById("countryCode");
countryCodes.forEach(c=>{
  let opt=document.createElement("option");
  opt.value=c.code;
  opt.textContent=`${c.name} (${c.code})`;
  select.appendChild(opt);
});

// Page Navigation
window.showPortfolio=()=>{document.getElementById('welcome').style.display='none';document.getElementById('portfolio').style.display='block'}
window.showOrderForm=()=>{document.getElementById('portfolio').style.display='none';document.getElementById('order').style.display='block'}
window.backToPortfolio=()=>{document.getElementById('finalConfirmation').style.display='none';document.getElementById('costConfirmation').style.display='none';document.getElementById('order').style.display='none';document.getElementById('portfolio').style.display='block'}
window.backToOrder=()=>{document.getElementById('costConfirmation').style.display='none';document.getElementById('order').style.display='block'}

// Lightbox
window.openLightboxIndex=(idx)=>{currentLightboxIndex=idx;document.getElementById('lightbox-img').src=galleryImages[idx];document.getElementById('lightbox').style.display='flex'}
window.prevImage=(event)=>{event.stopPropagation();currentLightboxIndex=(currentLightboxIndex-1+galleryImages.length)%galleryImages.length;document.getElementById('lightbox-img').src=galleryImages[currentLightboxIndex]}
window.nextImage=(event)=>{event.stopPropagation();currentLightboxIndex=(currentLightboxIndex+1)%galleryImages.length;document.getElementById('lightbox-img').src=galleryImages[currentLightboxIndex]}
window.closeLightbox=(event)=>{if(event)event.stopPropagation();document.getElementById('lightbox').style.display='none'}

// Like button
window.toggleLike=async(id)=>{
  const likeRef=ref(db,"likes/"+id);
  const snapshot=await get(likeRef);
  let currentLikes = snapshot.exists() ? snapshot.val() : 0;
  currentLikes++;
  await set(likeRef,currentLikes);
  document.getElementById(id).innerText=currentLikes;
}

// Preview
window.previewImage=(event)=>{const preview=document.getElementById('uploadPreview');preview.src=URL.createObjectURL(event.target.files[0]);preview.style.display='block'}

// Submit Order (JSON download)
window.submitOrder=async(event)=>{
  event.preventDefault();
  const fileInput=document.getElementById('uploadPhoto');
  let photoFilename="";
  if(fileInput.files[0]){
    const file=fileInput.files[0];
    photoFilename=file.name.replace(/\s/g,"_");
    const photoBlob=new Blob([file],{type:file.type});
    const photoLink=document.createElement("a");
    photoLink.href=URL.createObjectURL(photoBlob);
    photoLink.download=photoFilename;
    photoLink.click();
  }

  tempOrderData={
    orderNumber:++orderCount,
    name:document.getElementById('name').value,
    email:document.getElementById('email').value,
    phone:document.getElementById('countryCode').value+' '+document.getElementById('phoneNumber').value,
    details:document.getElementById('details').value,
    photoFilename:photoFilename
  };

  const blob=new Blob([JSON.stringify(tempOrderData,null,2)],{type:"application/json"});
  const a=document.createElement("a");
  a.href=URL.createObjectURL(blob);
  a.download=tempOrderData.name.replace(/\s/g,"_")+"_order.json";
  a.click();

  document.getElementById('order').style.display='none';
  document.getElementById('costConfirmation').style.display='block';
}

// Confirm final + send SMS
window.confirmOrderFinal=async()=>{
  document.getElementById('costConfirmation').style.display='none';
  document.getElementById('finalConfirmation').style.display='block';
  document.getElementById('currentOrderNumber').innerText=tempOrderData.orderNumber;
  orders.push({number:tempOrderData.orderNumber,name:tempOrderData.name});
  updateOrderQueue();

  // Send SMS automatically via backend
  try{
    await fetch('http://localhost:3000/send-sms',{
      method:'POST',
      headers:{'Content-Type':'application/json'},
      body:JSON.stringify({
        phone: tempOrderData.phone.replace(/\s/g,''), // remove spaces
        name: tempOrderData.name,
        orderNumber: tempOrderData.orderNumber
      })
    });
    console.log('SMS sent successfully.');
  }catch(err){
    console.error('Error sending SMS:',err);
  }

  // Clear form
  document.getElementById('name').value='';
  document.getElementById('email').value='';
  document.getElementById('phoneNumber').value='';
  document.getElementById('details').value='';
  document.getElementById('uploadPhoto').value='';
  document.getElementById('uploadPreview').style.display='none';
  tempOrderData=null;
};

// Order queue
function updateOrderQueue(){
  const orderQueue=document.getElementById('orderQueue');
  orderQueue.innerHTML='';
  orders.sort((a,b)=>a.number-b.number);
  orders.forEach((order,index)=>{
    const li=document.createElement('li');
    li.innerHTML=`#${order.number} - ${order.name} <button class="btn" onclick="confirmDelivery(${index})">Confirm Delivery</button>`;
    orderQueue.appendChild(li);
  });
}
window.confirmDelivery=(index)=>{orders.splice(index,1);updateOrderQueue();}
window.viewMyOrder=()=>{
  const inputName=document.getElementById('checkName').value.trim().toLowerCase();
  const myOrders=orders.filter(o=>o.name.toLowerCase()===inputName);
  const customerOrderList=document.getElementById('customerOrderList');
  if(myOrders.length){customerOrderList.innerText='Your Orders: '+myOrders.map(o=>o.number).join(', ');}
  else{customerOrderList.innerText='No orders found with this name.';}
}
</script>

</body>
</html>
