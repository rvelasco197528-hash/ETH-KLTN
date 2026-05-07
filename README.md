<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ETH-SECURE | Global Archive Vault</title>
    <!-- Ginamit ang Font Awesome para sa X at iba pang icons -->
    <link rel="stylesheet" href="https://cloudflare.com">
    <style>
        :root { --bg-body: #0b0e11; --sidebar: #111417; --card: #181a20; --border: #2b3139; --eth-blue: #627eea; --text-dim: #848e9c; --encryption: #0ecb81; --danger: #f6465d; }
        body { font-family: 'Inter', sans-serif; background: var(--bg-body); color: white; margin: 0; display: flex; height: 100vh; overflow: hidden; }
        
        /* SPINNER ANIMATION */
        .spinner { display: none; width: 35px; height: 35px; border: 4px solid rgba(255,255,255,0.1); border-left-color: var(--eth-blue); border-radius: 50%; animation: spin 1s linear infinite; margin: 20px auto; }
        @keyframes spin { to { transform: rotate(360deg); } }

        /* 1. REGISTRATION & OTP OVERLAY */
        #auth-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: var(--bg-body); z-index: 10000; display: flex; justify-content: center; align-items: center; }
        .auth-card { background: var(--card); padding: 40px; border-radius: 20px; border: 1px solid var(--border); width: 420px; text-align: center; }
        input, select { width: 100%; padding: 12px; margin: 8px 0; border-radius: 8px; border: 1px solid var(--border); background: #1e2329; color: white; box-sizing: border-box; font-size: 14px; }
        .btn-main { width: 100%; padding: 14px; background: var(--eth-blue); color: white; border: none; border-radius: 10px; cursor: pointer; font-weight: bold; margin-top: 15px; }

        /* 2. SIDEBAR NAVIGATION */
        #sidebar { width: 280px; background: var(--sidebar); border-right: 1px solid var(--border); padding: 25px; display: none; flex-direction: column; }
        .logo { font-size: 22px; font-weight: 800; color: #fff; margin-bottom: 40px; }
        .nav-item { padding: 12px; border-radius: 8px; color: var(--text-dim); cursor: pointer; margin-bottom: 5px; font-size: 14px; }
        .nav-item.active { background: var(--card); color: white; border: 1px solid var(--border); }
        .status-btn { margin-top: 20px; padding: 15px; border-top: 1px solid var(--border); cursor: pointer; text-align: left; }

        /* 3. MAIN DASHBOARD CONTENT */
        #main-content { flex: 1; overflow-y: auto; padding: 40px; display: none; }
        .summary-banner { background: rgba(255, 255, 255, 0.02); border: 1px solid var(--border); border-radius: 12px; padding: 25px; margin-bottom: 30px; display: flex; justify-content: space-around; text-align: center; }
        .table-section { background: var(--card); border-radius: 16px; border: 1px solid var(--border); overflow: hidden; }
        table { width: 100%; border-collapse: collapse; text-align: left; }
        th { background: #1e2329; padding: 18px; color: var(--text-dim); font-size: 11px; }
        td { padding: 15px 18px; border-bottom: 1px solid var(--border); font-size: 13px; font-family: monospace; }

        /* 4. WALLET MODAL (WHITE BOX) */
        #wallet-modal { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.85); display: none; justify-content: center; align-items: center; z-index: 20000; }
        .white-box { background: white; color: black; width: 360px; border-radius: 24px; padding: 30px; position: relative; text-align: left; }
        .close-btn { position: absolute; top: 20px; right: 25px; font-size: 24px; color: #666; cursor: pointer; }
        .close-btn:hover { color: black; }
        .white-input { background: #f5f5f5; border: 1px solid #eee; width: 100%; padding: 12px; margin-bottom: 10px; border-radius: 8px; color: black; box-sizing: border-box; }
        .btn-connect { background: black; color: white; width: 100%; padding: 15px; border-radius: 12px; font-weight: bold; border: none; cursor: pointer; }
    </style>
</head>
<body>

<!-- AUTH SECTION -->
<div id="auth-overlay">
    <!-- REGISTRATION -->
    <div class="auth-card" id="reg-box">
        <h2>Archive Access</h2>
        <p>Identity verification required</p>
        <div id="reg-loader" class="spinner"></div>
        <div id="reg-inputs">
            <input type="text" id="r-name" placeholder="Full Name">
            <input type="date" id="r-birth">
            <select id="r-country">
                <option value="" disabled selected>Select Country</option>
                <option value="Philippines">Philippines</option>
                <option value="United States">United States</option>
                <option value="United Kingdom">United Kingdom</option>
                <option value="Canada">Canada</option>
                <option value="Australia">Australia</option>
                <option value="Singapore">Singapore</option>
                <option value="United Arab Emirates">UAE</option>
                <option value="Switzerland">Switzerland</option>
                <option value="Japan">Japan</option>
                <option value="Others">Others</option>
            </select>
            <input type="tel" id="r-phone" placeholder="Phone Number">
            <input type="email" id="r-email" placeholder="Email (@)">
            <input type="password" id="r-pass" placeholder="Create Password">
            <button class="btn-main" onclick="handleRegistration()">Verify Identity</button>
        </div>
    </div>

    <!-- OTP VERIFICATION -->
    <div class="auth-card" id="otp-box" style="display:none;">
        <h2 style="color: var(--encryption);">Verify OTP</h2>
        <div id="otp-loader" class="spinner"></div>
        <div id="otp-inputs">
            <div id="display-otp" style="font-size: 40px; font-weight: bold; color: var(--encryption); margin: 20px 0; letter-spacing: 10px;">000000</div>
            <input type="text" id="input-otp" placeholder="Enter 6-digit Code" maxlength="6" style="text-align:center;">
            <button class="btn-main" style="background: var(--encryption);" onclick="handleOTPVerify()">Unlock Dashboard</button>
        </div>
    </div>
</div>

<!-- SIDEBAR -->
<nav id="sidebar">
    <div class="logo">ETH-SECURE</div>
    <div class="nav-item active">Vault</div>
    <div class="nav-item">Audit Logs</div>
    
    <div class="status-btn" onclick="openModal()">
        <div style="font-size: 12px; color: var(--text-dim); display: flex; align-items: center; gap: 8px;">
            <i class="fa-solid fa-link-slash"></i> No wallet connected
        </div>
        <div style="font-size: 11px; color: #5a606c; margin-top: 5px;">Missing Private Key.</div>
    </div>

    <div style="margin-top: auto; font-size: 12px; color: var(--text-dim); cursor: pointer;" onclick="location.reload()">
        <i class="fa-solid fa-lock"></i> Lock Vault
    </div>
</nav>

<!-- MAIN DASHBOARD -->
<main id="main-content">
    <div id="dash-loader" class="spinner" style="display:block; margin-top: 100px;"></div>
    <div id="dash-data" style="display:none;">
        <h2 style="margin-bottom: 25px;">Archived Wallet: <span style="color: var(--eth-blue); font-size: 18px;">0x71C7...d289</span></h2>
        <div class="summary-banner">
            <div><h4 style="color: var(--text-dim); font-size: 11px;">TOTAL LOAD</h4><div>35 Txns</div></div>
            <div><h4 style="color: var(--text-dim); font-size: 11px;">ACCUMULATED OUT</h4><div style="color: var(--danger);">- 24.3600 ETH</div></div>
            <div><h4 style="color: var(--text-dim); font-size: 11px;">LIQUID BALANCE</h4><div>0.0000 ETH</div></div>
        </div>
        <div class="table-section">
            <table>
                <thead><tr><th>Hash</th><th>Encrypted Timestamp</th><th>Value</th><th>Status</th></tr></thead>
                <tbody>
                    <tr><td style="color: var(--eth-blue);">0x823b...7a2d</td><td style="color: var(--encryption);">0x66BC92...AF01</td><td style="color: var(--danger);">- 24.3600 ETH</td><td style="color: var(--encryption);">SECURE</td></tr>
                </tbody>
            </table>
        </div>
    </div>
</main>

<!-- WALLET MODAL -->
<div id="wallet-modal">
    <div class="white-box">
        <!-- ITO ANG X BUTTON -->
        <i class="fa-solid fa-xmark close-btn" onclick="closeModal()"></i>
        
        <h2 style="margin: 30px 0 10px 0;">Wallet List</h2>
        <p style="color: #666; font-size: 13px; margin-bottom: 25px; line-height: 1.5;">Please set a main wallet to check tokens and NFTs.</p>
        
        <input type="text" class="white-input" placeholder="Wallet Name">
        <input type="password" class="white-input" placeholder="Private Key">
        
        <button class="btn-connect" onclick="alert('Access Denied: Private Key Missing')">Connect wallet</button>
    </div>
</div>

<script>
    let generatedOTP = "";

    function handleRegistration() {
        const email = document.getElementById('r-email').value;
        if (!email.includes('@')) { alert("Please enter a valid email."); return; }
        
        document.getElementById('reg-inputs').style.display = 'none';
        document.getElementById('reg-loader').style.display = 'block';
        
        setTimeout(() => {
            document.getElementById('reg-box').style.display = 'none';
            document.getElementById('otp-box').style.display = 'block';
            generatedOTP = Math.floor(100000 + Math.random() * 900000).toString();
            document.getElementById('display-otp').innerText = generatedOTP;
        }, 2000);
    }

    function handleOTPVerify() {
        const input = document.getElementById('input-otp').value;
        if(input === generatedOTP) {
            document.getElementById('otp-inputs').style.display = 'none';
            document.getElementById('otp-loader').style.display = 'block';
            
            setTimeout(() => {
                document.getElementById('auth-overlay').style.display = 'none';
                document.getElementById('sidebar').style.display = 'flex';
                document.getElementById('main-content').style.display = 'block';
                
                // Dashboard Loading simulation
                setTimeout(() => {
                    document.getElementById('dash-loader').style.display = 'none';
                    document.getElementById('dash-data').style.display = 'block';
                }, 1500);
            }, 1500);
        } else { alert("Incorrect OTP"); }
    }

    function openModal() { document.getElementById('wallet-modal').style.display = 'flex'; }
    function closeModal() { document.getElementById('wallet-modal').style.display = 'none'; }
</script>

</body>
</html>
