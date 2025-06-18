
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>نظام طباعة الباركود</title>
  <script src="https://cdn.jsdelivr.net/npm/jsbarcode@3.11.5/dist/JsBarcode.all.min.js"></script>
  <style>
    body {
      margin: 0;
      font-family: Arial, sans-serif;
      background: #f5f5f5;
    }
    .container {
      max-width: 500px;
      margin: 20px auto;
      padding: 20px;
      background: white;
      border-radius: 10px;
      box-shadow: 0 0 15px rgba(0,0,0,0.1);
    }
    h2 {
      color: #2c3e50;
      text-align: center;
      margin-bottom: 25px;
    }
    .label {
      border: 1px solid #000;
      margin: 15px auto;
      padding: 10px;
      text-align: center;
      background: white;
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      page-break-inside: avoid;
    }
    .store-name {
      font-size: 20px;
      font-weight: bold;
      margin-bottom: 5px;
    }
    .price {
      font-size: 16px;
      font-weight: bold;
      margin-bottom: 10px;
    }
    .barcode-container {
      margin: 5px 0;
    }
    .barcode-number {
      font-family: monospace;
      font-size: 14px;
      letter-spacing: 1px;
      margin-top: 5px;
    }
    .form-group {
      margin-bottom: 25px;
    }
    .form-group label {
      display: block;
      margin-bottom: 8px;
      font-weight: 600;
      color: #34495e;
      font-size: 18px;
    }
    select, input {
      width: 100%;
      padding: 12px;
      border: 1px solid #ddd;
      border-radius: 6px;
      font-size: 16px;
      background: #f9f9f9;
    }
    .print-btn {
      width: 100%;
      padding: 15px;
      background: #2ecc71;
      color: white;
      border: none;
      border-radius: 6px;
      font-size: 18px;
      font-weight: bold;
      cursor: pointer;
      transition: background 0.3s;
      margin-top: 15px;
    }
    .print-btn:hover {
      background: #27ae60;
    }
    @media print {
      body * {
        visibility: hidden;
      }
      #printPreview, #printPreview * {
        visibility: visible;
      }
      #printPreview {
        position: absolute;
        left: 0;
        top: 0;
        width: 100%;
      }
    }
  </style>
</head>
<body>
  <div class="container">
    <div id="printer">
      <h2>طباعة الباركود</h2>

      <div class="form-group">
        <label for="printProduct">اختر منتج للطباعة</label>
        <select id="printProduct">
          <option value="">-- اختر منتجا --</option>
        </select>
      </div>

      <div class="form-group">
        <label for="printQty">عدد الملصقات (1-50)</label>
        <input type="number" id="printQty" value="1" min="1" max="50">
      </div>

      <div class="form-group">
        <label for="labelSize">حجم الملصق</label>
        <select id="labelSize">
          <option value="30x20">30x20 مم</option>
          <option value="40x30" selected>40x30 مم</option>
          <option value="50x30">50x30 مم</option>
        </select>
      </div>

      <button class="print-btn" id="printBtn">🖨️ طباعة الملصقات</button>
      <button class="print-btn" style="background:#3498db;" onclick="window.print()">🖨️ طباعة فورية</button>
      <button class="print-btn" style="background:#e74c3c;" onclick="document.getElementById('printPreview').innerHTML=''">🗑️ مسح المعاينة</button>

      <div id="printPreview"></div>
    </div>
  </div>

  <script>
    let products = [];

    try {
      const storedProducts = localStorage.getItem("products");
      if (storedProducts) {
        products = JSON.parse(storedProducts);
      }
    } catch (e) {
      console.error("خطأ في تحميل المنتجات:", e);
    }

    function initializeProducts() {
      const select = document.getElementById("printProduct");
      select.innerHTML = '<option value="">-- اختر منتجا --</option>';

      if (products.length === 0) {
        const option = document.createElement("option");
        option.textContent = "لا توجد منتجات متاحة";
        option.disabled = true;
        select.appendChild(option);
        return;
      }

      products.forEach(product => {
        const option = document.createElement("option");
        option.value = product.barcode;
        option.textContent = product.name;
        select.appendChild(option);
      });
    }

    function printBarcodes() {
      const barcode = document.getElementById("printProduct").value;
      const qty = parseInt(document.getElementById("printQty").value) || 1;
      const labelSize = document.getElementById("labelSize").value;
      const preview = document.getElementById("printPreview");

      if (!barcode) return alert("❗ يرجى اختيار منتج للطباعة");
      if (qty < 1 || qty > 50) return alert("❗ عدد الملصقات يجب أن يكون بين 1 و 50");

      const product = products.find(p => String(p.barcode) === String(barcode));
      if (!product) return alert("❌ المنتج غير موجود");

      // إعدادات حجم الملصق
      let labelWidth = "300px", labelHeight = "180px";
      let barcodeHeight = 40, barcodeWidth = 2;

      if (labelSize === "30x20") {
        labelWidth = "240px";
        labelHeight = "120px";
        barcodeHeight = 25;
        barcodeWidth = 1.5;
      } else if (labelSize === "50x30") {
        labelWidth = "360px";
        labelHeight = "180px";
        barcodeHeight = 50;
        barcodeWidth = 2.5;
      }

      preview.innerHTML = "";

      for (let i = 0; i < qty; i++) {
        const label = document.createElement("div");
        label.className = "label";
        label.style.width = labelWidth;
        label.style.height = labelHeight;
        label.innerHTML = `
          <div class="store-name">BAZAR SERDANI</div>
          <div class="price">Prix: ${formatPrice(product.price || 0)} DA</div>
          <div class="barcode-container">
            <svg class="barcode" data-code="${product.barcode}"></svg>
            <div class="barcode-number">${product.barcode}</div>
          </div>
        `;
        preview.appendChild(label);
      }

      document.querySelectorAll('.barcode').forEach(el => {
        const code = el.getAttribute("data-code");
        JsBarcode(el, code, {
          format: "EAN13",
          displayValue: false,
          height: barcodeHeight,
          margin: 5,
          width: barcodeWidth
        });
      });

      alert(`✅ تم تحضير ${qty} ملصق للطباعة`);
    }

    function formatPrice(price) {
      const priceStr = Math.round(price).toString();
      return priceStr.length > 3
        ? priceStr.slice(0, -3) + ' ' + priceStr.slice(-3)
        : priceStr;
    }

    document.addEventListener('DOMContentLoaded', function () {
      initializeProducts();
      document.getElementById("printBtn").addEventListener('click', printBarcodes);
    });
  </script>
</body>
</html>
