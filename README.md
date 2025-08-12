<!DOCTYPE html>
<html lang="en" dir="ltr">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Inventory Dresses</title>
  <style>
    /* تصميم عام للصفحة */
    body {
      font-family: Arial, sans-serif;
      padding: 20px;
      background-color: #f9f9f9;
      margin: 0;
    }

    /* عنوان الصفحة */
    h2 {
      text-align: center;
      color: #333;
      margin-bottom: 15px;
    }

    /* صندوق البحث */
    #searchBox {
      display: block;
      width: 100%;
      max-width: 600px;
      margin: 0 auto 25px auto;
      padding: 12px 15px;
      font-size: 16px;
      border: 1px solid #ccc;
      border-radius: 6px;
      box-sizing: border-box;
      transition: border-color 0.3s ease;
    }
    #searchBox:focus {
      border-color: #007BFF;
      outline: none;
      box-shadow: 0 0 5px rgba(0, 123, 255, 0.5);
    }

    /* حاوية الفساتين */
    #dresses-container {
      max-width: 600px;
      margin: 0 auto;
      min-height: 200px;
    }

    /* كل عنصر فستان */
    .dress-item {
      background-color: #fff;
      border-radius: 8px;
      padding: 18px 20px;
      margin-bottom: 15px;
      box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
      font-size: 17px;
      color: #444;
      line-height: 1.5;
      transition: background-color 0.2s ease;
    }
    .dress-item:hover {
      background-color: #e9f1ff;
    }

    /* تسميات داخل عناصر الفساتين */
    .dress-item span.label {
      font-weight: 700;
      color: #555;
      display: inline-block;
      width: 90px;
    }

    /* رسالة تحميل البيانات أو الأخطاء */
    #dresses-container.loading,
    #dresses-container.error {
      text-align: center;
      font-style: italic;
      color: #777;
      margin-top: 50px;
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

  <div id="dresses-container" class="loading">
    Loading dresses...
  </div>

  <script type="module">
    /* استيراد مكتبات Firebase الأساسية و Firestore */
    import { initializeApp } from "https://www.gstatic.com/firebasejs/12.1.0/firebase-app.js";
    import {
      getFirestore,
      collection,
      getDocs,
    } from "https://www.gstatic.com/firebasejs/12.1.0/firebase-firestore.js";

    /* إعدادات Firebase الخاصة بك */
    const firebaseConfig = {
      apiKey: "AIzaSyBDVd2SHAmtIbJGlH0lrnPSz-eHfsg6Gyc",
      authDomain: "show-inventory.firebaseapp.com",
      projectId: "show-inventory",
      storageBucket: "show-inventory.appspot.com",
      messagingSenderId: "697364304883",
      appId: "1:697364304883:web:a7b41cc929228976533886",
    };

    /* تهيئة التطبيق وقاعدة البيانات */
    const app = initializeApp(firebaseConfig);
    const db = getFirestore(app);

    /* قائمة لحفظ بيانات الفساتين محلياً للبحث والتصفية */
    let dressList = [];

    /* دالة لتحميل بيانات الفساتين من مجموعة "dresses" في Firestore */
    async function fetchDresses() {
      const container = document.getElementById("dresses-container");
      container.classList.remove("error");
      container.classList.add("loading");
      container.textContent = "Loading dresses...";

      try {
        const dressesCol = collection(db, "dresses");
        const dressSnapshot = await getDocs(dressesCol);

        if (dressSnapshot.empty) {
          container.textContent = "No dresses found in stock.";
          container.classList.remove("loading");
          return;
        }

        /* تحويل كل مستند إلى كائن مع حفظ المعرف (الباركود) */
        dressList = dressSnapshot.docs.map((doc) => ({
          id: doc.id,
          ...doc.data(),
        }));

        /* عرض القائمة كاملة */
        displayDresses(dressList);
      } catch (error) {
        container.textContent = "Error loading dresses: " + error.message;
        container.classList.add("error");
        console.error("Firestore loading error:", error);
      } finally {
        container.classList.remove("loading");
      }
    }

    /* دالة عرض الفساتين في صفحة الويب */
    function displayDresses(list) {
      const container = document.getElementById("dresses-container");
      container.innerHTML = "";

      if (list.length === 0) {
        container.textContent = "No dresses found matching your search.";
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

    /* إضافة مستمع لحدث الكتابة في مربع البحث لتصفية النتائج */
    document.getElementById("searchBox").addEventListener("input", (e) => {
      const query = e.target.value.trim().toLowerCase();

      if (!query) {
        // إذا البحث فارغ، عرض القائمة الكاملة
        displayDresses(dressList);
        return;
      }

      // تصفية الفساتين بناءً على البحث في الموديل أو اللون أو المقاس
      const filtered = dressList.filter(
        (dress) =>
          dress.model.toLowerCase().includes(query) ||
          dress.color.toLowerCase().includes(query) ||
          dress.size.toLowerCase().includes(query)
      );

      displayDresses(filtered);
    });

    /* تحميل الفساتين عند تحميل الصفحة */
    window.onload = () => {
      fetchDresses();
    };
  </script>

</body>
</html>
