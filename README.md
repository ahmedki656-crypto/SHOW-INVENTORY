<!DOCTYPE html>
<html lang="en" dir="ltr">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Inventory Dresses</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      padding: 20px;
      background: #f9f9f9;
    }
    h2 {
      color: #333;
      text-align: center;
    }
    #searchBox {
      width: 100%;
      max-width: 600px;
      margin: 10px auto 20px;
      display: block;
      padding: 10px;
      font-size: 16px;
      border: 1px solid #ccc;
      border-radius: 5px;
      box-sizing: border-box;
    }
    #dresses-container {
      max-width: 600px;
      margin: 0 auto;
    }
    .dress-item {
      background: white;
      padding: 15px;
      margin-bottom: 10px;
      border-radius: 6px;
      box-shadow: 0 0 8px #ccc;
      font-size: 16px;
      line-height: 1.4;
    }
    .dress-item span.label {
      font-weight: bold;
      color: #555;
      display: inline-block;
      width: 80px;
    }
  </style>
</head>
<body>
  <h2>Available Dresses in Stock</h2>
  <input
    type="text"
    id="searchBox"
    placeholder="Search by Model, Color, or Size..."
    autocomplete="off"
  />
  <div id="dresses-container">Loading dresses...</div>

  <script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/12.1.0/firebase-app.js";
    import {
      getFirestore,
      collection,
      getDocs,
    } from "https://www.gstatic.com/firebasejs/12.1.0/firebase-firestore.js";

    const firebaseConfig = {
      apiKey: "AIzaSyBDVd2SHAmtIbJGlH0lrnPSz-eHfsg6Gyc",
      authDomain: "show-inventory.firebaseapp.com",
      projectId: "show-inventory",
      storageBucket: "show-inventory.appspot.com",
      messagingSenderId: "697364304883",
      appId: "1:697364304883:web:a7b41cc929228976533886",
    };

    const app = initializeApp(firebaseConfig);
    const db = getFirestore(app);

    let dressList = [];

    async function fetchDresses() {
      const dressesCol = collection(db, "dresses");
      const dressSnapshot = await getDocs(dressesCol);
      dressList = dressSnapshot.docs.map((doc) => {
        return { id: doc.id, ...doc.data() };
      });
      displayDresses(dressList);
    }

    function displayDresses(list) {
      const container = document.getElementById("dresses-container");
      container.innerHTML = "";
      if (list.length === 0) {
        container.textContent = "No dresses found.";
        return;
      }
      list.forEach((dress) => {
        const div = document.createElement("div");
        div.className = "dress-item";
        div.innerHTML = `
          <div><span class="label">Model:</span> ${dress.model}</div>
          <div><span class="label">Color:</span> ${dress.color}</div>
          <div><span class="label">Size:</span> ${dress.size}</div>
          <div><span class="label">Barcode:</span> ${dress.id}</div>
        `;
        container.appendChild(div);
      });
    }

    document.getElementById("searchBox").addEventListener("input", (e) => {
      const query = e.target.value.trim().toLowerCase();
      if (!query) {
        displayDresses(dressList);
        return;
      }
      const filtered = dressList.filter(
        (d) =>
          d.model.toLowerCase().includes(query) ||
          d.color.toLowerCase().includes(query) ||
          d.size.toLowerCase().includes(query)
      );
      displayDresses(filtered);
    });

    window.onload = () => {
      fetchDresses().catch((e) => {
        const container = document.getElementById("dresses-container");
        container.textContent = "Error loading dresses: " + e.message;
        console.error(e);
      });
    };
  </script>
</body>
</html>

