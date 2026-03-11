<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>在线问卷系统</title>
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css">
<style>
body{background:#f8f9fa;padding:20px}
.card{margin-bottom:20px;border-radius:12px;box-shadow:0 2px 10px rgba(0,0,0,0.08)}
.question-item{background:#f8f9fa;padding:15px;border-radius:10px;margin-bottom:12px}
.modal-backdrop{opacity:0.5 !important}
</style>
</head>
<body>
<div class="container" style="max-width:900px">
  <h2 class="mb-4 text-center">📋 在线问卷系统</h2>
  <ul class="nav nav-pills mb-4 justify-content-center">
    <li class="nav-item"><a class="nav-link active" data-page="fill">填写问卷</a></li>
    <li class="nav-item"><a class="nav-link" data-page="edit" onclick="checkAdmin('edit')">编辑问卷</a></li>
    <li class="nav-item"><a class="nav-link" data-page="result" onclick="checkAdmin('result')">查看结果</a></li>
  </ul>

  <!-- 填写问卷（所有人可见） -->
  <div class="page" id="fill-page" style="display:block">
    <div class="card p-4" id="fill-card"></div>
  </div>

  <!-- 编辑问卷（需密码） -->
  <div class="page" id="edit-page" style="display:none">
    <div class="card p-4">
      <div class="mb-3">
        <label>问卷标题</label>
        <input class="form-control" id="survey-title" value="调查问卷">
      </div>
      <div id="question-list"></div>
      <button class="btn btn-primary mt-3" onclick="addQuestion()">+ 添加题目</button>
      <button class="btn btn-success mt-3 ms-2" onclick="saveSurvey()">💾 保存并发布</button>
    </div>
  </div>

  <!-- 查看结果（需密码） -->
  <div class="page" id="result-page" style="display:none">
    <div class="card p-4">
      <h5>📊 填写结果</h5>
      <div id="result-list"></div>
    </div>
  </div>
</div>

<!-- 密码验证弹窗 -->
<div class="modal fade" id="adminModal" tabindex="-1">
  <div class="modal-dialog">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title">管理员验证</h5>
        <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
      </div>
      <div class="modal-body">
        <input type="password" id="adminPwd" class="form-control" placeholder="请输入管理员密码">
      </div>
      <div class="modal-footer">
        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">取消</button>
        <button type="button" class="btn btn-primary" onclick="verifyAdmin()">确认</button>
      </div>
    </div>
  </div>
</div>

<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
<script>
// 配置：管理员密码（可自行修改）
const ADMIN_PASSWORD = "admin123";
let isAdmin = false;
let targetPage = "";
let survey = { title: "问卷", questions: [] };
let answers = [];

// 页面切换
document.querySelectorAll(".nav-link").forEach(tab => {
  tab.onclick = (e) => {
    if (tab.dataset.page === "fill") {
      switchPage("fill");
      loadFill();
    }
  };
});

function switchPage(page) {
  document.querySelectorAll(".nav-link").forEach(t => t.classList.remove("active"));
  document.querySelector(`[data-page="${page}"]`).classList.add("active");
  document.querySelectorAll(".page").forEach(p => p.style.display = "none");
  document.getElementById(page + "-page").style.display = "block";
}

// 管理员验证
function checkAdmin(page) {
  if (isAdmin) {
    switchPage(page);
    if (page === "edit") renderEdit();
    if (page === "result") loadResult();
  } else {
    targetPage = page;
    new bootstrap.Modal(document.getElementById("adminModal")).show();
  }
}

function verifyAdmin() {
  const pwd = document.getElementById("adminPwd").value;
  if (pwd === ADMIN_PASSWORD) {
    isAdmin = true;
    bootstrap.Modal.getInstance(document.getElementById("adminModal")).hide();
    switchPage(targetPage);
    if (targetPage === "edit") renderEdit();
    if (targetPage === "result") loadResult();
  } else {
    alert("密码错误！");
  }
}

// 编辑问卷相关
function renderEdit() {
  const s = JSON.parse(localStorage.getItem("survey")) || survey;
  document.getElementById("survey-title").value = s.title;
  document.getElementById("question-list").innerHTML = "";
  s.questions.forEach(q => addQuestion(q));
}

function addQuestion(d = null) {
  d = d || {type:"radio",title:"",options:["选项1","选项2"]};
  const dom = document.createElement("div");
  dom.className = "question-item";
  dom.innerHTML = `
    <input class="form-control q-title mb-2" placeholder="题目" value="${d.title}">
    <select class="form-select q-type mb-2">
      <option value="radio" ${d.type==="radio"?"selected":""}>单选</option>
      <option value="checkbox" ${d.type==="checkbox"?"selected":""}>多选</option>
      <option value="text" ${d.type==="text"?"selected":""}>填空</option>
    </select>
    <div class="opts">
      ${d.options.map(o=>`<input class="form-control mt-2 q-option" value="${o}">`).join('')}
    </div>
    <button class="btn btn-sm btn-outline-primary mt-2" onclick="addOpt(this)">+选项</button>
    <button class="btn btn-sm btn-danger mt-2 ms-2" onclick="del(this)">删除</button>
  `;
  document.getElementById("question-list").appendChild(dom);
}

function addOpt(e){
  const i=document.createElement("input");
  i.className="form-control mt-2 q-option";
  e.previousElementSibling.appendChild(i);
}
function del(e){e.closest(".question-item").remove()}

function saveSurvey(){
  const t=document.getElementById("survey-title").value;
  const qs=[];
  document.querySelectorAll(".question-item").forEach(it=>{
    const tt=it.querySelector(".q-title").value;
    const ty=it.querySelector(".q-type").value;
    const os=[...it.querySelectorAll(".q-option")].map(x=>x.value).filter(Boolean);
    if(tt)qs.push({title:tt,type:ty,options:os});
  });
  survey={title:t,questions:qs};
  localStorage.setItem("survey",JSON.stringify(survey));
  alert("保存成功！");
}

// 填写问卷相关
function loadFill(){
  const s=JSON.parse(localStorage.getItem("survey"))||survey;
  const c=document.getElementById("fill-card");
  c.innerHTML=`<h4>${s.title}</h4>`;
  s.questions.forEach((q,i)=>{
    const d=document.createElement("div");
    d.className="mb-4";
    d.innerHTML=`<label>${i+1}. ${q.title}</label>`;
    if(q.type==="radio")q.options.forEach(o=>{
      d.innerHTML+=`<div class="form-check"><input class="form-check-input" type="radio" name=q${i} value="${o}"><label>${o}</label></div>`;
    });
    else if(q.type==="checkbox")q.options.forEach(o=>{
      d.innerHTML+=`<div class="form-check"><input class="form-check-input" type="checkbox" name=q${i} value="${o}"><label>${o}</label></div>`;
    });
    else d.innerHTML+=`<textarea class="form-control" name=q${i} rows=2></textarea>`;
    c.appendChild(d);
  });
  const b=document.createElement("button");
  b.className="btn btn-primary w-100";
  b.innerText="提交";
  b.onclick=sub;
  c.appendChild(b);
}

function sub(){
  const s=JSON.parse(localStorage.getItem("survey"))||survey;
  const a={time:new Date().toLocaleString(),data:[]};
  let ok=1;
  document.querySelectorAll("#fill-card .mb-4").forEach((it,i)=>{
    const t=s.questions[i].title;
    let v="";
    const r=it.querySelector('input[type="radio"]:checked');
    const c=[...it.querySelectorAll('input[type="checkbox"]:checked')].map(x=>x.value);
    const tx=it.querySelector("textarea");
    if(r)v=r.value;else if(c.length)v=c.join(",");else if(tx)v=tx.value;
    if(!v)ok=0;
    a.data.push({title:t,answer:v});
  });
  if(!ok)return alert("请完成所有题目");
  answers=JSON.parse(localStorage.getItem("answers")||"[]");
  answers.push(a);
  localStorage.setItem("answers",JSON.stringify(answers));
  alert("提交成功！");
  loadFill();
}

// 查看结果相关
function loadResult(){
  answers=JSON.parse(localStorage.getItem("answers")||"[]");
  const l=document.getElementById("result-list");
  l.innerHTML="";
  answers.forEach((a,i)=>{
    const d=document.createElement("div");
    d.className="p-3 border rounded mb-2";
    d.innerHTML=`<h6>第${i+1}份 ${a.time}</h6>`;
    a.data.forEach(x=>{
      d.innerHTML+=`<div class='small text-muted'>${x.title}</div><div>${x.answer}</div>`;
    });
    l.appendChild(d);
  });
}

// 初始化
window.onload=()=>{
  loadFill();
};
</script>
</body>
</html>
