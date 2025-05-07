<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>GST Calculator</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    :root {
      --bg-color: #f4f4f4;
      --text-color: #333;
      --card-bg: white;
      --accent-color: #007bff;
    }

    [data-theme="dark"] {
      --bg-color: #1f1f1f;
      --text-color: #eee;
      --card-bg: #2a2a2a;
      --accent-color: #00c3ff;
    }

    body {
      background-color: var(--bg-color);
      color: var(--text-color);
      font-family: 'Segoe UI', sans-serif;
      display: flex;
      flex-direction: column;
      align-items: center;
      margin: 0;
      padding: 2rem;
      transition: background 0.4s, color 0.4s;
      min-height: 100vh;
    }

    .container {
      max-width: 400px;
      width: 100%;
      background: var(--card-bg);
      padding: 2rem;
      border-radius: 15px;
      box-shadow: 0 4px 10px rgba(0,0,0,0.1);
      position: relative;
    }

    input, select {
      width: 100%;
      padding: 0.7rem;
      margin: 0.5rem 0;
      border: 1px solid #ccc;
      border-radius: 8px;
      font-size: 1rem;
      text-align: left; /* align text left */
      height: 45px;
      box-sizing: border-box;
    }

    button {
      width: 100%;
      padding: 0.7rem;
      margin: 0.5rem 0;
      border: 1px solid #ccc;
      border-radius: 8px;
      font-size: 1rem;
      height: 45px;
      text-align: center;      
      background: var(--accent-color);
      color: white;
      font-weight: bold;
      cursor: pointer;
      transition: transform 0.2s, background 0.3s;
    }

    button:hover {
      transform: scale(1.05);
      background: #0056b3;
    }

    .result-card {
      display: none;
      background: var(--card-bg);
      padding: 1rem;
      margin-top: 1rem;
      border-radius: 10px;
      animation: fadeIn 0.8s ease-in forwards;
      box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
    }

    @keyframes fadeIn {
      from {opacity: 0;}
      to {opacity: 1;}
    }

    .spinner {
      display: none;
      margin-top: 1rem;
      text-align: center;
    }

    .info-button {
      position: absolute;
      top: 1rem;
      left: 1rem;
      width: calc(100% - 2rem);
      height: auto;
      padding: 0.7rem;
      background: var(--accent-color);
      color: white;
      border: none;
      border-radius: 8px;
      font-size: 1rem;
      cursor: pointer;
      transition: background 0.3s, transform 0.2s;
      display: flex;
      align-items: center;
      justify-content: center;
    }

    .info-button:hover {
      background: #0056b3;
      transform: scale(1.05);
    }

    .theme-icon {
      position: absolute;
      right: 10px;
      font-size: 1.6rem;
      cursor: pointer;
      transition: transform 0.3s;
    }

    .theme-icon:hover {
      transform: scale(1.3) rotate(20deg);
    }

    .info-box {
      display: none;
      position: absolute;
      top: 4.5rem;
      left: 1rem;
      background: var(--card-bg);
      padding: 1rem;
      border-radius: 10px;
      box-shadow: 0 0 10px rgba(0,0,0,0.2);
      width: calc(100% - 2rem);
      z-index: 100;
    }

    canvas {
      margin-top: 1rem;
    }

    footer {
      margin-top: auto;
      text-align: center;
      padding: 1rem;
      font-size: 0.9rem;
      color: var(--text-color);
    }

    @media(max-width: 480px) {
      .container {
        padding: 1rem;
      }
    }

    @media print {
      body * {
        visibility: hidden;
      }
      .result-card, .result-card * {
        visibility: visible;
      }
      .result-card {
        position: absolute;
        top: 10%;
        left: 0;
        width: 100%;
        padding: 2rem;
        background: white !important;
        color: black !important;
        border: none;
        box-shadow: none;
      }
    }

    h2 {
      margin-top: 3rem;
      font-size: 1.5rem;
      text-align: center;
      color: var(--text-color);
    }
  </style>
</head>
<body>
  <div class="container">
    <button class="info-button" onclick="toggleInfo()">
      ℹ
      <span class="theme-icon" onclick="event.stopPropagation(); toggleTheme()">🌞</span>
    </button>
    <div class="info-box" id="infoBox">
      GST (Goods and Services Tax) is a value-added tax levied on most goods and services sold for domestic consumption in India. This tool helps you calculate GST inclusively or exclusively.
    </div>

    <h2>GST Calculator</h2>
    <input type="number" id="amount" placeholder="Enter Amount">
    <select id="gstRate">
      <option value="5">5%</option>
      <option value="12">12%</option>
      <option value="18">18%</option>
      <option value="28">28%</option>
    </select>
    <select id="gstType">
      <option value="exclusive">GST Exclusive</option>
      <option value="inclusive">GST Inclusive</option>
    </select>
    <button onclick="calculateGST()">Calculate</button>
    <button onclick="clearAll()">Clear All</button>

    <div class="spinner" id="spinner">Calculating...</div>

    <div class="result-card" id="resultCard">
      <p id="gstDetails"></p>
      <p id="funMessage"></p>
      <canvas id="gstChart" width="300" height="300"></canvas>
    </div>
  </div>

  <footer>
    © 2025 GST Calculator || Made by Afiya Amer | Alfia suroor | Amatullah || Shadan Women's College Of Engineering And Technology
  </footer>

  <script>
    let currentTheme = 'light';

    function toggleTheme() {
      currentTheme = currentTheme === 'light' ? 'dark' : 'light';
      document.documentElement.setAttribute('data-theme', currentTheme);
      const themeIcon = document.querySelector('.theme-icon');
      themeIcon.innerText = currentTheme === 'light' ? '🌞' : '🌜';
    }

    function toggleInfo() {
      const box = document.getElementById('infoBox');
      box.style.display = box.style.display === 'block' ? 'none' : 'block';
    }

    function getColorByRate(rate) {
      switch (+rate) {
        case 5: return 'green';
        case 12: return 'orange';
        case 18: return 'blue';
        case 28: return 'red';
        default: return 'gray';
      }
    }

    function clearAll() {
      document.getElementById('amount').value = '';
      document.getElementById('gstRate').value = '5';
      document.getElementById('gstType').value = 'exclusive';
      document.getElementById('resultCard').style.display = 'none';
    }

    function calculateGST() {
      const amount = parseFloat(document.getElementById('amount').value);
      let gstRate = parseFloat(document.getElementById('gstRate').value);
      const gstType = document.getElementById('gstType').value;

      if (isNaN(amount)) {
        alert('Please enter a valid amount.');
        return;
      }

      document.getElementById('spinner').style.display = 'block';
      document.getElementById('resultCard').style.display = 'none';

      setTimeout(() => {
        let gstAmount, totalAmount, baseAmount = amount;

        if (gstType === 'exclusive') {
          gstAmount = (amount * gstRate) / 100;
          totalAmount = amount + gstAmount;
        } else {
          gstAmount = (amount * gstRate) / (100 + gstRate);
          totalAmount = amount;
          baseAmount = amount - gstAmount;
        }

        document.getElementById('gstDetails').innerHTML = `
          <strong style="color: ${getColorByRate(gstRate)};">Base Amount: ₹${baseAmount.toFixed(2)}</strong><br>
          <strong style="color: ${getColorByRate(gstRate)};">GST Amount: ₹${gstAmount.toFixed(2)}</strong><br>
          <strong style="color: ${getColorByRate(gstRate)};">Total Amount: ₹${totalAmount.toFixed(2)}</strong>
        `;

        const messages = [
          "GST Made Easy!",
          "You saved ₹" + gstAmount.toFixed(0) + " in tax!",
          "Simple. Smart. Tax Ready."
        ];

        document.getElementById('funMessage').innerText = messages[Math.floor(Math.random() * messages.length)];

        document.getElementById('resultCard').style.display = 'block';
        document.getElementById('spinner').style.display = 'none';

        renderChart(baseAmount, gstAmount, getColorByRate(gstRate));
      }, 1000);
    }

    function renderChart(base, gst, color) {
      const ctx = document.getElementById('gstChart').getContext('2d');
      if (window.gstChart && typeof window.gstChart.destroy === 'function') {
        window.gstChart.destroy();
      }
      window.gstChart = new Chart(ctx, {
        type: 'pie',
        data: {
          labels: ['Base Amount', 'GST'],
          datasets: [{
            data: [base, gst],
            backgroundColor: ['#ccc', color],
          }]
        },
        options: {
          responsive: true,
          plugins: {
            legend: {
              position: 'top',
              labels: {
                font: {
                  size: 14
                }
              }
            },
            tooltip: {
              callbacks: {
                label: function(tooltipItem) {
                  return tooltipItem.label + ': ₹' + tooltipItem.raw.toFixed(2);
                }
              }
            }
          }
        }
      });
    }
  </script>
</body>
</html>
