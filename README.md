<!DOCTYPE html>
<html lang="ar">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>إدارة المخزون والمبيعات</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
  body {
    font-family: 'Poppins', sans-serif;
    background: linear-gradient(to bottom, #e0eafc, #cfdef3);
    margin: 0;
    padding: 0;
  }
  h1, h2 {
    text-align: center;
    color: #34495e;
    margin-bottom: 20px;
  }
  .container {
    width: 95%;
    margin: auto;
    padding: 20px;
  }
  .section {
    margin-bottom: 40px;
  }
  input[type="text"] {
    width: 100%;
    padding: 8px 12px;
    margin-bottom: 15px;
    font-size: 1rem;
    border-radius: 5px;
    border: 1px solid #ccc;
    box-sizing: border-box;
  }
  table {
    width: 100%;
    border-collapse: collapse;
    margin-bottom: 20px;
  }
  th, td {
    padding: 10px;
    text-align: center;
    border: 1px solid #ddd;
  }
  th {
    background-color: #34495e;
    color: white;
  }
  td {
    background-color: #f9f9f9;
  }
  .low-stock {
    color: red;
    font-weight: bold;
  }
  button {
    background-color: #34495e;
    color: white;
    border: none;
    padding: 6px 10px;
    margin-left: 5px;
    border-radius: 4px;
    cursor: pointer;
    font-size: 0.9rem;
  }
  button:hover {
    background-color: #2c3e50;
  }
  input[type="number"] {
    width: 70px;
    padding: 5px;
    border-radius: 4px;
    border: 1px solid #ccc;
    font-size: 0.9rem;
    text-align: center;
  }
  /* حاوية الرسوم */
  .charts {
    display: flex;
    flex-wrap: nowrap;
    justify-content: center;
    gap: 15px;
    overflow-x: auto;
    padding: 10px 0;
  }
  canvas {
    flex: 0 0 auto;
    width: 280px !important;
    height: 220px !important;
    border: 1px solid #ddd;
    border-radius: 8px;
    background-color: transparent !important;
    box-shadow: none;
  }
  </style>

<script type="module">
  // استيراد وظائف Firebase
  import { initializeApp } from "https://www.gstatic.com/firebasejs/11.7.1/firebase-app.js";
 // import { getFirestore, collection, onSnapshot, doc, updateDoc, getDoc } from "https://www.gstatic.com/firebasejs/11.7.1/firebase-firestore.js";
//import { getFirestore, collection, onSnapshot, doc, updateDoc, getDoc } from "https://www.gstatic.com/firebasejs/11.7.1/firebase-firestore.js";
import { getFirestore, collection, onSnapshot, doc, updateDoc, getDoc, getDocs } from "https://www.gstatic.com/firebasejs/11.7.1/firebase-firestore.js";

  // إعدادات اتصال Firebase الخاصة بك
  const firebaseConfig = {
    apiKey: "AIzaSyB3IqR9tpf87yWbX55LWsAYRewsjySUpcA",
    authDomain: "chocola-shop-f7b87.firebaseapp.com",
    projectId: "chocola-shop-f7b87",
    storageBucket: "chocola-shop-f7b87.appspot.com",
    messagingSenderId: "991324220722",
    appId: "1:991324220722:web:ce611e60ee29d0d4d8bb2b",
    measurementId: "G-VKNV6C8CTT"
  };

  // تهيئة تطبيق Firebase و Firestore
  const app = initializeApp(firebaseConfig);
  const db = getFirestore(app);

  // متغيرات الرسوم البيانية
  let flowerChart, flowerPieChart, flowerProfitChart;
  let chocolateChart, chocolatePieChart, chocolateProfitChart;

  // دالة لإنشاء الرسوم البيانية عند تحميل الصفحة
  function initializeCharts() {
    flowerChart = new Chart(document.getElementById("flower-chart"), {
      type: "bar",
      data: { labels: [], datasets: [] },
      options: { responsive: true, plugins: { legend: { position: "top" } } }
    });

    flowerPieChart = new Chart(document.getElementById("flower-pie-chart"), {
      type: "pie",
      data: {
        labels: [],
        datasets: [{
          data: [],
          backgroundColor: ["#ff9999", "#ffcc99", "#99ff99", "#66b3ff"]
        }]
      },
      options: { responsive: true }
    });

    flowerProfitChart = new Chart(document.getElementById("flower-profit-chart"), {
      type: "line",
      data: {
        labels: [],
        datasets: [{
          label: "الأرباح (ريال)",
          data: [],
          borderColor: "green",
          fill: false
        }]
      },
      options: { responsive: true, plugins: { legend: { position: "top" } } }
    });

    chocolateChart = new Chart(document.getElementById("chocolate-chart"), {
      type: "bar",
      data: { labels: [], datasets: [] },
      options: { responsive: true, plugins: { legend: { position: "top" } } }
    });

    chocolatePieChart = new Chart(document.getElementById("chocolate-pie-chart"), {
      type: "pie",
      data: {
        labels: [],
        datasets: [{
          data: [],
          backgroundColor: ["#ff9999", "#ffcc99", "#99ff99", "#66b3ff"]
        }]
      },
      options: { responsive: true }
    });

    chocolateProfitChart = new Chart(document.getElementById("chocolate-profit-chart"), {
      type: "line",
      data: {
        labels: [],
        datasets: [{
          label: "الأرباح (ريال)",
          data: [],
          borderColor: "green",
          fill: false
        }]
      },
      options: { responsive: true, plugins: { legend: { position: "top" } } }
    });
  }

  // دالة لتحميل البيانات من مجموعة Firestore وتحديث الجدول والرسوم البيانية
 function loadCollectionData(collectionName, tableBodyId, chart, pieChart, profitChart) {
  const tableBody = document.getElementById(tableBodyId);

  onSnapshot(collection(db, collectionName), (snapshot) => {
    const labels = [];
    const salesData = [];       // عدد الوحدات المباعة
    const salesCountData = [];  // عدد مرات البيع
    const stockData = [];
    const profitData = [];

    tableBody.innerHTML = ""; // مسح محتوى الجدول قبل التحديث

    snapshot.forEach(docSnap => {
      const data = docSnap.data();
      const name = docSnap.id;
      const sold = data.sold || 0;
      const salesCount = data.salesCount || 0;   // قراءة عدد مرات البيع
      const stock = data.stock || 0;
      const price = data.price || 0;
      const profit = sold * price;

      labels.push(name);
      salesData.push(sold);
      salesCountData.push(salesCount);  // نستخدمها إذا حبيت ترسم الرسم البياني بعد
      stockData.push(stock);
      profitData.push(profit);

      const lowStockWarning = stock <= 5 ? `<span class="low-stock">⚠️ مخزون منخفض</span>` : "";

      const row = document.createElement("tr");
row.innerHTML = `
  <td>${name}</td>
  <td>${sold}</td>
  <td>${salesCount}</td>  <!-- أضف هذا العمود -->
  <td>${stock} ${lowStockWarning}</td>
  <td>${price} ريال</td>
  <td>${profit} ريال</td>
  <td>
    <input type="number" id="${collectionName}-${name}-add-sales" placeholder="مبيعات" min="0" />
    <button onclick="addSales('${collectionName}', '${name}')">أضف مبيعات</button><br><br>
    <input type="number" id="${collectionName}-${name}-add-stock" placeholder="مخزون" min="0" />
    <button onclick="addStock('${collectionName}', '${name}')">أضف مخزون</button>
  </td>
`;

        
        tableBody.appendChild(row);
      });

      // تحديث الرسم البياني الشريطي (المبيعات والمخزون)
      chart.data.labels = labels;
      chart.data.datasets = [
        { label: "المبيعات", data: salesData, backgroundColor: "orange" },
        { label: "المخزون", data: stockData, backgroundColor: "blue" }
      ];
      chart.update();

      // تحديث الرسم البياني الدائري (المبيعات فقط)
      pieChart.data.labels = labels;
      pieChart.data.datasets[0].data = salesData;
      pieChart.update();

      // تحديث الرسم البياني الخطي (الأرباح)
      profitChart.data.labels = labels;
      profitChart.data.datasets[0].data = profitData;
      profitChart.update();
    });
  }

  // دالة لإضافة كمية المبيعات مع تحديث Firestore
  window.addSales = async (collectionName, productId) => {
    const input = document.getElementById(`${collectionName}-${productId}-add-sales`);
    const qty = parseInt(input.value);

    if (isNaN(qty) || qty <= 0) return alert("أدخل رقم صالح.");

    const ref = doc(db, collectionName, productId);
    const snap = await getDoc(ref);

    if (snap.exists()) {
      const data = snap.data();
      const currentSold = data.sold || 0;
      const currentStock = data.stock || 0;

      if (qty > currentStock) {
        return alert("❌ لا يوجد مخزون كافٍ لإتمام العملية.");
      }

      // تحديث المبيعات والمخزون في قاعدة البيانات
      await updateDoc(ref, {
        sold: currentSold + qty,
        stock: currentStock - qty
      });

      input.value = ""; // تفريغ حقل الإدخال بعد الإضافة
    }
  };

  // دالة لإضافة كمية المخزون مع تحديث Firestore
  window.addStock = async (collectionName, productId) => {
    const input = document.getElementById(`${collectionName}-${productId}-add-stock`);
    const qty = parseInt(input.value);

    if (isNaN(qty) || qty < 0) return alert("أدخل رقم صالح.");

    const ref = doc(db, collectionName, productId);
    const snap = await getDoc(ref);

    if (snap.exists()) {
      const current = snap.data().stock || 0;
      await updateDoc(ref, { stock: current + qty });
      input.value = ""; // تفريغ الحقل بعد التحديث
    }
  };

  // دالة فلترة الجدول حسب كلمة البحث
  function filterTable(tableBodyId, searchTerm) {
    const tableBody = document.getElementById(tableBodyId);
    const rows = tableBody.getElementsByTagName('tr');
    const term = searchTerm.toLowerCase();

    for (let row of rows) {
      const nameCell = row.getElementsByTagName('td')[0];
      if (nameCell) {
        const text = nameCell.textContent.toLowerCase();
        row.style.display = text.includes(term) ? '' : 'none';
      }
    }
  }
function loadForecastFromExistingData() {
  const tableBody = document.getElementById("forecast-table-body");
  const alertsDiv = document.getElementById("alerts");
  tableBody.innerHTML = "";
  alertsDiv.innerHTML = "";

  // ثابتات لحساب EOQ (تقديرية)
  const orderCost = 50; // تكلفة الطلب
  const holdingCost = 5; // تكلفة التخزين لكل وحدة

  const deliveryDays = 3; // مدة التوصيل ثابتة

  // نجمع البيانات من Flower و chocolate
  const collectionsToLoad = ["Flower", "chocolate"];
  const allData = [];

  let collectionsLoaded = 0;

  collectionsToLoad.forEach(collectionName => {
    getDocs(collection(db, collectionName)).then(snapshot => {
      snapshot.forEach(docSnap => {
        const data = docSnap.data();
        allData.push({
          name: docSnap.id,
          sales: data.sold || 0,
          stock: data.stock || 0,
          price: data.price || 0
        });
      });

      collectionsLoaded++;
      if (collectionsLoaded === collectionsToLoad.length) {
        // كل البيانات جمعت، نحسب التنبؤات
        allData.forEach(item => {
          const D = item.sales; // مجموع المبيعات خلال فترة (مثلاً شهر)
          
          // نفرض أن فترة المراقبة 30 يوم عشان نطلع متوسط الطلب اليومي
          const avgDailyDemand = D / 30;

          const S = orderCost;
          const H = holdingCost;

          // حساب EOQ مع تقريب لأعلى عدد صحيح
          const EOQ = Math.ceil(Math.sqrt((2 * D * S) / H));

          // حساب نقطة إعادة الطلب = متوسط الطلب اليومي × مدة التوصيل مع تقريب لأعلى عدد صحيح
          const reorderPoint = Math.ceil(avgDailyDemand * deliveryDays);

          // التنبيه لو المخزون أقل من 5
          const alertMsg = item.stock <= 5 ? "مخزون منخفض" : "";

          const row = document.createElement("tr");
          row.innerHTML = `
            <td>${item.name}</td>
            <td>${D}</td>
            <td>${item.stock}</td>
            <td>${deliveryDays}</td>
            <td>${EOQ}</td>
            <td>${reorderPoint}</td>
            <td style="color: ${alertMsg ? "red" : "black"};">${alertMsg}</td>
          `;
          tableBody.appendChild(row);

          if (alertMsg) {
            const alertDiv = document.createElement("div");
            alertDiv.textContent = `تنبيه: ${item.name} - ${alertMsg}`;
            alertsDiv.appendChild(alertDiv);
          }
        });
      }
    });
  });
}

window.addSales = async (collectionName, productId) => {
  const input = document.getElementById(`${collectionName}-${productId}-add-sales`);
  const qty = parseInt(input.value);

  if (isNaN(qty) || qty <= 0) return alert("أدخل رقم صالح.");

  const ref = doc(db, collectionName, productId);
  const snap = await getDoc(ref);

  if (snap.exists()) {
    const data = snap.data();
    const currentSold = data.sold || 0;
    const currentStock = data.stock || 0;
    const currentSalesCount = data.salesCount || 0;  // قراءة عدد مرات البيع الحالي

    if (qty > currentStock) {
      return alert("❌ لا يوجد مخزون كافٍ لإتمام العملية.");
    }

    // تحديث المبيعات، المخزون وعدد مرات البيع
    await updateDoc(ref, {
      sold: currentSold + qty,
      stock: currentStock - qty,
      salesCount: currentSalesCount + 1    // زيادة عدد مرات البيع بواحد فقط
    });

    input.value = ""; // تفريغ حقل الإدخال بعد الإضافة
  }
};

// عند تحميل الصفحة: تهيئة الرسوم وتحميل البيانات وحساب EOQ ونقطة إعادة الطلب
window.addEventListener('load', () => {
  initializeCharts();

  // تحميل بيانات الزهور والشوكولاتة للرسوم والجداول
  loadCollectionData("Flower", "flower-table-body", flowerChart, flowerPieChart, flowerProfitChart);
  loadCollectionData("chocolate", "chocolate-table-body", chocolateChart, chocolatePieChart, chocolateProfitChart);

  // تحميل حسابات الجدول النهائي (EOQ، reorder point والتنبيهات)
  loadForecastFromExistingData();
});

</script>


</head>
<body>
  <h1>إدارة المخزون والمبيعات</h1>
<div class="container">
  <div class="section">
    <h2>الزهور</h2>
    <input
      type="text"
      id="flower-search"
      placeholder="ابحث باسم المنتج في الزهور..."
      onkeyup="filterTable('flower-table-body', this.value)"
      style="width: 100%; padding: 8px; margin-bottom: 10px;"
    />
    <table>
      <thead>
       <tr>
  <th>الاسم</th>
  <th>المبيعات</th>
  <th>عدد مرات البيع</th>  <!-- أضف هذا العمود -->
  <th>المخزون</th>
  <th>السعر</th>
  <th>الإجمالي (ريال)</th>
  <th>الإجراءات</th>
</tr>

      </thead>
      <tbody id="flower-table-body"></tbody>
    </table>
    <div class="charts">
      <canvas id="flower-chart"></canvas>
      <canvas id="flower-pie-chart"></canvas>
      <canvas id="flower-profit-chart"></canvas>
    </div>
  </div>

  <div class="section">
    <h2>الشوكولاتة</h2>
    <input
      type="text"
      id="chocolate-search"
      placeholder="ابحث باسم المنتج في الشوكولاتة..."
      onkeyup="filterTable('chocolate-table-body', this.value)"
      style="width: 100%; padding: 8px; margin-bottom: 10px;"
    />
    <table>
      <thead>
        <tr>
          <th>الاسم</th>
             <th>عدد الوحدات المباعة</th>  <!-- بدل "المبيعات" -->
    <th>عدد مرات البيع</th> 
          <th>المخزون</th>
          <th>السعر</th>
          <th>الإجمالي (ريال)</th>
          <th>الإجراءات</th>
        </tr>
      </thead>
      <tbody id="chocolate-table-body"></tbody>
    </table>
    <div class="charts">
      <canvas id="chocolate-chart"></canvas>
      <canvas id="chocolate-pie-chart"></canvas>
      <canvas id="chocolate-profit-chart"></canvas>
    </div>
  </div>

  <!-- قسم توقعات الطلب والتنبيهات -->
    <div class="section" style="margin-top: 40px;">
      <h2>توقعات الطلب والتنبيهات</h2>
      <table>
        <thead>
          <tr>
            <th>اسم المنتج</th>
               
    <th>عدد مرات البيع</th> 
            <th>المخزون الحالي</th>
            <th>مدة التوصيل (يوم)</th>
            <th>EOQ</th>
            <th>نقطة إعادة الطلب</th>

            <th>تنبيه</th>
          </tr>
        </thead>
        <tbody id="forecast-table-body"></tbody>
      </table>
      <div id="alerts" style="margin-top: 20px; font-weight: bold; color: darkred;"></div>
    </div>
  </div>
  <script>

    
  // دالة لتصفية الجدول بناءً على البحث باسم المنتج
function filterTable(tableBodyId, searchTerm) {
  const tableBody = document.getElementById(tableBodyId);
  const rows = tableBody.getElementsByTagName('tr');
  const filter = searchTerm.toLowerCase();

  for (let i = 0; i < rows.length; i++) {
    const firstCell = rows[i].getElementsByTagName('td')[0];
    if (firstCell) {
      const txtValue = firstCell.textContent || firstCell.innerText;
      if (txtValue.toLowerCase().indexOf(filter) > -1) {
        rows[i].style.display = "";
      } else {
        rows[i].style.display = "none";
      }
    }
  }
}
  </script>
</body>
</html>
