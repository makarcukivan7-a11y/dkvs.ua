<!-- Замініть лише функцію submitForm у вашому index.html на цю: -->
<script>
  // Після деплою Apps Script отримаєш WEB_APP_URL — вставити сюди
  const APPS_SCRIPT_URL = "https://script.google.com/macros/s/ВАШ_СКРИПТ_ID/exec"; // <-- заміни на свій

  async function submitForm(e){
    e.preventDefault();
    const status = document.getElementById('formStatus');
    const data = {
      fullname: document.getElementById('fullname').value.trim(),
      nick: document.getElementById('inGame').value.trim(),
      email: document.getElementById('email').value.trim(),
      unit: document.getElementById('unit').value,
      motivation: document.getElementById('motivation').value.trim(),
      date: new Date().toISOString()
    };
    if(!data.fullname || !data.nick || !data.email){
      status.textContent = 'Будь ласка, заповніть обовʼязкові поля.';
      return;
    }
    try{
      // POST JSON до Apps Script
      const resp = await fetch(APPS_SCRIPT_URL, {
        method: 'POST',
        mode: 'cors',
        headers: {'Content-Type':'application/json'},
        body: JSON.stringify(data)
      });
      const j = await resp.json();
      if(resp.ok && j.result === 'success'){
        status.textContent = 'Дякуємо — заявка надіслана. Ми звʼяжемося в Discord або на форумі.';
        document.getElementById('joinForm').reset();
      } else {
        status.textContent = 'Помилка відправлення. Спробуйте пізніше.';
        console.error('AppsScript error', j);
      }
    }catch(err){
      console.error(err);
      status.textContent = 'Не вдалося відправити заявку — перевірте підключення.';
    }
    setTimeout(()=> status.textContent = '', 7000);
  }
</script>
[
  {
    "id": 1,
    "title": "Старт набору до ДКВС — осінь 2025",
    "date": "2025-10-20",
    "summary": "Оголошується набір нових співробітників до Оперативного відділу та НАК.",
    "content": "<p>Подробиці набору: ...</p>"
  },
  {
    "id": 2,
    "title": "Наказ №12 — зміни в процедурах",
    "date": "2025-09-01",
    "summary": "Оновлено процедури внутрішнього контролю.",
    "content": "<p>Пункти наказу: ...</p>"
  }
]
<!doctype html>
<html lang="uk">
<head>
<meta charset="utf-8"/>
<meta name="viewport" content="width=device-width,initial-scale=1"/>
<title>Новини — ДКВС</title>
<style>
  body{font-family:Inter,system-ui; background:#051026;color:#eaf3fb;margin:0;padding:24px}
  .container{max-width:1000px;margin:0 auto}
  h1{margin-top:6px}
  .grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(260px,1fr));gap:16px;margin-top:18px}
  .card{background:#0e1724;padding:16px;border-radius:10px;border:1px solid rgba(255,255,255,0.03)}
  .card h3{margin:0 0 8px}
  .muted{color:#98a3ad;font-size:13px}
  a{color:inherit}
</style>
</head>
<body>
  <div class="container">
    <a href="index.html">← На головну</a>
    <h1>Новини ДКВС</h1>
    <div id="news" class="grid"></div>
  </div>

<script>
  async function loadNews(){
    try{
      const resp = await fetch('/news.json', {cache: "no-store"});
      const data = await resp.json();
      const wrap = document.getElementById('news');
      if(!data || !data.length){ wrap.innerHTML = '<div class="muted">Новин немає</div>'; return; }
      wrap.innerHTML = data.map(n => `
        <div class="card">
          <h3>${n.title}</h3>
          <div class="muted">${n.date}</div>
          <p style="color:#cfe7f5">${n.summary}</p>
          <div><a href="news_item_${n.id}.html">Детальніше →</a></div>
        </div>
      `).join('');
    }catch(err){
      console.error(err);
      document.getElementById('news').innerHTML = '<div class="muted">Помилка завантаження новин.</div>';
    }
  }
  loadNews();
</script>
</body>
</html>
// Apps Script: Code.gs
const SHEET_ID = "ВАШ_SHEET_ID"; // <-- вставити ID таблиці (із URL)
function doPost(e){
  try{
    const ss = SpreadsheetApp.openById(SHEET_ID);
    const sheet = ss.getSheetByName('Sheet1') || ss.getSheets()[0];
    let payload;
    if(e.postData && e.postData.type === "application/json"){
      payload = JSON.parse(e.postData.contents);
    } else if(e.parameter){
      payload = e.parameter;
    } else {
      payload = {};
    }
    const ts = new Date();
    sheet.appendRow([
      ts,
      payload.fullname || '',
      payload.nick || '',
      payload.email || '',
      payload.unit || '',
      payload.motivation || '',
      JSON.stringify(payload)
    ]);
    return ContentService.createTextOutput(JSON.stringify({result:'success'})).setMimeType(ContentService.MimeType.JSON);
  }catch(err){
    Logger.log(err);
    return ContentService.createTextOutput(JSON.stringify({result:'error', message: err.toString()})).setMimeType(ContentService.MimeType.JSON);
  }
}
function doOptions(e){
  return ContentService.createTextOutput('').setMimeType(ContentService.MimeType.TEXT);
}
name: Deploy to GitHub Pages
on:
  push:
    branches:
      - main

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # На випадок, якщо потрібна збірка (наприклад, npm). Якщо не потрібно — цей крок лишається, але не використовуємо
      # - name: Setup Node
      #   uses: actions/setup-node@v4
      #   with:
      #     node-version: 18

      - name: Upload artifact for GitHub Pages
        uses: actions/upload-pages-artifact@v1
        with:
          path: ./   # віддаємо весь репо як артефакт для Pages

      - name: Deploy to GitHub Pages
        uses: actions/deploy-pages@v1
// після успішного додавання рядка
const webhookUrl = "https://discord.com/api/webhooks/ВАШ_WEBHOOK_ID/ТОКЕН";
const payloadForDiscord = {
  content: `Нова заявка: ${payload.fullname} (${payload.nick})\nEmail: ${payload.email}\nПідрозділ: ${payload.unit}`
};
UrlFetchApp.fetch(webhookUrl, {
  method: 'post',
  contentType: 'application/json',
  payload: JSON.stringify(payloadForDiscord)
});
cd path/to/dkvs_site
git init
git branch -M main
git remote add origin https://github.com/ТВОЙ_ЮЗЕР/dkvs-ukraine-gta.git
git add .
git commit -m "Initial site for DKVS (Ukraine GTA)"
git push -u origin main
const SHEET_ID = "ВАШ_SHEET_ID"; // <- вставити

function doPost(e){
  try{
    const ss = SpreadsheetApp.openById(SHEET_ID);
    const sheet = ss.getSheets()[0];
    let payload = {};
    if(e.postData && e.postData.type === "application/json"){
      payload = JSON.parse(e.postData.contents);
    } else if(e.parameter){
      payload = e.parameter;
    }
    const ts = new Date();
    sheet.appendRow([
      ts,
      payload.fullname || '',
      payload.nick || '',
      payload.email || '',
      payload.unit || '',
      payload.motivation || '',
      JSON.stringify(payload)
    ]);
    return ContentService.createTextOutput(JSON.stringify({result:'success'})).setMimeType(ContentService.MimeType.JSON);
  }catch(err){
    return ContentService.createTextOutput(JSON.stringify({result:'error',message:err.toString()})).setMimeType(ContentService.MimeType.JSON);
  }
}
const webhookUrl = "https://discord.com/api/webhooks/ВАШ_ID/ВАШ_ТОКЕН";
const payloadForDiscord = {
  content: `Нова заявка: ${payload.fullname} (${payload.nick})\nEmail: ${payload.email}\nПідрозділ: ${payload.unit}`
};
UrlFetchApp.fetch(webhookUrl, {
  method: 'post',
  contentType: 'application/json',
  payload: JSON.stringify(payloadForDiscord)
});
