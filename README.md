<!DOCTYPE html>
<html lang="ur">
<head>
  <meta charset="UTF-8">
  <title>WhatsApp Status Panel (Offline)</title>
  <link rel="stylesheet" href="test.css">
  <style>
    body { font-family: Arial; direction: rtl; background: #f5f5f5; padding: 20px; }
    .total-box { font-size: 18px; font-weight: bold; margin-bottom: 20px; }
    .card { background: white; padding: 15px; margin-bottom: 15px; border-radius: 10px; box-shadow: 0 2px 6px rgba(0,0,0,0.1); }
    .top-row { display: flex; justify-content: space-between; align-items: center; }
    .badge { background: #009688; color: white; padding: 5px 10px; border-radius: 5px; }
    .status-tag { padding: 3px 8px; border-radius: 5px; }
    .status-approved { background: green; color: white; }
    .status-pending { background: orange; color: white; }
    .get-btn, .add-btn, .how-btn { padding: 10px 15px; margin: 5px; border: none; background: #007BFF; color: white; border-radius: 5px; cursor: pointer; }
    .history-entry { font-size: 14px; margin-top: 10px; }
  </style>
</head>
<body>
  <div class="total-box" id="totalAmount">Total Amount: Rs. 0.00</div>
  <div id="cardsContainer"></div>
  <button class="add-btn" onclick="addWhatsapp()">Add WhatsApp</button>
  <button class="how-btn" onclick="howToApproved()">How To Approved WhatsApp Number</button>
  <button class="how-btn" onclick="window.open('https://wa.me/923333748210?text=Assalamualaikum Sir Main Whatsapp Se Earn Karna Chahta Hu Pls Help')">Contact Admin</button>

  <script>
    // LocalStorage Key
    const DATA_KEY = "whatsapp_offline_data";

    // Load data from localStorage
    function getData() {
      return JSON.parse(localStorage.getItem(DATA_KEY)) || {};
    }

    // Save data to localStorage
    function saveData(data) {
      localStorage.setItem(DATA_KEY, JSON.stringify(data));
    }

    // Normalize number
    function normalizeNumber(number) {
      number = number.replace(/\D/g, "");
      if (number.startsWith("0")) return "92" + number.slice(1);
      if (number.startsWith("3")) return "92" + number;
      if (number.startsWith("+")) return number.slice(1);
      if (number.startsWith("0092")) return number.replace("0092", "92");
      return number;
    }

    // Add WhatsApp number
    function addWhatsapp() {
      let number = prompt("Enter WhatsApp Number:");
      if (!number) return;
      number = normalizeNumber(number);
      if (!/^923\d{9}$/.test(number)) return alert("Invalid number format");

      let data = getData();
      if (!data.numbers) data.numbers = {};
      if (!data.numbers[number]) {
        data.numbers[number] = { status: "Pending", points: 0, payments: [] };
        saveData(data);
        loadNumbers();
      } else {
        alert("Number already exists");
      }
    }

    // Show How-To Guide
    function howToApproved() {
      alert("اپنا WhatsApp نمبر approved کرنے کے لیے آپ WhatsApp Web کے ذریعے لگائیں، اور admin سے رابطہ کریں");
    }

    // Load numbers from storage
    function loadNumbers() {
      const data = getData();
      const container = document.getElementById("cardsContainer");
      container.innerHTML = "";
      let totalPoints = 0;

      if (!data.numbers) return;

      for (let number in data.numbers) {
        const info = data.numbers[number];
        const status = info.status || "Pending";
        const points = parseFloat(info.points) || 0;
        totalPoints += points;
        const statusClass = status === "Approved" ? "status-approved" : "status-pending";

        const card = document.createElement("div");
        card.className = "card";
        card.innerHTML = `
          <div class="top-row">
            <div><div class="number">${number}</div></div>
            <div style="text-align: right;">
              <div class="badge">${points.toFixed(2)}</div>
              <span class="status-tag ${statusClass}">${status}</span><br><br>
              <button class="get-btn" onclick="getAmount('${number}', ${points})">GET</button>
            </div>
          </div>
          <div class="history" id="history-${number}">${getHistoryHTML(info.payments)}</div>
        `;
        container.appendChild(card);
      }

      document.getElementById("totalAmount").innerText = "Total Amount: Rs. " + totalPoints.toFixed(2);
    }

    // Generate history HTML
    function getHistoryHTML(payments = []) {
      if (!payments.length) return "No payment history yet.";

      let html = "";
      for (let entry of payments.reverse()) {
        html += `
          <div class="history-entry">
            Amount: Rs. ${(entry.amount || 0).toFixed(2)} <br>
            Name: ${entry.account_name}<br>
            Number: ${entry.account_number}<br>
            Time: ${entry.time}<br>
            <span class="history-status" style="color:${entry.status === 'Approved' ? 'green' : 'red'}">
              ${entry.status}
            </span>
          </div>
        `;
      }
      return html;
    }

    // Request payment
    function getAmount(number, points) {
      if (points < 50) return alert("آپ کا amount کم ہے، براہ کرم Rs. 50 مکمل کریں");

      const name = prompt("JazzCash/EasyPaisa نام:");
      if (!name) return alert("نام ضروری ہے");

      const accNumber = prompt("اکاؤنٹ نمبر:");
      if (!accNumber || isNaN(accNumber)) return alert("اکاؤنٹ نمبر درست نہیں");

      const data = getData();
      const payments = data.numbers[number].payments || [];
      payments.push({
        account_name: name,
        account_number: accNumber,
        amount: 50,
        time: new Date().toLocaleString(),
        status: "Pending"
      });

      data.numbers[number].points -= 50;
      data.numbers[number].payments = payments;
      saveData(data);
      loadNumbers();
    }

    // Load on start
    window.onload = loadNumbers;
  </script>
</body>
</html>
