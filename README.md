<!DOCTYPE html>
<html lang="ar">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>تحديث المخزون</title>

  <!-- مكتبة Chart.js -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

  <!-- تنسيق CSS -->
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      margin: 0;
      padding: 0;
      background-color: #F9F9F9;  /* خلفية فاتحة للأزهار */
      color: #333;  /* لون النص الأساسي */
    }

    h1 {
      background-color: #ADD8E6;  /* أزرق فاتح لعناوين الأقسام */
      color: white;
      padding: 20px;
      margin: 0;
    }

    .container {
      display: flex;
      justify-content: space-around;
      align-items: flex-start;
      gap: 40px;
      margin: 40px 0;
      flex-wrap: wrap;
    }

    table {
      border-collapse: collapse;
      width: 400px;
      margin: 20px auto;
      background-color: white;
      border-radius: 10px;
      box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    }

    table, th, td {
      border: 1px solid #ddd;
      text-align: center;
    }

    th, td {
      padding: 10px;
    }

    th {
      background-color: #ADD8E6;  /* أزرق فاتح لعناوين الجداول */
      color: white;
    }

    canvas {
      width: 300px !important;
      height: 300px !important;
      margin-top: 20px;
      border: 1px solid #ddd;
    }

    .update-form {
      margin-top: 20px;
    }

    .update-form input[type="number"] {
      width: 80px;
      padding: 5px;
      background-color: #FAFAD2;  /* أصفر فاتح لخلفية الحقول */
    }

    .update-form button {
      padding: 5px 10px;
      margin-left: 10px;
      cursor: pointer;
      background-color: #ADD8E6;  /* أزرق فاتح للأزرار */
      color: white;
      border: none;
      border-radius: 5px;
    }

    .update-form button:hover {
      background-color: #87CEEB;  /* أزرق سماوي عند التمرير */
    }

    .container div {
      width: 100%;
      max-width: 400px;
      margin: 0 auto;
    }
  </style>
  
  <script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/11.0.2/firebase-app.js";
    import { getFirestore, collection, getDocs, doc, updateDoc } from "https://www.gstatic.com/firebasejs/11.0.2/firebase-firestore.js";

    const firebaseConfig = {
      apiKey: "AIzaSyC-hgcKLED99nXYNCso9I5IUWwr4sG_ufE",
      authDomain: "chocola-shop-274e3.firebaseapp.com",
      databaseURL: "https://chocola-shop-274e3-default-rtdb.firebaseio.com",
      projectId: "chocola-shop-274e3",
      storageBucket: "chocola-shop-274e3.appspot.com",
      messagingSenderId: "723756715672",
      appId: "1:723756715672:web:13140a3d0221846adb0aed",
      measurementId: "G-4J61KLNMPC"
    };

    const app = initializeApp(firebaseConfig);
    const db = getFirestore(app);

    const flowerLabels = [];
    const flowerSoldData = [];
    const flowerStockData = [];

    const chocolateLabels = [];
    const chocolateSoldData = [];
    const chocolateStockData = [];

    const fetchData = async () => {
      try {
        const flowerSnapshot = await getDocs(collection(db, "flower"));
        const flowerTable = document.getElementById("flower-table-body");
        flowerSnapshot.forEach((doc) => {
          const data = doc.data();
          flowerLabels.push(doc.id);
          flowerSoldData.push(data.sold);
          flowerStockData.push(data.stock);

          const row = flowerTable.insertRow();
          row.innerHTML = `
            <td>${doc.id}</td>
            <td>${data.sold}</td>
            <td>${data.stock}</td>
            <td>
              <form class="update-form" onsubmit="updateStock(event, '${doc.id}', 'flower')">
                <input type="number" placeholder="كمية جديدة" min="0" required>
                <button type="submit">تحديث</button>
              </form>
            </td>`;
        });

        const chocolateSnapshot = await getDocs(collection(db, "chocolate"));
        const chocolateTable = document.getElementById("chocolate-table-body");
        chocolateSnapshot.forEach((doc) => {
          const data = doc.data();
          chocolateLabels.push(doc.id);
          chocolateSoldData.push(data.sold);
          chocolateStockData.push(data.stock);

          const row = chocolateTable.insertRow();
          row.innerHTML = `
            <td>${doc.id}</td>
            <td>${data.sold}</td>
            <td>${data.stock}</td>
            <td>
              <form class="update-form" onsubmit="updateStock(event, '${doc.id}', 'chocolate')">
                <input type="number" placeholder="كمية جديدة" min="0" required>
                <button type="submit">تحديث</button>
              </form>
            </td>`;
        });

        drawCharts();
      } catch (error) {
        console.error("خطأ أثناء جلب البيانات:", error);
      }
    };

    window.updateStock = async (event, itemId, collectionName) => {
      event.preventDefault();
      const newStock = event.target.querySelector("input").value;
      try {
        const itemDoc = doc(db, collectionName, itemId);
        await updateDoc(itemDoc, { stock: parseInt(newStock) });
        alert(`تم تحديث المخزون لـ "${itemId}" بنجاح!`);
        window.location.reload();
      } catch (error) {
        console.error("خطأ أثناء تحديث المخزون:", error);
        alert("حدث خطأ أثناء التحديث. حاول مرة أخرى.");
      }
    };

    const drawCharts = () => {
      new Chart(document.getElementById("flower-chart"), {
        type: "bar",
        data: {
          labels: flowerLabels,
          datasets: [
            {
              label: "الكمية المباعة",
              data: flowerSoldData,
              backgroundColor: "rgba(173, 216, 230, 0.5)",  /* أزرق فاتح */
              borderColor: "rgba(173, 216, 230, 1)",
            },
            {
              label: "المخزون",
              data: flowerStockData,
              backgroundColor: "rgba(135, 206, 235, 0.5)",  /* أزرق سماوي */
              borderColor: "rgba(135, 206, 235, 1)",
            },
          ],
        },
      });

      new Chart(document.getElementById("flower-pie-chart"), {
        type: "pie",
        data: {
          labels: flowerLabels,
          datasets: [{
            label: "المخزون مقابل الكمية المباعة",
            data: flowerStockData.map((stock, index) => stock + flowerSoldData[index]), 
            backgroundColor: [
  "rgba(135, 206, 235, 0.5)",  /* أزرق سماوي */
  "rgba(173, 216, 230, 0.5)",  /* أزرق فاتح */
  "rgba(100, 149, 237, 0.5)"   /* أزرق ملكي فاتح */
]

          }]
        }
      });

      new Chart(document.getElementById("chocolate-chart"), {
        type: "bar",
        data: {
          labels: chocolateLabels,
          datasets: [
            {
              label: "الكمية المباعة",
              data: chocolateSoldData,
              backgroundColor: [
  "rgba(135, 206, 235, 0.5)",  /* أزرق سماوي */
  "rgba(173, 216, 230, 0.5)",  /* أزرق فاتح */
  "rgba(100, 149, 237, 0.5)"   /* أزرق ملكي فاتح */
]

            },
            {
              label: "المخزون",
              data: chocolateStockData,
              backgroundColor: [
  "rgba(135, 206, 235, 0.5)",  /* أزرق سماوي */
  "rgba(173, 216, 230, 0.5)",  /* أزرق فاتح */
  "rgba(100, 149, 237, 0.5)"   /* أزرق ملكي فاتح */
]

            },
          ],
        },
      });

      new Chart(document.getElementById("chocolate-pie-chart"), {
        type: "pie",
        data: {
          labels: chocolateLabels,
          datasets: [{
            label: "المخزون مقابل الكمية المباعة",
            data: chocolateStockData.map((stock, index) => stock + chocolateSoldData[index]),
            backgroundColor: [
  "rgba(135, 206, 235, 0.5)",  /* أزرق سماوي */
  "rgba(173, 216, 230, 0.5)",  /* أزرق فاتح */
  "rgba(100, 149, 237, 0.5)"   /* أزرق ملكي فاتح */
]

          }]
        }
      });
    };

    fetchData();
  </script>
</head>
<body>
  <h1>تحديث المخزون للأزهار والشوكولاتة</h1>

  <div class="container">
    <div>
      <h2>جدول بيانات الأزهار</h2>
      <table>
        <thead>
          <tr>
            <th>المنتج</th>
            <th>الكمية المباعة</th>
            <th>المخزون</th>
            <th>تحديث المخزون</th>
          </tr>
        </thead>
        <tbody id="flower-table-body"></tbody>
      </table>
      <canvas id="flower-chart"></canvas>
      <canvas id="flower-pie-chart"></canvas>
    </div>

    <div>
      <h2>جدول بيانات الشوكولاتة</h2>
      <table>
        <thead>
          <tr>
            <th>المنتج</th>
            <th>الكمية المباعة</th>
            <th>المخزون</th>
            <th>تحديث المخزون</th>
          </tr>
        </thead>
        <tbody id="chocolate-table-body"></tbody>
      </table>
      <canvas id="chocolate-chart"></canvas>
      <canvas id="chocolate-pie-chart"></canvas>
    </div>
  </div>
</body>
</html>