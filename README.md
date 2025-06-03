<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Harness Check In/Out</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      max-width: 700px;
      margin: auto;
      padding: 2em;
    }
    input, textarea, button {
      width: 100%;
      padding: 0.5em;
      margin-top: 0.5em;
      font-size: 1em;
    }
    .inline-buttons {
      display: flex;
      gap: 1em;
      margin-top: 1em;
    }
    .inline-buttons button {
      flex: 1;
      padding: 0.75em;
      font-weight: bold;
      border: 2px solid #ccc;
      cursor: pointer;
    }
    .highlight-yes {
      background-color: #d4f8d4;
      border-color: green;
    }
    .highlight-no {
      background-color: #f8d4d4;
      border-color: red;
    }
    #reasonBox {
      display: none;
      margin-top: 1em;
    }
    #logList {
      margin-top: 2em;
      padding-top: 1em;
      border-top: 1px solid #ccc;
    }
    li {
      margin-bottom: 0.5em;
    }
  </style>
</head>
<body>

  <h2>Prototype Harness Check In/Out</h2>

  <label>Scan or Enter Part Number</label>
  <input type="text" id="barcode" placeholder="e.g., H123456" autocomplete="off">

  <label>First Name</label>
  <input type="text" id="firstName">

  <label>Last Name</label>
  <input type="text" id="lastName">

  <label>Does this harness have a build paper?</label>
  <div class="inline-buttons">
    <button id="yesBtn" onclick="selectBuildPaper(true)">Yes</button>
    <button id="noBtn" onclick="selectBuildPaper(false)">No</button>
  </div>

  <div id="reasonBox">
    <label>Why no build paper?</label>
    <textarea id="noReason" rows="3" placeholder="Explain why..."></textarea>
  </div>

  <div class="inline-buttons">
    <button onclick="submitData('Check Out')">Check Out</button>
    <button onclick="submitData('Check In')">Check In</button>
  </div>

  <div id="logList">
    <h3>Recent Activity</h3>
    <ul id="log"></ul>
  </div>

<script>
  let hasBuildPaper = null;

  function selectBuildPaper(choice) {
    hasBuildPaper = choice;
    const yesBtn = document.getElementById('yesBtn');
    const noBtn = document.getElementById('noBtn');
    const reasonBox = document.getElementById('reasonBox');

    if (choice) {
      yesBtn.classList.add("highlight-yes");
      noBtn.classList.remove("highlight-no");
      reasonBox.style.display = "none";
    } else {
      noBtn.classList.add("highlight-no");
      yesBtn.classList.remove("highlight-yes");
      reasonBox.style.display = "block";
    }
  }

  function submitData(actionType) {
    const barcode = document.getElementById('barcode').value.trim();
    const firstName = document.getElementById('firstName').value.trim();
    const lastName = document.getElementById('lastName').value.trim();
    const reason = document.getElementById('noReason').value.trim();
    const timestamp = new Date().toLocaleString();

    if (!barcode || !firstName || !lastName || hasBuildPaper === null) {
      alert("Please fill in all required fields and select build paper status.");
      return;
    }

    if (!hasBuildPaper && !reason) {
      alert("Please provide a reason for missing build paper.");
      return;
    }

    const payload = {
      action: actionType,
      partNumber: barcode,
      firstName,
      lastName,
      hasBuildPaper,
      reason,
      date: timestamp
    };

    fetch("https://script.google.com/macros/s/AKfycbz-qoqOoINibATHwLdXLUNST7_PGfgUgR4jvDL3YOCC4kbSPqroqnDoBZcszjkKUcQE7Q/exec", {
      method: "POST",
      body: JSON.stringify(payload),
      headers: { "Content-Type": "application/json" }
    })
    .then(response => response.text())
    .then(() => {
      logAction(payload);
      clearForm();
      alert("Submitted to Google Sheet!");
    })
    .catch(error => {
      console.error("Error:", error);
      alert("Failed to submit.");
    });
  }

  function logAction(data) {
    const entry = `${data.date} — ${data.firstName} ${data.lastName} ${data.action} ${data.partNumber} — Build Paper: ${data.hasBuildPaper ? 'Yes' : 'No'}${data.reason ? ' — Reason: ' + data.reason : ''}`;
    const li = document.createElement('li');
    li.textContent = entry;
    document.getElementById('log').prepend(li);
  }

  function clearForm() {
    document.getElementById('barcode').value = '';
    document.getElementById('firstName').value = '';
    document.getElementById('lastName').value = '';
    document.getElementById('noReason').value = '';
    hasBuildPaper = null;
    document.getElementById('yesBtn').classList.remove("highlight-yes");
    document.getElementById('noBtn').classList.remove("highlight-no");
    document.getElementById('reasonBox').style.display = "none";
  }
</script>

</body>
</html>
