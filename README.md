<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>

  <title>Video Archive</title>

  <style>

    *{
      margin:0;
      padding:0;
      box-sizing:border-box;
      font-family:sans-serif;
    }

    body{
      background:#F7FAFD;
      padding:16px;
      color:#2D3A45;
    }

    .container{
      max-width:480px;
      margin:0 auto;
    }

    .title{
      text-align:center;
      font-size:18px;
      margin-bottom:18px;
      color:#6C9FCF;
      letter-spacing:1px;
    }

    .search{
      width:100%;
      padding:12px;
      border:1px solid #D9E7F5;
      border-radius:12px;
      outline:none;
      margin-bottom:14px;
      font-size:14px;
      background:white;
    }

    .search:focus{
      border-color:#9EC4E8;
    }

    .input-box{
      display:flex;
      flex-direction:column;
      gap:8px;
      margin-bottom:18px;
    }

    .input-box input{
      padding:12px;
      border-radius:12px;
      border:1px solid #D9E7F5;
      outline:none;
      background:white;
      font-size:14px;
    }

    .input-box input:focus{
      border-color:#9EC4E8;
    }

    .add-btn{
      padding:12px;
      border:none;
      border-radius:12px;
      background:#9EC4E8;
      color:white;
      cursor:pointer;
      font-size:14px;
      transition:0.2s;
    }

    .add-btn:hover{
      opacity:0.9;
    }

    .item{
      background:white;
      border:1px solid #D9E7F5;
      border-radius:14px;
      padding:14px;
      margin-bottom:10px;

      display:flex;
      justify-content:space-between;
      align-items:center;
      gap:10px;
    }

    .item-title{
      flex:1;
      overflow:hidden;
      text-overflow:ellipsis;
      white-space:nowrap;
      font-size:14px;
      color:#3D4D5C;
    }

    .button-group{
      display:flex;
      gap:6px;
    }

    .open-btn{
      border:none;
      background:#9EC4E8;
      color:white;
      padding:8px 12px;
      border-radius:10px;
      cursor:pointer;
      font-size:12px;
    }

    .delete-btn{
      border:none;
      background:#EDF5FC;
      color:#6D8DAD;
      padding:8px 10px;
      border-radius:10px;
      cursor:pointer;
      font-size:12px;
    }

    .empty{
      text-align:center;
      margin-top:30px;
      color:#9AA9B8;
      font-size:14px;
    }

  </style>
</head>

<body>

  <div class="container">

    <h1 class="title">VIDEO ARCHIVE</h1>

    <input
      type="text"
      id="searchInput"
      class="search"
      placeholder="Search..."
    />

    <div class="input-box">

      <input
        type="text"
        id="titleInput"
        placeholder="Title"
      />

      <input
        type="text"
        id="linkInput"
        placeholder="Notion Link"
      />

      <button
        id="addButton"
        class="add-btn"
      >
        ADD
      </button>

    </div>

    <div id="list"></div>

  </div>


<script type="module">

import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";

import {
  getFirestore,
  collection,
  addDoc,
  onSnapshot,
  deleteDoc,
  doc,
  query,
  orderBy
} from "https://www.gstatic.com/firebasejs/10.12.2/firebase-firestore.js";



// 🔥 Firebase
const firebaseConfig = {
  apiKey: "AIzaSyB0lsvFnLW5TQMHYgQa-XOugHp4L4Yv49o",
  authDomain: "idol-archive.firebaseapp.com",
  projectId: "idol-archive",
  storageBucket: "idol-archive.firebasestorage.app",
  messagingSenderId: "1044257923635",
  appId: "1:1044257923635:web:aa476a3f2363109255f946"
};


const app = initializeApp(firebaseConfig);
const db = getFirestore(app);


// 🌟 archive 분리
const params = new URLSearchParams(window.location.search);

const archiveName =
  params.get("archive") || "default";


// 📦 Firebase 저장 위치
const archiveRef = collection(
  db,
  "archives",
  archiveName,
  "items"
);


// 📱 HTML 요소
const titleInput =
  document.getElementById("titleInput");

const linkInput =
  document.getElementById("linkInput");

const addButton =
  document.getElementById("addButton");

const list =
  document.getElementById("list");

const searchInput =
  document.getElementById("searchInput");


// 📚 전체 데이터 저장용
let allItems = [];


// ➕ 추가 버튼
addButton.addEventListener(
  "click",
  async () => {

    const title =
      titleInput.value.trim();

    const link =
      linkInput.value.trim();

    if(!title || !link){
      alert("Please enter title and link!");
      return;
    }

    await addDoc(archiveRef,{
      title,
      link,
      createdAt:Date.now()
    });

    titleInput.value = "";
    linkInput.value = "";

  }
);


// 🔄 실시간 동기화
const q = query(
  archiveRef,
  orderBy("createdAt","desc")
);

onSnapshot(q,(snapshot)=>{

  allItems = [];

  snapshot.forEach((item)=>{

    allItems.push({
      id:item.id,
      ...item.data()
    });

  });

  renderItems(allItems);

});


// 🔍 검색
searchInput.addEventListener(
  "input",
  ()=>{

    const keyword =
      searchInput.value.toLowerCase();

    const filtered =
      allItems.filter(item =>
        item.title
          .toLowerCase()
          .includes(keyword)
      );

    renderItems(filtered);

  }
);


// 🎨 화면 출력
function renderItems(items){

  list.innerHTML = "";

  if(items.length === 0){

    list.innerHTML = `
      <div class="empty">
        No Archive Yet
      </div>
    `;

    return;
  }

  items.forEach(item=>{

    const div =
      document.createElement("div");

    div.className = "item";

    div.innerHTML = `

      <div class="item-title">
        ${item.title}
      </div>

      <div class="button-group">

        <button
          class="open-btn"
          data-link="${item.link}"
        >
          OPEN
        </button>

        <button
          class="delete-btn"
          data-id="${item.id}"
        >
          DEL
        </button>

      </div>

    `;

    list.appendChild(div);

  });


  // OPEN 버튼
  document
    .querySelectorAll(".open-btn")
    .forEach(btn=>{

      btn.addEventListener(
        "click",
        ()=>{

          window.open(
            btn.dataset.link,
            "_blank"
          );

        }
      );

    });


  // DELETE 버튼
  document
    .querySelectorAll(".delete-btn")
    .forEach(btn=>{

      btn.addEventListener(
        "click",
        async ()=>{

          const check =
            confirm("Delete this archive?");

          if(!check) return;

          await deleteDoc(
            doc(
              db,
              "archives",
              archiveName,
              "items",
              btn.dataset.id
            )
          );

        }
      );

    });

}

</script>

</body>
</html>
