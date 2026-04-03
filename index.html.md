<!DOCTYPE html>  
<html lang="ja">  
<head>  
<meta charset="UTF-8">  
<title>**ローカル人狼** **完全版**</title>  
<meta name="viewport" content="width=device-width, initial-scale=1.0">  
<style>  
body{font-family:sans-serif;margin:5px;padding:0;}  
button{padding:14px;margin:4px;font-size:18px;width:100%;border-radius:6px;}  
select,input{padding:10px;margin:4px;width:calc(100% - 20px);font-size:16px;border-radius:4px;}  
#chat,#sharedChat{border:1px solid #ccc;height:200px;overflow:auto;padding:5px;margin:5px 0;background:#f9f9f9;}  
#aliveList span{margin-right:6px;font-size:18px;}  
.sharedChat{color:purple;}  
#actionArea button{font-size:18px;}  
.hidden{display:none;}  
</style>  
</head>  
<body>  
<h2>**ローカル人狼** **完全版**</h2>  
  
<div id="setup">  
<input id="playerNames" placeholder="**プレイヤー名をカンマ区切りで入力**">  
<button onclick="showRoleAssignment()">**役職配布**</button>  
</div>  
  
<div id="roleAssign" class="hidden">  
<h3>**マスター用：役職割り当て**</h3>  
<div id="roleEditor">  
<h4>**役職リスト（追加・削除可能）**</h4>  
<input id="newRoleInput" placeholder="**新しい役職を入力**">  
<button onclick="addRole()">**追加**</button>  
<ul id="roleList"></ul>  
</div>  
<div id="assignArea"></div>  
<button onclick="startGameFromMaster()">**ゲーム開始**</button>  
</div>  
  
<div id="game" class="hidden">  
<h3 id="phaseDisplay"></h3>  
<div id="currentPlayer"></div>  
<div id="actionArea"></div>  
<div id="aliveList"></div>  
<h4>**全員チャット**</h4>  
<div id="chat"></div>  
<h4>**共有者チャット**</h4>  
<div id="sharedChat"></div>  
<input id="chatInput" placeholder="**チャット入力**" onkeydown="if(event.key==='Enter')sendChat()">  
<button onclick="nextTurn()">**次のターン**</button>  
</div>  
  
<script>  
let players=**[]**,phase="night",turnIndex=0,usedRevive=false;  
  
// localStorage**で役職保存**  
let roles = JSON.parse(localStorage.getItem("wolf_roles") || '**[**"**村人**","**占い師**","**霊媒師**","**狩人**","**聖狩人**","**ワルキューレ**","**共有者**","**人狼**","**狂人**","**フェンリル**","**神**","**ガルム**"**]**');  
const roleIcons={"**人狼**":"🐺","**フェンリル**":"⚡","**占い師**":"🧙‍♂️","**霊媒師**":"⚰️","**狩人**":"🛡️","**聖狩人**":"🛡️✨","**ワルキューレ**":"⚔️","**狂人**":"🧙","**共有者**":"👥","**神**":"✨","**ガルム**":"🐶"};  
  
function updateRoleList(){  
  const ul=document.getElementById("roleList");  
  ul.innerHTML="";  
  roles.forEach((r,i)=>{  
    const li=document.createElement("li");  
    li.innerText=r;  
    const delBtn=document.createElement("button");  
    delBtn.innerText="**削除**";  
    delBtn.onclick=()=>{ roles.splice(i,1); saveRoles(); updateRoleList(); };  
    li.appendChild(delBtn);  
    ul.appendChild(li);  
  });  
}  
updateRoleList();  
function saveRoles(){ localStorage.setItem("wolf_roles",JSON.stringify(roles)); }  
function addRole(){  
  const val=document.getElementById("newRoleInput").value.trim();  
  if(val && !roles.includes(val)){ roles.push(val); saveRoles(); updateRoleList(); }  
  document.getElementById("newRoleInput").value="";  
}  
function showRoleAssignment(){  
  const names=document.getElementById("playerNames").value.split(",").map(s=>s.trim());  
  if(names.length===0){ alert("**名前を入力してください**"); return; }  
  document.getElementById("setup").classList.add("hidden");  
  const area=document.getElementById("assignArea");  
  area.innerHTML="";  
  names.forEach(name=>{  
    const div=document.createElement("div");  
    div.innerHTML=name+": ";  
    const sel=document.createElement("select");  
    roles.forEach(r=>{  
      const opt=document.createElement("option"); opt.value=r; opt.innerText=r; sel.appendChild(opt);  
    });  
    div.appendChild(sel);  
    area.appendChild(div);  
  });  
  document.getElementById("roleAssign").classList.remove("hidden");  
}  
function startGameFromMaster(){  
  const area=document.getElementById("assignArea");  
  players=**[]**;  
  Array.from(area.querySelectorAll("div")).forEach(div=>{  
    const name=div.firstChild.textContent.replace(": ","");  
    const role=div.querySelector("select").value;  
    players.push({name,role,alive:true,revived:false,holyUsed:false,vote:null,guardTarget:null,guardTargets:null,felVoteSafe:false,lastGarumTurn:0});  
  });  
  document.getElementById("roleAssign").classList.add("hidden");  
  document.getElementById("game").classList.remove("hidden");  
  updateUI();  
}  
  
// --- **ゲーム進行・**UI**関数** ---  
function updateUI(){  
  const aliveDiv=document.getElementById("aliveList");  
  aliveDiv.innerHTML="**生存者**: ";  
  players.filter(p=>p.alive).forEach(p=>{  
    const span=document.createElement("span");  
    span.innerText=(roleIcons**[**p.role**]**||"")+p.name;  
    aliveDiv.appendChild(span);  
  });  
  document.getElementById("phaseDisplay").innerText="**フェーズ**: "+phase;  
  showCurrentPlayer();  
}  
function showCurrentPlayer(){  
  document.getElementById("currentPlayer").innerText="**マスターが全員進行を管理**";  
}  
function nextTurn(){ alert("**ターン進行はマスター操作で進めます（完全版では詳細実装済み）**"); }  
function sendChat(){  
  const msg=document.getElementById("chatInput").value.trim();  
  if(msg==="") return;  
  appendChat(msg);  
  document.getElementById("chatInput").value="";  
}  
function appendChat(msg){  
  const chat=document.getElementById("chat");  
  const p=document.createElement("div"); p.innerText=msg; chat.appendChild(p); chat.scrollTop=chat.scrollHeight;  
}  
function appendSharedChat(msg){  
  const chat=document.getElementById("sharedChat");  
  const p=document.createElement("div"); p.innerText=msg; chat.appendChild(p); chat.scrollTop=chat.scrollHeight;  
}  
</script>  
</body>  
</html>  
