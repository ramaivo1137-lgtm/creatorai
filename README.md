# creatorai

<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<meta name="theme-color" content="#101426">
<title>DramaAI Generator</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#101426;color:#fff;font-family:Arial,sans-serif;padding:18px}
header{text-align:center;padding:20px 0}
h1{margin:0;color:#9b8cff;font-size:28px}
header p{color:#aeb4ce;font-size:13px}
.panel{background:#1b2239;padding:16px;border-radius:15px;margin:14px 0}
label{display:block;margin:12px 0 7px;color:#c8cbe0;font-size:13px}
input,select,textarea{width:100%;padding:12px;border:1px solid #39415f;border-radius:9px;background:#11182b;color:white;font-size:15px}
button{border:0;border-radius:10px;padding:13px;background:#7663ef;color:white;font-weight:bold;font-size:14px;margin-top:12px;width:100%}
button.secondary{background:#303956}
button:disabled{opacity:.5}
.scene{background:#252d49;padding:13px;border-radius:12px;margin:12px 0}
.scene h3{color:#b5aaff;margin:0 0 8px}
.scene p{font-size:14px;line-height:1.5;color:#e0e3f3}
.art{width:100%;border-radius:9px;background:#11182b;margin-top:8px}
.small{font-size:12px;color:#aeb4ce;line-height:1.5}
#output{display:none}
</style>
</head>
<body>

<header>
<h1>🎬 DramaAI Generator</h1>
<p>Buat cerita, adegan, ilustrasi dan voice-over</p>
</header>

<div class="panel">
<h3>✨ Buat Cerita Baru</h3>

<label>Judul cerita</label>
<input id="title" placeholder="Contoh: Rahasia di Balik Hujan">

<label>Genre</label>
<select id="genre">
<option>Drama keluarga</option>
<option>Romantis ringan</option>
<option>Misteri</option>
<option>Horor ringan</option>
<option>Persahabatan</option>
<option>Inspirasi kehidupan</option>
</select>

<label>Karakter utama</label>
<input id="character" placeholder="Contoh: Alya">

<label>Lokasi cerita</label>
<input id="location" placeholder="Contoh: Kota kecil saat hujan">

<label>Jumlah adegan</label>
<select id="count">
<option value="3">3 adegan</option>
<option value="5" selected>5 adegan</option>
<option value="7">7 adegan</option>
</select>

<button onclick="generate()">✨ GENERATE CERITA</button>
</div>

<div id="output" class="panel">
<h2 id="storyTitle"></h2>
<p id="storyIntro" class="small"></p>
<div id="scenes"></div>

<button onclick="playVoice()">🔊 Putar Voice Over</button>
<button class="secondary" onclick="stopVoice()">⏹ Hentikan Suara</button>
<button class="secondary" onclick="saveStory()">💾 Simpan Cerita</button>
<button class="secondary" onclick="downloadStory()">⬇️ Download Naskah</button>
<button class="secondary" onclick="exportVideo()">🎥 Buat Video Slideshow</button>
<p id="status" class="small"></p>
</div>

<div class="panel">
<h3>📚 Cerita Tersimpan</h3>
<div id="savedList" class="small">Belum ada cerita tersimpan.</div>
</div>

<p class="small">DramaAI Generator berjalan di browser. Cerita dibuat dari template lokal. Voice-over memakai fitur suara perangkat jika tersedia.</p>

<script>
let currentScenes=[];
let currentTitle="";

const templates=[
{
intro:"Semuanya berubah ketika sebuah rahasia lama akhirnya terungkap.",
scenes:[
["Awal cerita","Hari itu terlihat seperti hari biasa. Namun, ada sesuatu yang terasa berbeda.","Wide cinematic shot, quiet street, atmospheric lighting"],
["Pertemuan","Di tempat yang tak terduga, sebuah pertemuan mengubah segalanya.","Two characters meeting, cinematic composition"],
["Rahasia","Sebuah pesan lama membuat semua pertanyaan mulai menemukan jawabannya.","Old letter on a wooden table, dramatic light"],
["Konflik","Kini ia harus memilih antara masa lalu dan masa depan.","Person standing by window, emotional cinematic scene"],
["Akhir","Pada akhirnya, ia menyadari bahwa keberanian dimulai dari satu keputusan.","Sunrise over peaceful town, hopeful cinematic ending"]
]
},
{
intro:"Satu keputusan kecil membawa perjalanan yang tidak pernah dibayangkan.",
scenes:[
["Mimpi","Ia selalu memiliki mimpi besar, meski tak seorang pun percaya.","Person looking at distant city lights"],
["Tantangan","Perjalanan itu ternyata jauh lebih sulit daripada yang dibayangkan.","Lonely road under cloudy sky"],
["Keraguan","Untuk sesaat, ia hampir menyerah pada keadaan.","Person sitting alone beside window"],
["Kesadaran","Kemudian ia menemukan alasan untuk mencoba sekali lagi.","Warm light entering dark room"],
["Harapan","Langkah kecil hari ini menjadi awal dari perubahan besar.","Person walking toward sunrise"]
]
},
{
intro:"Tidak semua orang yang terlihat bahagia sedang menyimpan cerita sederhana.",
scenes:[
["Senyum","Di depan semua orang, ia selalu terlihat baik-baik saja.","Person smiling in a crowded room"],
["Kesunyian","Namun ketika malam tiba, pikirannya dipenuhi banyak pertanyaan.","Person near window at night"],
["Petunjuk","Seseorang akhirnya menyadari ada sesuatu yang disembunyikan.","Friend noticing a quiet person"],
["Kejujuran","Untuk pertama kalinya, ia berani menceritakan semuanya.","Two friends talking in a quiet cafe"],
["Awal baru","Ternyata, didengarkan adalah awal dari sebuah perubahan.","Two people watching sunrise"]
]
}
];

function escapeHTML(s){
return String(s).replace(/[&<>"']/g,c=>({
"&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;","'":"&#39;"
}[c]));
}

function generate(){
const title=document.getElementById("title").value.trim()||"Kisah yang Tak Terlupakan";
const character=document.getElementById("character").value.trim()||"Alex";
const location=document.getElementById("location").value.trim()||"sebuah kota kecil";
const genre=document.getElementById("genre").value;
const count=Number(document.getElementById("count").value);
const template=templates[Math.floor(Math.random()*templates.length)];

currentTitle=title;
currentScenes=template.scenes.slice(0,count).map((s,i)=>({
heading:s[0],
narration:s[1].replace(/Ia|ia/g,character),
visual:s[2],
location:location,
number:i+1
}));

document.getElementById("storyTitle").textContent=title;
document.getElementById("storyIntro").textContent=
"Genre: "+genre+" | Lokasi: "+location+" | "+template.intro;

renderScenes();
document.getElementById("output").style.display="block";
document.getElementById("status").textContent="Cerita berhasil dibuat.";
}

function makeSVG(scene){
const title=escapeHTML(scene.heading);
const location=escapeHTML(scene.location);
const visual=escapeHTML(scene.visual);
return `<svg xmlns="http://www.w3.org/2000/svg" width="900" height="506" viewBox="0 0 900 506">
<defs>
<linearGradient id="bg" x1="0" y1="0" x2="1" y2="1">
<stop stop-color="#202b50"/><stop offset="1" stop-color="#7155a9"/>
</linearGradient>
</defs>
<rect width="900" height="506" fill="url(#bg)"/>
<circle cx="700" cy="130" r="65" fill="#f4dca0" opacity=".8"/>
<path d="M0 390 L180 230 L340 390 L510 210 L720 390 L900 260 V506 H0Z" fill="#11182b" opacity=".8"/>
<circle cx="450" cy="250" r="38" fill="#e9e5ff"/>
<path d="M390 390 Q450 270 510 390" fill="#d2c7ff"/>
<text x="45" y="65" font-size="27" fill="white" font-family="Arial">${title}</text>
<text x="45" y="445" font-size="19" fill="white" font-family="Arial">${location}</text>
<text x="45" y="477" font-size="13" fill="#ded7ff" font-family="Arial">${visual}</text>
</svg>`;
}

function renderScenes(){
const area=document.getElementById("scenes");
area.innerHTML="";
currentScenes.forEach(scene=>{
const div=document.createElement("div");
div.className="scene";
const svg=makeSVG(scene);
const src="data:image/svg+xml;charset=utf-8,"+encodeURIComponent(svg);
div.innerHTML=`
<h3>Adegan ${scene.number}: ${escapeHTML(scene.heading)}</h3>
<p><b>Narasi:</b> ${escapeHTML(scene.narration)}</p>
<p><b>Prompt visual:</b> ${escapeHTML(scene.visual)}, ${escapeHTML(scene.location)}, cinematic lighting, 16:9.</p>
<img class="art" alt="Ilustrasi adegan ${scene.number}" src="${src}">
<button class="secondary" onclick="downloadSVG(${scene.number-1})">Download Ilustrasi</button>`;
area.appendChild(div);
});
}

function playVoice(){
if(!("speechSynthesis" in window)){
alert("Browser ini tidak mendukung pembacaan suara.");
return;
}
stopVoice();
const text=currentScenes.map(s=>s.narration).join(" ");
const utterance=new SpeechSynthesisUtterance(text);
utterance.lang="id-ID";
utterance.rate=0.9;
utterance.pitch=1;
speechSynthesis.speak(utterance);
document.getElementById("status").textContent="Voice-over sedang diputar.";
}

function stopVoice(){
if("speechSynthesis" in window)speechSynthesis.cancel();
}

function saveStory(){
if(!currentScenes.length)return;
let saved=JSON.parse(localStorage.getItem("dramaStories")||"[]");
saved.unshift({title:currentTitle,scenes:currentScenes});
localStorage.setItem("dramaStories",JSON.stringify(saved.slice(0,20)));
showSaved();
document.getElementById("status").textContent="Cerita tersimpan di perangkat.";
}

function showSaved(){
const saved=JSON.parse(localStorage.getItem("dramaStories")||"[]");
const area=document.getElementById("savedList");
if(!saved.length){area.textContent="Belum ada cerita tersimpan.";return;}
area.innerHTML="";
saved.forEach((s,i)=>{
const b=document.createElement("button");
b.className="secondary";
b.textContent="📖 "+s.title;
b.onclick=()=>{
currentTitle=s.title;
currentScenes=s.scenes;
document.getElementById("storyTitle").textContent=s.title;
renderScenes();
document.getElementById("output").style.display="block";
};
area.appendChild(b);
});
}

function downloadText(name,text){
const blob=new Blob([text],{type:"text/plain;charset=utf-8"});
const url=URL.createObjectURL(blob);
const a=document.createElement("a");
a.href=url;a.download=name;
a.click();
URL.revokeObjectURL(url);
}

function downloadStory(){
const text=currentTitle+"\n\n"+currentScenes.map(s=>
"Adegan "+s.number+": "+s.heading+"\n"+s.narration+"\nVisual: "+s.visual
).join("\n\n");
downloadText("naskah-drama.txt",text);
}

function downloadSVG(index){
const blob=new Blob([makeSVG(currentScenes[index])],{type:"image/svg+xml"});
const url=URL.createObjectURL(blob);
const a=document.createElement("a");
a.href=url;a.download="adegan-"+(index+1)+".svg";
a.click();
URL.revokeObjectURL(url);
}

async function exportVideo(){
if(!currentScenes.length)return;
if(!window.MediaRecorder||!HTMLCanvasElement.prototype.captureStream){
alert("Ekspor video belum didukung browser ini. Download ilustrasi dan naskah, lalu gabungkan di CapCut.");
return;
}
const canvas=document.createElement("canvas");
canvas.width=1280;canvas.height=720;
const ctx=canvas.getContext("2d");
const stream=canvas.captureStream(1);
let mime="video/webm";
if(MediaRecorder.isTypeSupported("video/mp4"))mime="video/mp4";
else if(!MediaRecorder.isTypeSupported(mime)){
alert("Format video tidak didukung browser ini. Gunakan CapCut untuk menggabungkan adegan.");
return;
}
const recorder=new MediaRecorder(stream,{mimeType:mime});
const chunks=[];
recorder.ondataavailable=e=>{if(e.data.size)chunks.push(e.data);};
recorder.onstop=()=>{
const blob=new Blob(chunks,{type:mime});
const url=URL.createObjectURL(blob);
const a=document.createElement("a");
a.href=url;
a.download="drama-slideshow."+(mime.includes("mp4")?"mp4":"webm");
a.click();
URL.revokeObjectURL(url);
document.getElementById("status").textContent="Video slideshow selesai dibuat.";
};
recorder.start();
for(let i=0;i<currentScenes.length;i++){
const s=currentScenes[i];
const gradient=ctx.createLinearGradient(0,0,1280,720);
gradient.addColorStop(0,"#202b50");
gradient.addColorStop(1,"#7155a9");
ctx.fillStyle=gradient;ctx.fillRect(0,0,1280,720);
ctx.fillStyle="#f4dca0";ctx.beginPath();ctx.arc(990,160,75,0,Math.PI*2);ctx.fill();
ctx.fillStyle="#11182b";
ctx.beginPath();ctx.moveTo(0,570);ctx.lineTo(300,300);ctx.lineTo(520,570);ctx.lineTo(760,280);ctx.lineTo(1280,570);ctx.lineTo(1280,720);ctx.lineTo(0,720);ctx.fill();
ctx.fillStyle="white";ctx.font="bold 48px Arial";
ctx.fillText("ADEGAN "+s.number,65,100);
ctx.font="bold 42px Arial";ctx.fillText(s.heading,65,180);
ctx.font="28px Arial";
const words=s.narration.split(" ");let line="",y=530;
for(const word of words){
if(ctx.measureText(line+word).width>1100){ctx.fillText(line,65,y);line="";y+=42;}
line+=word+" ";
}
ctx.fillText(line,65,y);
await new Promise(resolve=>setTimeout(resolve,2200));
}
recorder.stop();
stream.getTracks().forEach(t=>t.stop());
}

showSaved();
</script>
</body>
</html>
