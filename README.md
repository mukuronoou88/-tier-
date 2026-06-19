<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>2択ランキングメーカー</title>

<style>
:root{
    --bg:#121212;
    --card:#1e1e1e;
    --button:#2d2d2d;
    --text:#ffffff;
    --accent:#4caf50;
}

*{
    box-sizing:border-box;
}

body{
    margin:0;
    padding:20px;
    background:var(--bg);
    color:var(--text);
    font-family:sans-serif;
}

.container{
    max-width:600px;
    margin:auto;
}

h1,h2,p{
    text-align:center;
}

textarea{
    width:100%;
    height:220px;
    padding:12px;
    font-size:16px;
    border-radius:12px;
    border:1px solid #444;
    background:var(--card);
    color:var(--text);
}

button{
    width:100%;
    padding:16px;
    margin-top:12px;
    border:none;
    border-radius:12px;
    background:var(--button);
    color:white;
    font-size:18px;
    cursor:pointer;
}

button:hover{
    opacity:0.9;
}

.choice{
    min-height:70px;
}

.bar{
    width:100%;
    height:18px;
    background:#333;
    border-radius:999px;
    overflow:hidden;
    margin-bottom:16px;
}

#progressBar{
    width:0%;
    height:100%;
    background:var(--accent);
    transition:0.3s;
}

.card{
    background:var(--card);
    padding:20px;
    border-radius:16px;
}

#compare,
#result{
    display:none;
}

ol{
    text-align:left;
    padding-left:24px;
}

li{
    margin-bottom:10px;
    font-size:18px;
}

.small{
    opacity:0.8;
    font-size:14px;
    text-align:center;
}
</style>
</head>
<body>

<div class="container">

    <div id="input" class="card">

        <h1>2択ランキングメーカー</h1>

        <p>1行につき1項目入力してください</p>

        <textarea id="items" placeholder="りんご
みかん
バナナ
ぶどう"></textarea>

        <button onclick="startRanking()">
            ランキング開始
        </button>

    </div>

    <div id="compare" class="card">

        <div class="bar">
            <div id="progressBar"></div>
        </div>

        <p id="progressText"></p>

        <h2>どちらが上位？</h2>

        <button class="choice" id="leftBtn"></button>

        <button class="choice" id="rightBtn"></button>

        <p class="small">
            途中で閉じても自動保存されます
        </p>

    </div>

    <div id="result" class="card">

        <h2>ランキング結果</h2>

        <ol id="rankingList"></ol>

        <button onclick="resetApp()">
            もう一度作成
        </button>

    </div>

</div>

<script>

let ranking = [];
let remaining = [];

let currentItem = null;

let low = 0;
let high = 0;
let mid = 0;

let totalItems = 0;
let insertedCount = 0;

const SAVE_KEY = "rankingMakerSave";

function saveState(){

    const data = {
        ranking,
        remaining,
        currentItem,
        low,
        high,
        totalItems,
        insertedCount
    };

    localStorage.setItem(
        SAVE_KEY,
        JSON.stringify(data)
    );
}

function loadState(){

    const raw =
        localStorage.getItem(SAVE_KEY);

    if(!raw) return false;

    try{

        const data = JSON.parse(raw);

        ranking = data.ranking || [];
        remaining = data.remaining || [];
        currentItem = data.currentItem;
        low = data.low;
        high = data.high;
        totalItems = data.totalItems;
        insertedCount = data.insertedCount;

        return true;

    }catch(e){

        return false;
    }
}

function clearSave(){

    localStorage.removeItem(SAVE_KEY);
}

function startRanking(){

    const items = document
        .getElementById("items")
        .value
        .split("\n")
        .map(v => v.trim())
        .filter(v => v);

    if(items.length < 2){

        alert("2個以上入力してください");
        return;
    }

    ranking = [items[0]];
    remaining = items.slice(1);

    totalItems = items.length;
    insertedCount = 1;

    nextItem();
}

function nextItem(){

    if(remaining.length === 0){

        showResult();
        return;
    }

    currentItem = remaining.shift();

    low = 0;
    high = ranking.length;

    saveState();

    askComparison();
}

function askComparison(){

    if(low >= high){

        ranking.splice(
            low,
            0,
            currentItem
        );

        insertedCount++;

        saveState();

        nextItem();

        return;
    }

    mid = Math.floor(
        (low + high) / 2
    );

    document.getElementById("input").style.display =
        "none";

    document.getElementById("result").style.display =
        "none";

    document.getElementById("compare").style.display =
        "block";

    updateProgress();

    document.getElementById("leftBtn")
        .textContent = currentItem;

    document.getElementById("rightBtn")
        .textContent = ranking[mid];
}

function chooseCurrent(){

    high = mid;

    saveState();

    askComparison();
}

function chooseExisting(){

    low = mid + 1;

    saveState();

    askComparison();
}

function updateProgress(){

    const percent =
        (insertedCount / totalItems) * 100;

    document.getElementById(
        "progressBar"
    ).style.width =
        percent + "%";

    document.getElementById(
        "progressText"
    ).textContent =
        `${insertedCount} / ${totalItems} 完了`;
}

function showResult(){

    clearSave();

    document.getElementById("compare").style.display =
        "none";

    document.getElementById("result").style.display =
        "block";

    const list =
        document.getElementById(
            "rankingList"
        );

    list.innerHTML = "";

    ranking.forEach(item => {

        const li =
            document.createElement("li");

        li.textContent = item;

        list.appendChild(li);

    });
}

function resetApp(){

    clearSave();

    location.reload();
}

document.getElementById(
    "leftBtn"
).addEventListener(
    "click",
    chooseCurrent
);

document.getElementById(
    "rightBtn"
).addEventListener(
    "click",
    chooseExisting
);

window.addEventListener(
    "load",
    () => {

        if(loadState()){

            const resume =
                confirm(
                    "保存されたランキングがあります。続きから再開しますか？"
                );

            if(resume){

                document.getElementById(
                    "input"
                ).style.display =
                    "none";

                askComparison();

            }else{

                clearSave();
            }
        }
    }
);

</script>

</body>
</html>
