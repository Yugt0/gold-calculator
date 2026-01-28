<script setup>
import { ref, reactive } from 'vue'

/* ================= LOGIN ================= */
const isAuthenticated = ref(false)
const username = ref('')
const password = ref('')
const error = ref('')

function login() {
  if (username.value === 'admin' && password.value === 'admin123') {
    isAuthenticated.value = true
    localStorage.setItem('auth', 'true')
    error.value = ''
  } else {
    error.value = 'Invalid username or password'
  }
}

function logout() {
  isAuthenticated.value = false
  localStorage.removeItem('auth')
}

/* ================= CALCULATORS ================= */
const vatRate = 0.12

// Six independent calculators
const calculators = reactive([
  { label: '24K (99.9%)', price: 9500, weight: 1, designFee: 0, color: '#FFD700' },
  { label: '23K (95.8%)', price: 9100, weight: 1, designFee: 0, color: '#FFCC00' },
  { label: '22K (91.6%)', price: 8700, weight: 1, designFee: 0, color: '#FFC107' },
  { label: '18K (75.0%)', price: 7100, weight: 1, designFee: 0, color: '#FFB300' },
  { label: '14K (58.3%)', price: 5500, weight: 1, designFee: 0, color: '#FF8F00' },
  { label: '10K (41.7%)', price: 3900, weight: 1, designFee: 0, color: '#FF6F00' }
])

// Computation functions
function computeBase(calc) {
  return calc.price * calc.weight
}

function computeSubtotal(calc) {
  return computeBase(calc) + Number(calc.designFee)
}

function computeVAT(calc) {
  return computeSubtotal(calc) * vatRate
}

function computeTotal(calc) {
  return computeSubtotal(calc) + computeVAT(calc)
}

// Format number with commas
function formatNumber(num) {
  return num.toLocaleString('en-PH', { minimumFractionDigits: 2, maximumFractionDigits: 2 })
}
</script>

<template>
  <div class="page">
    <!-- LOGIN -->
    <div v-if="!isAuthenticated" class="login-card">
      <h2>Admin Login</h2>
      <input placeholder="Username" v-model="username" />
      <input type="password" placeholder="Password" v-model="password" />
      <p v-if="error" class="error">{{ error }}</p>
      <button @click="login">Login</button>
    </div>

    <!-- CALCULATORS -->
    <div v-else class="calculators-container">
      <div class="logout-container">
        <button class="logout" @click="logout">Logout</button>
      </div>

      <div class="calculators-grid">
        <div
          class="calculator-card"
          v-for="calc in calculators"
          :key="calc.label"
          :style="{ borderTop: '4px solid ' + calc.color }"
        >
          <h2>{{ calc.label }}</h2>
          <p class="market-price">₱{{ calc.price.toLocaleString() }} / gram</p>

          <div class="form">
            <div class="input-group">
              <label>Weight (grams)</label>
              <input type="number" v-model.number="calc.weight" min="0.1" step="0.1"/>
            </div>
            <div class="input-group">
              <label>Design Fee (PHP)</label>
              <input type="number" v-model.number="calc.designFee" min="0"/>
            </div>
          </div>

          <div class="divider"></div>

          <div class="summary">
            <div>
              <span>Base Value</span>
              <span>₱{{ formatNumber(computeBase(calc)) }}</span>
            </div>
            <div>
              <span>VAT (12%)</span>
              <span>₱{{ formatNumber(computeVAT(calc)) }}</span>
            </div>
            <div class="total">
              <span>Total</span>
              <span>₱{{ formatNumber(computeTotal(calc)) }}</span>
            </div>
          </div>

          <button class="quote-button">Generate Quote</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
* { box-sizing: border-box; }
body { margin: 0; font-family: 'Inter', system-ui, sans-serif; background: #f5f6fa; color: #111827; }

.page {
  min-height: 100vh;
  padding: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center; /* centers vertically */
}

/* Login Card */
.login-card {
  width: 400px;
  background: #fff;
  padding: 32px;
  border-radius: 16px;
  box-shadow: 0 10px 30px rgba(0,0,0,0.08);
  text-align: center;
}
.login-card h2 { margin-bottom: 20px; font-weight: 700; font-size: 1.4rem; color: #333; }
.login-card input { width: 100%; padding: 12px 14px; margin-bottom: 16px; border-radius: 8px; border: 1px solid #d1d5db; }
.login-card input:focus { outline: none; border-color: #f59e0b; box-shadow: 0 0 5px rgba(245,158,11,0.3); }
.login-card button { width: 100%; padding: 12px; background: #f59e0b; border: none; border-radius: 10px; font-weight: 700; cursor: pointer; color: #fff; font-size: 1rem; transition: background 0.3s; }
.login-card button:hover { background: #d97706; }
.error { color: #dc2626; font-size: 13px; margin-bottom: 10px; }

/* Logout */
.logout-container { width: 100%; display: flex; justify-content: flex-end; margin-bottom: 20px; }
.logout { background: none; border: none; color: #dc2626; font-weight: 600; cursor: pointer; transition: color 0.3s; }
.logout:hover { color: #a91a1a; }

/* Grid for Calculators */
.calculators-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 20px;
  width: 100%;
  max-width: 1400px;
  justify-items: center; /* centers cards horizontally */
}

/* Individual Calculator Card */
.calculator-card {
  background: #ffffff;
  padding: 24px;
  border-radius: 12px;
  border: 1px solid #e5e7eb;
  box-shadow: 0 4px 12px rgba(0,0,0,0.05);
  text-align: center;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  transition: transform 0.3s, box-shadow 0.3s;
  width: 100%;
  max-width: 250px;
}
.calculator-card:hover { transform: translateY(-4px); box-shadow: 0 8px 24px rgba(0,0,0,0.12); }

.calculator-card h2 { font-size: 1.1rem; margin: 0 0 6px 0; font-weight: 700; color: #222; }
.market-price { font-size: 13px; color: #6b7280; margin-bottom: 14px; font-weight: 500; }

.form { display: grid; gap: 12px; margin-bottom: 16px; }
.input-group { display: flex; flex-direction: column; text-align: left; }
.input-group label { font-size: 12px; font-weight: 600; color: #374151; margin-bottom: 4px; }
.input-group input { padding: 10px; border-radius: 6px; border: 1px solid #d1d5db; }
.input-group input:focus { outline: none; border-color: #f59e0b; box-shadow: 0 0 4px rgba(245,158,11,0.2); }

.divider { height: 1px; background: #e5e7eb; margin: 12px 0; }

.summary div { display: flex; justify-content: space-between; margin-bottom: 8px; font-size: 14px; font-weight: 500; }
.summary .total { font-weight: 700; font-size: 1rem; color: #16a34a; }

.quote-button {
  margin-top: 10px;
  padding: 10px 0;
  width: 100%;
  background: #f59e0b;
  border: none;
  border-radius: 8px;
  font-weight: 600;
  color: #fff;
  cursor: pointer;
  transition: background 0.3s;
}
.quote-button:hover { background: #d97706; }
</style>
