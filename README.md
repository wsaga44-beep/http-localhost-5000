# http-localhost-5000
import { useState, useEffect } from "react";

const API_BASE = "http://localhost:5000";

// ── Colour tokens ──────────────────────────────────────────
const C = {
  navy:    "#0F1E3C",
  navyMid: "#1A2F54",
  gold:    "#F5A623",
  goldDim: "#B87800",
  green:   "#00C48C",
  red:     "#FF4D6A",
  bg:      "#0A1628",
  surface: "#111E35",
  border:  "#1E3050",
  textPri: "#E8EEF8",
  textSec: "#7A8FAD",
  white:   "#FFFFFF",
};

// ── Reusable atoms ─────────────────────────────────────────
const styles = {
  app: {
    minHeight: "100vh",
    background: C.bg,
    fontFamily: "'Inter', 'SF Pro Display', system-ui, sans-serif",
    color: C.textPri,
    maxWidth: 480,
    margin: "0 auto",
    position: "relative",
  },
  card: {
    background: C.surface,
    border: `1px solid ${C.border}`,
    borderRadius: 16,
    padding: 20,
    marginBottom: 14,
  },
  input: {
    width: "100%",
    background: C.navy,
    border: `1px solid ${C.border}`,
    borderRadius: 10,
    color: C.textPri,
    fontSize: 15,
    padding: "12px 14px",
    outline: "none",
    boxSizing: "border-box",
    marginBottom: 12,
  },
  btnPrimary: {
    width: "100%",
    background: `linear-gradient(135deg, ${C.gold}, ${C.goldDim})`,
    border: "none",
    borderRadius: 12,
    color: C.navy,
    fontSize: 16,
    fontWeight: 700,
    padding: "14px 0",
    cursor: "pointer",
    letterSpacing: 0.3,
  },
  btnSecondary: {
    width: "100%",
    background: "transparent",
    border: `1px solid ${C.gold}`,
    borderRadius: 12,
    color: C.gold,
    fontSize: 15,
    fontWeight: 600,
    padding: "13px 0",
    cursor: "pointer",
    marginTop: 10,
  },
  btnDanger: {
    background: "transparent",
    border: `1px solid ${C.red}`,
    borderRadius: 10,
    color: C.red,
    fontSize: 14,
    fontWeight: 600,
    padding: "10px 18px",
    cursor: "pointer",
  },
  label: {
    fontSize: 12,
    color: C.textSec,
    letterSpacing: 1,
    textTransform: "uppercase",
    marginBottom: 6,
    display: "block",
  },
  tag: (color) => ({
    display: "inline-block",
    background: color + "22",
    border: `1px solid ${color}44`,
    borderRadius: 6,
    color,
    fontSize: 11,
    fontWeight: 700,
    padding: "3px 8px",
    letterSpacing: 0.5,
  }),
  navBar: {
    position: "fixed",
    bottom: 0,
    left: "50%",
    transform: "translateX(-50%)",
    width: "100%",
    maxWidth: 480,
    background: C.navyMid,
    borderTop: `1px solid ${C.border}`,
    display: "flex",
    justifyContent: "space-around",
    padding: "10px 0 14px",
    zIndex: 100,
  },
  navItem: (active) => ({
    display: "flex",
    flexDirection: "column",
    alignItems: "center",
    gap: 3,
    cursor: "pointer",
    color: active ? C.gold : C.textSec,
    fontSize: 11,
    fontWeight: active ? 700 : 400,
    border: "none",
    background: "none",
    padding: "4px 12px",
  }),
};

// ── Mock API (works offline; replace calls with real fetch) ─
let mockDB = {
  customers: [],
  trades: [],
  nextId: 1,
};

const PACKAGES = {
  free:    { name: "FREE",    trades: 3,    price: 0,    features: ["3 trades/day", "Basic AI"] },
  micro:   { name: "MICRO",   trades: 5,    price: 50,   original: 500, disc: "90% OFF", features: ["5 trades/day", "90% discount"] },
  student: { name: "STUDENT", trades: 10,   price: 100,  original: 500, disc: "80% OFF", features: ["10 trades/day", "80% discount"] },
  bronze:  { name: "BRONZE",  trades: 20,   price: 500,  features: ["20 trades/day", "Daily info"] },
  silver:  { name: "SILVER",  trades: 50,   price: 1000, features: ["50 trades/day", "Priority"] },
  gold:    { name: "GOLD",    trades: 9999, price: 2500, features: ["Unlimited trades", "All features", "24/7 support"] },
};

function hashPass(p) {
  // Simple demo hash (real app uses backend SHA-256)
  return btoa(p + "_hashed");
}

const api = {
  register: (email, password, name, profitAcc) => {
    if (mockDB.customers.find((c) => c.email === email))
      return { success: false, message: "Email already registered" };
    const cust = {
      id: mockDB.nextId++, email, password: hashPass(password),
      name, profit_account: profitAcc, money: 0,
      total_profit: 0, package: "free", trades_today: 0,
    };
    mockDB.customers.push(cust);
    return { success: true, message: "Registered!", customer_id: cust.id };
  },
  login: (email, password) => {
    const c = mockDB.customers.find(
      (x) => x.email === email && x.password === hashPass(password)
    );
    if (!c) return { success: false, message: "Wrong email / password" };
    const pkg = PACKAGES[c.package];
    return { success: true, customer: { ...c, package_info: pkg, trades_left: pkg.trades - c.trades_today } };
  },
  addMoney: (id, amount) => {
    if (amount <= 0) return { success: false, message: "Amount must be > 0" };
    const c = mockDB.customers.find((x) => x.id === id);
    if (!c) return { success: false, message: "Not found" };
    c.money += amount;
    return { success: true, money: c.money };
  },
  buyPackage: (id, pkg) => {
    const c = mockDB.customers.find((x) => x.id === id);
    const info = PACKAGES[pkg];
    if (!info) return { success: false, message: "Invalid package" };
    if (c.money < info.price)
      return { success: false, message: `Need ₹${info.price}, you have ₹${c.money}` };
    c.money -= info.price;
    c.package = pkg;
    c.trades_today = 0;
    return { success: true, message: `${info.name} activated!`, package_info: info };
  },
  aiTrade: (id) => {
    const c = mockDB.customers.find((x) => x.id === id);
    if (!c) return { success: false, message: "Not found" };
    const pkg = PACKAGES[c.package];
    if (c.trades_today >= pkg.trades)
      return { success: false, message: `Daily limit reached (${pkg.trades}/day)` };
    if (c.money === 0)
      return { success: false, message: "No money! Add money first." };

    // Simulate AI logic
    const companies = ["RELIANCE.NS", "TCS.NS", "INFY.NS", "HDFCBANK.NS"];
    const prices = { "RELIANCE.NS": 2845.5, "TCS.NS": 3450, "INFY.NS": 1780, "HDFCBANK.NS": 1620 };
    const newsTypes = ["GOOD", "NEUTRAL", "BAD"];
    const picked = companies[Math.floor(Math.random() * companies.length)];
    const news = newsTypes[Math.floor(Math.random() * 2)]; // 50/50 GOOD or NEUTRAL

    const invest = c.money * 0.8;
    const backup = c.money * 0.2;
    c.trades_today += 1;

    const trade = {
      id: mockDB.trades.length + 1,
      customer_id: id,
      ticker: picked,
      price: prices[picked],
      news_type: news,
      invest_amount: invest,
      backup_amount: backup,
      status: "ACTIVE",
      traded_at: new Date().toLocaleString(),
      decision: "TRADE",
    };
    mockDB.trades.push(trade);

    return {
      success: true,
      trade,
      message: `80% invested (₹${invest.toFixed(2)}), 20% backup (₹${backup.toFixed(2)})`,
      trades_left: pkg.trades - c.trades_today,
    };
  },
  getAccount: (id) => {
    const c = mockDB.customers.find((x) => x.id === id);
    if (!c) return { success: false };
    const pkg = PACKAGES[c.package];
    return { success: true, account: { ...c, package_info: pkg, trades_left: pkg.trades - c.trades_today } };
  },
  getTrades: (id) => ({
    success: true,
    trades: mockDB.trades.filter((t) => t.customer_id === id).reverse(),
  }),
};

// ── Screen components ──────────────────────────────────────

function LoginScreen({ onLogin, onGoRegister }) {
  const [email, setEmail] = useState("");
  const [pass, setPass] = useState("");
  const [err, setErr] = useState("");
  const [loading, setLoading] = useState(false);

  const submit = () => {
    setErr("");
    if (!email || !pass) return setErr("Fill all fields");
    setLoading(true);
    setTimeout(() => {
      const r = api.login(email, pass);
      setLoading(false);
      if (r.success) onLogin(r.customer);
      else setErr(r.message);
    }, 600);
  };

  return (
    <div style={{ padding: "60px 24px 30px" }}>
      {/* Logo */}
      <div style={{ textAlign: "center", marginBottom: 40 }}>
        <div style={{ fontSize: 52, marginBottom: 8 }}>🤖</div>
        <h1 style={{ margin: 0, fontSize: 26, fontWeight: 800, color: C.gold }}>
          AI Trading
        </h1>
        <p style={{ margin: "6px 0 0", color: C.textSec, fontSize: 14 }}>
          Smart investing, automated
        </p>
      </div>

      <div style={styles.card}>
        <label style={styles.label}>Email</label>
        <input
          style={styles.input}
          type="email"
          placeholder="your@email.com"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
        />
        <label style={styles.label}>Password</label>
        <input
          style={styles.input}
          type="password"
          placeholder="••••••••"
          value={pass}
          onChange={(e) => setPass(e.target.value)}
        />
        {err && (
          <p style={{ color: C.red, fontSize: 13, margin: "-4px 0 10px" }}>{err}</p>
        )}
        <button style={styles.btnPrimary} onClick={submit} disabled={loading}>
          {loading ? "Signing in…" : "Sign In"}
        </button>
        <button style={styles.btnSecondary} onClick={onGoRegister}>
          Create Account
        </button>
      </div>

      <p style={{ textAlign: "center", color: C.textSec, fontSize: 12, marginTop: 16 }}>
        Demo mode — no real trades are executed
      </p>
    </div>
  );
}

function RegisterScreen({ onBack }) {
  const [form, setForm] = useState({ email: "", password: "", name: "", profit_account: "" });
  const [err, setErr] = useState("");
  const [success, setSuccess] = useState(false);

  const set = (k) => (e) => setForm({ ...form, [k]: e.target.value });

  const submit = () => {
    if (!form.email || !form.password || !form.name)
      return setErr("Fill all required fields");
    const r = api.register(form.email, form.password, form.name, form.profit_account);
    if (r.success) setSuccess(true);
    else setErr(r.message);
  };

  if (success)
    return (
      <div style={{ padding: "80px 24px", textAlign: "center" }}>
        <div style={{ fontSize: 60, marginBottom: 16 }}>✅</div>
        <h2 style={{ color: C.green }}>Account Created!</h2>
        <p style={{ color: C.textSec }}>You can now sign in.</p>
        <button style={{ ...styles.btnPrimary, marginTop: 20 }} onClick={onBack}>
          Go to Sign In
        </button>
      </div>
    );

  return (
    <div style={{ padding: "40px 24px 30px" }}>
      <button onClick={onBack} style={{ background: "none", border: "none", color: C.gold, fontSize: 15, cursor: "pointer", marginBottom: 16 }}>
        ← Back
      </button>
      <h2 style={{ margin: "0 0 20px", fontSize: 22, fontWeight: 800 }}>Create Account</h2>
      <div style={styles.card}>
        {["name", "email", "password", "profit_account"].map((k) => (
          <div key={k}>
            <label style={styles.label}>{k === "profit_account" ? "Profit Account (email/bank)" : k}</label>
            <input
              style={styles.input}
              type={k === "password" ? "password" : "email".includes(k) ? "email" : "text"}
              placeholder={k === "name" ? "Full Name" : k === "profit_account" ? "Optional" : ""}
              value={form[k]}
              onChange={set(k)}
            />
          </div>
        ))}
        {err && <p style={{ color: C.red, fontSize: 13, margin: "-4px 0 10px" }}>{err}</p>}
        <button style={styles.btnPrimary} onClick={submit}>Register</button>
      </div>
    </div>
  );
}

function HomeScreen({ customer, onRefresh }) {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);
  const acc = api.getAccount(customer.id).account;
  const pkg = acc.package_info;

  const doTrade = () => {
    setLoading(true);
    setResult(null);
    setTimeout(() => {
      const r = api.aiTrade(customer.id);
      setLoading(false);
      setResult(r);
      onRefresh();
    }, 1800);
  };

  return (
    <div style={{ padding: "24px 20px 90px" }}>
      {/* Header */}
      <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 20 }}>
        <div>
          <p style={{ margin: 0, color: C.textSec, fontSize: 12 }}>Welcome back</p>
          <h2 style={{ margin: 0, fontSize: 20, fontWeight: 800 }}>{acc.name}</h2>
        </div>
        <span style={styles.tag(C.gold)}>{pkg.name}</span>
      </div>

      {/* Balance card */}
      <div style={{ ...styles.card, background: `linear-gradient(135deg, ${C.navyMid}, #162040)`, border: `1px solid ${C.gold}33` }}>
        <p style={{ margin: "0 0 4px", color: C.textSec, fontSize: 12, letterSpacing: 1, textTransform: "uppercase" }}>Total Balance</p>
        <p style={{ margin: "0 0 20px", fontSize: 34, fontWeight: 800, color: C.white }}>
          ₹{acc.money.toLocaleString("en-IN", { minimumFractionDigits: 2 })}
        </p>
        <div style={{ display: "flex", gap: 12 }}>
          <div style={{ flex: 1, background: C.green + "18", border: `1px solid ${C.green}33`, borderRadius: 10, padding: 12, textAlign: "center" }}>
            <p style={{ margin: 0, color: C.green, fontSize: 18, fontWeight: 700 }}>
              ₹{(acc.money * 0.8).toFixed(0)}
            </p>
            <p style={{ margin: "4px 0 0", color: C.textSec, fontSize: 11 }}>80% Trading</p>
          </div>
          <div style={{ flex: 1, background: C.gold + "18", border: `1px solid ${C.gold}33`, borderRadius: 10, padding: 12, textAlign: "center" }}>
            <p style={{ margin: 0, color: C.gold, fontSize: 18, fontWeight: 700 }}>
              ₹{(acc.money * 0.2).toFixed(0)}
            </p>
            <p style={{ margin: "4px 0 0", color: C.textSec, fontSize: 11 }}>20% Backup</p>
          </div>
        </div>
      </div>

      {/* Stats row */}
      <div style={{ display: "flex", gap: 12, marginBottom: 14 }}>
        <div style={{ ...styles.card, flex: 1, marginBottom: 0, textAlign: "center" }}>
          <p style={{ margin: 0, fontSize: 18, fontWeight: 700, color: C.green }}>
            ₹{acc.total_profit.toFixed(0)}
          </p>
          <p style={{ margin: "4px 0 0", color: C.textSec, fontSize: 11 }}>Total Profit</p>
        </div>
        <div style={{ ...styles.card, flex: 1, marginBottom: 0, textAlign: "center" }}>
          <p style={{ margin: 0, fontSize: 18, fontWeight: 700, color: C.gold }}>
            {acc.trades_left === 9999 ? "∞" : acc.trades_left}
          </p>
          <p style={{ margin: "4px 0 0", color: C.textSec, fontSize: 11 }}>Trades Left</p>
        </div>
      </div>

      {/* AI Trade button */}
      <button
        style={{
          ...styles.btnPrimary,
          padding: "18px 0",
          fontSize: 18,
          borderRadius: 16,
          opacity: loading ? 0.7 : 1,
          boxShadow: loading ? "none" : `0 4px 24px ${C.gold}44`,
          marginBottom: 16,
        }}
        onClick={doTrade}
        disabled={loading}
      >
        {loading ? "🤖 AI Analysing…" : "🚀 AI TRADE NOW"}
      </button>

      {/* Trade result */}
      {result && (
        <div style={{
          ...styles.card,
          border: `1px solid ${result.success && result.trade?.decision === "TRADE" ? C.green : C.red}44`,
          animation: "fadeIn 0.4s ease",
        }}>
          {result.success && result.trade?.decision === "TRADE" ? (
            <>
              <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 12 }}>
                <span style={{ fontWeight: 700, fontSize: 16, color: C.green }}>✅ Trade Successful</span>
                <span style={styles.tag(C.green)}>{result.trade.news_type}</span>
              </div>
              <p style={{ margin: "0 0 4px", color: C.textSec, fontSize: 13 }}>Company</p>
              <p style={{ margin: "0 0 12px", fontWeight: 700, fontSize: 18 }}>{result.trade.ticker}</p>
              <p style={{ margin: "0 0 4px", color: C.textSec, fontSize: 13 }}>Price</p>
              <p style={{ margin: "0 0 12px", fontWeight: 700 }}>₹{result.trade.price.toLocaleString("en-IN")}</p>
              <div style={{ display: "flex", gap: 10 }}>
                <div style={{ flex: 1, background: C.green + "18", borderRadius: 8, padding: "8px 10px" }}>
                  <p style={{ margin: 0, color: C.green, fontWeight: 700 }}>₹{result.trade.invest_amount.toFixed(0)}</p>
                  <p style={{ margin: 0, color: C.textSec, fontSize: 11 }}>Trading (80%)</p>
                </div>
                <div style={{ flex: 1, background: C.gold + "18", borderRadius: 8, padding: "8px 10px" }}>
                  <p style={{ margin: 0, color: C.gold, fontWeight: 700 }}>₹{result.trade.backup_amount.toFixed(0)}</p>
                  <p style={{ margin: 0, color: C.textSec, fontSize: 11 }}>Backup (20%)</p>
                </div>
              </div>
            </>
          ) : (
            <div style={{ textAlign: "center", padding: "8px 0" }}>
              <div style={{ fontSize: 32, marginBottom: 8 }}>😞</div>
              <p style={{ margin: 0, fontWeight: 700, color: C.red }}>{result.message}</p>
              <p style={{ margin: "6px 0 0", color: C.textSec, fontSize: 13 }}>Your money is 100% safe.</p>
            </div>
          )}
        </div>
      )}
    </div>
  );
}

function AddMoneyScreen({ customer, onSuccess }) {
  const [amount, setAmount] = useState("");
  const [msg, setMsg] = useState(null);
  const quick = [500, 1000, 5000, 10000];

  const add = (val) => {
    const n = Number(val);
    const r = api.addMoney(customer.id, n);
    if (r.success) {
      setMsg({ type: "ok", text: `₹${n} added! Balance: ₹${r.money}` });
      setAmount("");
      onSuccess();
    } else {
      setMsg({ type: "err", text: r.message });
    }
  };

  return (
    <div style={{ padding: "24px 20px 90px" }}>
      <h2 style={{ marginBottom: 20 }}>Add Money</h2>
      <div style={styles.card}>
        <label style={styles.label}>Enter Amount (₹)</label>
        <input
          style={styles.input}
          type="number"
          placeholder="0"
          value={amount}
          onChange={(e) => setAmount(e.target.value)}
          min="1"
        />
        <div style={{ display: "flex", gap: 8, flexWrap: "wrap", marginBottom: 16 }}>
          {quick.map((q) => (
            <button
              key={q}
              style={{
                background: C.navy,
                border: `1px solid ${C.border}`,
                borderRadius: 8,
                color: C.textPri,
                fontSize: 13,
                fontWeight: 600,
                padding: "8px 14px",
                cursor: "pointer",
              }}
              onClick={() => { setAmount(q); }}
            >
              ₹{q.toLocaleString()}
            </button>
          ))}
        </div>
        {msg && (
          <p style={{ color: msg.type === "ok" ? C.green : C.red, fontSize: 13, marginBottom: 10 }}>
            {msg.text}
          </p>
        )}
        <button style={styles.btnPrimary} onClick={() => add(amount)} disabled={!amount || Number(amount) <= 0}>
          Add Money
        </button>
      </div>

      <div style={styles.card}>
        <p style={{ margin: "0 0 12px", fontWeight: 700, fontSize: 15 }}>Payment Methods</p>
        {["UPI / GPay / PhonePe", "Credit / Debit Card", "Net Banking"].map((m) => (
          <div
            key={m}
            style={{
              display: "flex",
              alignItems: "center",
              gap: 10,
              padding: "12px 0",
              borderBottom: `1px solid ${C.border}`,
            }}
          >
            <span style={{ fontSize: 20 }}>{m.includes("UPI") ? "📱" : m.includes("Card") ? "💳" : "🏦"}</span>
            <span style={{ fontSize: 14, color: C.textSec }}>{m}</span>
          </div>
        ))}
      </div>
    </div>
  );
}

function PackagesScreen({ customer, onSuccess }) {
  const [buyin
