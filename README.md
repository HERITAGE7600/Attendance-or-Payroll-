# Attendance-or-Payroll-
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Heritage Auto Finance - Admin</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.5.23/jspdf.plugin.autotable.min.js"></script>
    <style>
        :root { --gold: #c59d5f; --purple: #3d2b56; --bg: #f8fafc; }
        body { font-family: 'Segoe UI', sans-serif; background: var(--bg); margin: 0; padding: 10px; color: #334155; }
        .container { max-width: 500px; margin: auto; background: white; padding: 20px; border-radius: 15px; box-shadow: 0 8px 20px rgba(0,0,0,0.1); border-top: 6px solid var(--gold); }
        .header { text-align: center; margin-bottom: 20px; }
        .logo { width: 150px; height: auto; margin-bottom: 10px; }
        h1 { color: var(--purple); font-size: 1.2rem; margin: 0; text-transform: uppercase; }
        .tagline { font-size: 0.7rem; color: var(--gold); font-weight: bold; margin-bottom: 15px; }
        .card { border: 1px solid #e2e8f0; padding: 15px; border-radius: 12px; margin-bottom: 15px; background: #fff; }
        input, select { width: 100%; padding: 12px; margin: 8px 0; border: 1px solid #cbd5e1; border-radius: 8px; font-size: 16px; box-sizing: border-box; }
        .btn { width: 100%; padding: 14px; border: none; border-radius: 8px; font-weight: bold; cursor: pointer; transition: 0.3s; margin-top: 5px; font-size: 1rem; }
        .btn-purple { background: var(--purple); color: white; }
        .btn-gold { background: var(--gold); color: white; }
        .btn-link { background: none; color: var(--purple); text-decoration: underline; font-size: 0.85rem; border: none; cursor: pointer; display: block; margin: 10px auto; }
        .table-wrap { overflow-x: auto; margin-top: 10px; }
        table { width: 100%; border-collapse: collapse; font-size: 0.8rem; }
        th, td { border: 1px solid #f1f5f9; padding: 10px; text-align: left; }
        th { background: #fafafa; color: var(--purple); }
        .hidden { display: none; }
    </style>
</head>
<body>

<div class="container">
    <div class="header">
        <img src="https://i.postimg.cc/mD8zW0bN/heritage-auto-finance-logo-removebg-preview.png" class="logo" alt="Heritage Logo">
        <h1>Heritage Auto Finance</h1>
        [span_0](start_span)<div class="tagline">Trusted and Reliable Loan Services[span_0](end_span)</div>
    </div>

    <div id="login-view">
        <div class="card">
            <select id="role" onchange="document.getElementById('c-box').classList.toggle('hidden', this.value==='owner')">
                <option value="employee">Staff Login</option>
                <option value="owner">Owner Login</option>
            </select>
            <input type="text" id="u-user" placeholder="Name / Username">
            <div id="c-box"><input type="text" id="u-code" placeholder="Employee Code"></div>
            <input type="password" id="u-pass" placeholder="Password">
            <button class="btn btn-purple" onclick="login()">Login</button>
            <button class="btn-link" onclick="showForgot()">Forgot Password?</button>
        </div>
    </div>

    <div id="forgot-view" class="hidden">
        <div class="card">
            <h3>Admin Reset</h3>
            <p style="font-size: 0.8rem;">OTP sent to Administrator:<br><b>+91 7990472413</b></p>
            <button class="btn btn-gold" id="otp-btn" onclick="sendOTP()">Send OTP</button>
            <div id="otp-box" class="hidden">
                <input type="number" id="otp-input" placeholder="4-Digit Code">
                <button class="btn btn-purple" onclick="verifyAndReset()">Verify & Reset Admin Credentials</button>
            </div>
            <button class="btn-link" onclick="location.reload()">Back</button>
        </div>
    </div>

    <div id="owner-view" class="hidden">
        <div class="card">
            <h3>Organization Reports</h3>
            <select id="m-select">
                <option value="0">January</option><option value="1">February</option><option value="2">March</option>
                <option value="3">April</option><option value="4">May</option><option value="5">June</option>
                <option value="6">July</option><option value="7">August</option><option value="8">September</option>
                <option value="9">October</option><option value="10">November</option><option value="11">December</option>
            </select>
            <button class="btn btn-gold" onclick="exportCSV()">Download Full Report (Excel)</button>
        </div>
        <div class="card">
            <h3>Staff & Salary Slips</h3>
            <div class="table-wrap">
                <table>
                    <thead><tr><th>Code</th><th>Name</th><th>Salary</th><th>Action</th></tr></thead>
                    <tbody id="staff-list"></tbody>
                </table>
            </div>
        </div>
        <button class="btn btn-purple" onclick="location.reload()">Logout</button>
    </div>

    <div id="emp-view" class="hidden">
        <div class="card">
            <h2 id="greet">Welcome</h2>
            <div style="display:flex; gap:10px">
                <button class="btn btn-purple" style="flex:1" onclick="punch('IN')">IN</button>
                <button class="btn btn-gold" style="flex:1" onclick="punch('OUT')">OUT</button>
            </div>
        </div>
        <div class="table-wrap">
            <table id="personal-table">
                <thead><tr><th>Date</th><th>In</th><th>Out</th><th>Status</th></tr></thead>
                <tbody id="personal-body"></tbody>
            </table>
        </div>
        <button class="btn btn-purple" onclick="location.reload()" style="margin-top:15px">Logout</button>
    </div>
</div>

<script>
    const LOGO = "https://i.postimg.cc/mD8zW0bN/heritage-auto-finance-logo-removebg-preview.png";
    let emps = JSON.parse(localStorage.getItem('h_emps')) || [];
    let logs = JSON.parse(localStorage.getItem('h_logs')) || [];
    let auth = JSON.parse(localStorage.getItem('h_auth')) || {u:'admin', p:'admin123'};
    let user = null;
    let otp = null;

    function login() {
        const r = document.getElementById('role').value;
        const u = document.getElementById('u-user').value;
        const p = document.getElementById('u-pass').value;
        const c = document.getElementById('u-code').value;

        if(r === 'owner') {
            if(u === auth.u && p === auth.p) { show('owner-view'); renderStaff(); }
            else alert("Invalid Admin Login");
        } else {
            const e = emps.find(x => x.code === c && x.pass === p && x.name === u);
            if(e) { user = e; show('emp-view'); document.getElementById('greet').innerText = "Hello, " + e.name; renderSelf(); }
            else alert("Staff not found");
        }
    }

    function sendOTP() {
        otp = Math.floor(1000 + Math.random() * 9000);
        alert("HERITAGE FINANCE AUTH:\nOTP Sent to +91 7990472413\n\nCode: " + otp);
        document.getElementById('otp-btn').classList.add('hidden');
        document.getElementById('otp-box').classList.remove('hidden');
    }

    function verifyAndReset() {
        if(document.getElementById('otp-input').value == otp) {
            auth = {u:'admin', p:'admin123'};
            localStorage.setItem('h_auth', JSON.stringify(auth));
            alert("Admin Reset to Default!\nUser: admin\nPass: admin123");
            location.reload();
        } else alert("Invalid OTP");
    }

    function punch(type) {
        const now = new Date();
        const date = now.toISOString().split('T')[0];
        const time = now.getHours() + ":" + now.getMinutes().toString().padStart(2, '0');
        let record = logs.find(l => l.code === user.code && l.date === date);

        if(type === 'IN') {
            if(record) return alert("Already marked IN");
            const late = (now.getHours() > 10 || (now.getHours() == 10 && now.getMinutes() > 30));
            logs.push({ code: user.code, name: user.name, date, month: now.getMonth(), in: time, out: null, status: late ? 'Half Day' : 'Present' });
        } else {
            if(!record) return alert("Mark IN first");
            record.out = time;
        }
        localStorage.setItem('h_logs', JSON.stringify(logs)); renderSelf();
    }

    function exportCSV() {
        const m = parseInt(document.getElementById('m-select').value);
        let csv = "Date,Name,Code,In,Out,Status\n";
        logs.filter(l => l.month === m).forEach(l => { csv += `${l.date},${l.name},${l.code},${l.in},${l.out||'-'},${l.status}\n`; });
        const a = document.createElement('a');
        a.href = URL.createObjectURL(new Blob([csv], {type:'text/csv'}));
        a.download = "Heritage_Report.csv"; a.click();
    }

    function genSlip(code) {
        const e = emps.find(x => x.code === code);
        const m = parseInt(document.getElementById('m-select').value);
        const myLogs = logs.filter(l => l.code === code && l.month === m);
        const pres = myLogs.filter(l => l.status === 'Present').length;
        const half = myLogs.filter(l => l.status === 'Half Day').length;
        const net = (pres * (e.salary/30)) + (half * (e.salary/30) * 0.5);

        const { jsPDF } = window.jspdf;
        const doc = new jsPDF();
        doc.addImage(LOGO, 'PNG', 85, 10, 40, 40);
        doc.setFontSize(14); doc.text("HERITAGE AUTO FINANCE", 105, 55, {align:'center'});
        doc.autoTable({
            startY: 65,
            body: [['Name', e.name], ['ID', e.code], ['Month', document.getElementById('m-select').options[m].text], ['Salary', e.salary], ['Net Pay', net.toFixed(2)]],
            theme: 'grid', headStyles: {fillColor: [197, 157, 95]}
        });
        doc.save(`Slip_${e.name}.pdf`);
    }

    function show(id) { document.getElementById('login-view').classList.add('hidden'); document.getElementById(id).classList.remove('hidden'); }
    function showForgot() { document.getElementById('login-view').classList.add('hidden'); document.getElementById('forgot-view').classList.remove('hidden'); }
    function renderStaff() { document.getElementById('staff-list').innerHTML = emps.map(e => `<tr><td>${e.code}</td><td>${e.name}</td><td>${e.salary}</td><td><button onclick="genSlip('${e.code}')">PDF</button></td></tr>`).join(''); }
    function renderSelf() { document.getElementById('personal-body').innerHTML = logs.filter(l => l.code === user.code).map(l => `<tr><td>${l.date}</td><td>${l.in}</td><td>${l.out||'-'}</td><td>${l.status}</td></tr>`).join(''); }
</script>
</body>
</html>
