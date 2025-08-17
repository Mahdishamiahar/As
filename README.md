<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Al.VIPZEXNET - چت هوش مصنوعی</title>
<style>
  body {
    margin: 0;
    font-family: "Segoe UI", sans-serif;
    background: linear-gradient(135deg, #0d1117, #1a1f29);
    color: #fff;
    display: flex;
    flex-direction: column;
    height: 100vh;
    overflow: hidden;
  }
  header {
    background: #161b22cc;
    backdrop-filter: blur(8px);
    padding: 10px 16px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    border-bottom: 1px solid #30363d;
  }
  header h1 { margin: 0; font-size: 18px; color: #58a6ff; }
  header select, header button {
    background: #0d1117;
    color: #fff;
    border: 1px solid #30363d;
    border-radius: 6px;
    padding: 6px 10px;
    cursor: pointer;
  }

  #chatWindow {
    flex: 1;
    overflow-y: auto;
    padding: 24px;
    display: flex;
    flex-direction: column;
    gap: 20px;
    scrollbar-width: thin;
  }

  .message-wrapper {
    display: flex;
    align-items: flex-end;
    gap: 12px;
  }
  .avatar {
    width: 40px;
    height: 40px;
    border-radius: 50%;
    flex-shrink: 0;
    background-size: cover;
    background-position: center;
  }
  .message {
    max-width: 75%;
    padding: 16px 20px;
    border-radius: 16px;
    line-height: 1.6;
    white-space: pre-wrap;
    box-shadow: 0 4px 10px rgba(0,0,0,0.3);
    word-break: break-word;
    opacity: 0;
    transform: translateY(5px);
    animation: fadeIn 0.3s ease forwards;
    font-size: 16px;
  }
  .user { background: #238636; color: #fff; border-bottom-right-radius: 4px; margin-left: auto; }
  .bot { background: #21262d; color: #fff; border-bottom-left-radius: 4px; margin-right: auto; }

  #inputArea {
    display: flex;
    padding: 16px;
    border-top: 1px solid #30363d;
    background: #161b22cc;
    backdrop-filter: blur(6px);
  }
  #userInput {
    flex: 1;
    padding: 12px 16px;
    border: none;
    border-radius: 12px;
    background: #0d1117;
    color: #fff;
    font-size: 16px;
    resize: none;
    max-height: 150px;
    line-height: 1.5;
  }
  #sendBtn {
    padding: 12px 20px;
    margin-left: 12px;
    background: #58a6ff;
    border: none;
    border-radius: 12px;
    cursor: pointer;
    color: #fff;
    font-size: 16px;
  }
  #sendBtn:hover { background: #1f6feb; }

  @keyframes fadeIn { to {opacity: 1; transform: translateY(0);} }

  #typingIndicator {
    display: flex;
    align-items: center;
    gap: 6px;
    margin-right: auto;
    font-size: 13px;
    color: #aaa;
  }
  #typingIndicator span {
    display:inline-block;
    width:7px; height:7px;
    background:#58a6ff;
    border-radius:50%;
    animation: blink 1s infinite;
  }
  #typingIndicator span:nth-child(2) { animation-delay:0.2s; }
  #typingIndicator span:nth-child(3) { animation-delay:0.4s; }
  @keyframes blink { 0%,80%,100%{opacity:0;} 40%{opacity:1;} }

  @media (max-width: 768px) {
    header h1 { font-size: 16px; }
    #chatWindow { padding: 16px; gap: 16px; }
    .message { max-width: 85%; font-size: 15px; padding: 14px 18px; }
    #userInput { font-size: 15px; padding: 10px 14px; }
    #sendBtn { font-size: 15px; padding: 10px 16px; }
  }
  @media (max-width: 480px) {
    header { flex-direction: column; align-items: flex-start; gap: 8px; }
    header select, header button { width: 100%; }
    .message { max-width: 90%; }
    #inputArea { flex-direction: column; gap: 10px; }
    #sendBtn { width: 100%; margin-left: 0; }
  }
</style>
</head>
<body>
<header>
<h1>🤖 Al.VIPZEXNET</h1>
<select id="modelSelect">
  <option value="openai">OpenAI</option>
  <option value="deepseek">DeepSeek</option>
  <option value="mistral">Mistral</option>
  <option value="openai-large">OpenAI Large (GPT-4)</option>
</select>
<button id="clearBtn">پاک کردن چت</button>
</header>

<div id="chatWindow"></div>

<div id="inputArea">
<textarea id="userInput" placeholder="پیام خود را به Al.VIPZEXNET بنویسید..." rows="1"
onkeydown="if(event.key==='Enter' && !event.shiftKey){event.preventDefault();sendMessage()}"></textarea>
<button id="sendBtn" onclick="sendMessage()">ارسال</button>
</div>

<script>
let selectedModel = "openai";
const chatHistory = [];

document.getElementById("modelSelect").addEventListener("change", function () {
  selectedModel = this.value;
});

document.getElementById("clearBtn").addEventListener("click", () => {
  chatHistory.length = 0;
  document.getElementById("chatWindow").innerHTML = "";
});

function addMessage(text, sender, index=null) {
  const chatWindow = document.getElementById("chatWindow");
  const wrapper = document.createElement("div");
  wrapper.className = "message-wrapper " + sender;

  const avatar = document.createElement("div");
  avatar.className = "avatar";
  avatar.style.backgroundImage = sender === "user" 
    ? "url('https://i.imgur.com/0y0y0y0.png')" 
    : "url('https://i.imgur.com/q1bPzOZ.png')";

  const msgDiv = document.createElement("div");
  msgDiv.className = "message " + sender;
  msgDiv.textContent = text;

  if (sender === "user") {
    msgDiv.style.cursor = "pointer";
    msgDiv.title = "برای ویرایش کلیک کنید";
    msgDiv.addEventListener("click", () => editMessage(msgDiv, index));
    wrapper.appendChild(msgDiv);
    wrapper.appendChild(avatar);
  } else {
    wrapper.appendChild(avatar);
    wrapper.appendChild(msgDiv);
  }

  chatWindow.appendChild(wrapper);
  chatWindow.scrollTop = chatWindow.scrollHeight;
}

function editMessage(msgDiv, index) {
  const original = msgDiv.textContent;
  const input = document.createElement("textarea");
  input.value = original;
  input.style.width = "100%";
  input.style.padding="6px";
  input.style.borderRadius="8px";
  input.style.boxSizing="border-box";
  msgDiv.replaceWith(input);
  input.focus();

  input.addEventListener("keydown", async e=>{
    if(e.key==="Enter" && !e.shiftKey){
      e.preventDefault();
      const newText = input.value;
      chatHistory[index].user=newText;
      const newMsg=document.createElement("div");
      newMsg.className="message user";
      newMsg.textContent=newText;
      newMsg.style.cursor="pointer";
      newMsg.title="برای ویرایش کلیک کنید";
      newMsg.addEventListener("click",()=>editMessage(newMsg,index));
      input.replaceWith(newMsg);
      await updateBotResponse(index);
    } else if(e.key==="Escape"){
      const oldMsg=document.createElement("div");
      oldMsg.className="message user";
      oldMsg.textContent=original;
      oldMsg.style.cursor="pointer";
      oldMsg.title="برای ویرایش کلیک کنید";
      oldMsg.addEventListener("click",()=>editMessage(oldMsg,index));
      input.replaceWith(oldMsg);
    }
  });
}

async function updateBotResponse(index){
  const loading = document.createElement("div");
  loading.className="message bot";
  loading.textContent="⌛ بروزرسانی پاسخ...";
  document.getElementById("chatWindow").appendChild(loading);

  try{
    const formData=new FormData();
    formData.append("userid","user123");
    formData.append("prompt",chatHistory[index].user);
    formData.append("model",selectedModel);

    const res=await fetch("https://api2.api-code.ir/gpt-save-v2/",{method:"POST",body:formData});
    const data=await res.text();
    loading.remove();
    chatHistory[index].bot=data;
    addMessage(data,"bot",index);
  } catch(err){
    loading.remove();
    addMessage("❌ خطا در بروزرسانی پاسخ","bot",index);
  }
}

async function sendMessage(){
  const input=document.getElementById("userInput");
  const text=input.value.trim();
  if(!text) return;
  const index=chatHistory.length;
  chatHistory.push({user:text,bot:""});
  addMessage(text,"user",index);
  input.value=""; input.style.height="auto";

  const typing=document.createElement("div");
  typing.id="typingIndicator";
  typing.innerHTML="<span></span><span></span><span></span> Al.VIPZEXNET در حال تایپ ...";
  document.getElementById("chatWindow").appendChild(typing);
  document.getElementById("chatWindow").scrollTop=document.getElementById("chatWindow").scrollHeight;

  try{
    const formData=new FormData();
    formData.append("userid","user123");
    formData.append("prompt",text);
    formData.append("model",selectedModel);

    const res=await fetch("https://api2.api-code.ir/gpt-save-v2/",{method:"POST",body:formData});
    const data=await res.text();
    typing.remove();
    chatHistory[index].bot=data;
    addMessage(data,"bot",index);
  } catch(err){
    typing.remove();
    addMessage("❌ خطا در دریافت پاسخ","bot",index);
  }
}
</script>
</body>
</html>
