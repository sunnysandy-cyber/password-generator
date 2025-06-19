<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Advanced Password Generator</title>
  <style>
    :root {
      --bg: #f0f2f5;
      --text: #333;
      --primary: #007bff;
      --success: #28a745;
      --danger: #dc3545;
    }
    body.dark {
      --bg: #121212;
      --text: #eee;
    }
    body {
      background: var(--bg);
      color: var(--text);
      font-family: 'Segoe UI', sans-serif;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      margin: 0;
      transition: background 0.3s;
    }
    .container {
      background: #fff;
      padding: 30px;
      border-radius: 15px;
      box-shadow: 0 0 15px rgba(0,0,0,0.1);
      width: 90%;
      max-width: 500px;
    }
    body.dark .container {
      background: #1e1e1e;
    }
    h2 {
      text-align: center;
    }
    .output {
      background: #f4f4f4;
      padding: 10px;
      border-radius: 8px;
      margin-bottom: 10px;
      word-break: break-all;
    }
    .strength {
      margin: 10px 0;
      font-weight: bold;
    }
    .controls label {
      display: block;
      margin: 5px 0;
    }
    .history {
      margin-top: 10px;
      font-size: 0.9em;
    }
    .btn {
      display: inline-block;
      padding: 10px 20px;
      margin: 5px 2px;
      background-color: var(--primary);
      color: white;
      border: none;
      border-radius: 8px;
      cursor: pointer;
    }
    .btn-success { background-color: var(--success); }
    .btn-danger { background-color: var(--danger); }
    .toggle-dark {
      float: right;
      margin-bottom: 10px;
      cursor: pointer;
    }
    input[type=range] {
      width: 100%;
    }
    @media print {
      .btn, .toggle-dark, .history { display: none; }
    }
  </style>
</head>
<body>
  <div class="container" role="main">
    <div class="toggle-dark" onclick="toggleDarkMode()">🌓 Dark Mode</div>
    <h2>Password Generator</h2>
    <div class="output" id="password" aria-label="Generated password">Your password will appear here</div>
    <div class="strength" id="strength">Strength: N/A</div>
    <div class="controls">
      <label>Password Length: <span id="lengthValue">12</span></label>
      <input type="range" id="length" min="6" max="32" value="12" oninput="lengthValue.innerText = this.value">
      <label><input type="checkbox" id="uppercase" checked> Include Uppercase</label>
      <label><input type="checkbox" id="lowercase" checked> Include Lowercase</label>
      <label><input type="checkbox" id="numbers" checked> Include Numbers</label>
      <label><input type="checkbox" id="symbols"> Include Symbols</label>
      <label><input type="checkbox" id="avoidAmbiguous"> Avoid Ambiguous Characters</label>
      <label><input type="checkbox" id="autoGen"> Auto Regenerate Every 5s</label>
    </div>
    <button class="btn" onclick="generatePassword()">Generate</button>
    <button class="btn btn-success" onclick="copyPassword()">Copy</button>
    <button class="btn btn-danger" onclick="exportPasswords()">Export History</button>
    <div class="history">
      <h4>Password History:</h4>
      <ul id="history"></ul>
    </div>
  </div>
  <script>
    const passwordEl = document.getElementById('password');
    const strengthEl = document.getElementById('strength');
    const historyEl = document.getElementById('history');
    let passwordHistory = [];
    let autoInterval;

    function generatePassword() {
      const length = +document.getElementById('length').value;
      const useUpper = document.getElementById('uppercase').checked;
      const useLower = document.getElementById('lowercase').checked;
      const useNumbers = document.getElementById('numbers').checked;
      const useSymbols = document.getElementById('symbols').checked;
      const avoidAmbiguous = document.getElementById('avoidAmbiguous').checked;

      const upper = 'ABCDEFGHJKLMNPQRSTUVWXYZ';
      const lower = 'abcdefghijkmnopqrstuvwxyz';
      const numbers = '23456789';
      const symbols = '!@#$%^&*()-_=+[]{};:,.<>?';
      const ambiguous = '0O1lI';

      let chars = '';
      if (useUpper) chars += upper;
      if (useLower) chars += lower;
      if (useNumbers) chars += numbers;
      if (useSymbols) chars += symbols;
      if (!avoidAmbiguous) chars += ambiguous;

      if (!chars) {
        passwordEl.innerText = 'Select at least one character set!';
        return;
      }

      let password = '';
      for (let i = 0; i < length; i++) {
        password += chars.charAt(Math.floor(Math.random() * chars.length));
      }

      passwordEl.innerText = password;
      showStrength(password);
      updateHistory(password);
    }

    function showStrength(pwd) {
      let score = 0;
      if (/[A-Z]/.test(pwd)) score++;
      if (/[a-z]/.test(pwd)) score++;
      if (/[0-9]/.test(pwd)) score++;
      if (/[^A-Za-z0-9]/.test(pwd)) score++;
      if (pwd.length > 12) score++;

      const levels = ['Very Weak', 'Weak', 'Fair', 'Strong', 'Very Strong'];
      strengthEl.innerText = `Strength: ${levels[score - 1] || 'N/A'}`;
    }

    function copyPassword() {
      const text = passwordEl.innerText;
      navigator.clipboard.writeText(text);
      const btn = event.target;
      btn.innerText = 'Copied!';
      setTimeout(() => btn.innerText = 'Copy', 2000);
    }

    function updateHistory(pwd) {
      passwordHistory.unshift(pwd);
      if (passwordHistory.length > 5) passwordHistory.pop();
      historyEl.innerHTML = passwordHistory.map(p => `<li>${p}</li>`).join('');
    }

    function exportPasswords() {
      const blob = new Blob([passwordHistory.join('\n')], { type: 'text/plain' });
      const a = document.createElement('a');
      a.href = URL.createObjectURL(blob);
      a.download = 'passwords.txt';
      a.click();
    }

    function toggleDarkMode() {
      document.body.classList.toggle('dark');
    }

    document.getElementById('autoGen').addEventListener('change', function () {
      if (this.checked) {
        autoInterval = setInterval(generatePassword, 5000);
      } else {
        clearInterval(autoInterval);
      }
    });
  </script>
</body>
</html>
