<script setup>
import { ref, onMounted } from 'vue'

const API = 'http://localhost:4000/api/items'
const items = ref([])
const form = ref({ name: '', description: '', price: '' })
const editId = ref(null)
const loading = ref(false)

async function load() {
  loading.value = true
  items.value = await fetch(API).then(r => r.json())
  loading.value = false
}

async function save() {
  const payload = { ...form.value, price: parseFloat(form.value.price) || null }
  if (editId.value) {
    await fetch(`${API}/${editId.value}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload)
    })
    editId.value = null
  } else {
    await fetch(API, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload)
    })
  }
  form.value = { name: '', description: '', price: '' }
  load()
}

function startEdit(item) {
  editId.value = item.id
  form.value = { name: item.name, description: item.description, price: item.price ?? '' }
}

function cancelEdit() {
  editId.value = null
  form.value = { name: '', description: '', price: '' }
}

async function remove(id) {
  await fetch(`${API}/${id}`, { method: 'DELETE' })
  load()
}

onMounted(load)
</script>

<template>
  <div class="app">
    <link href="https://fonts.googleapis.com/css2?family=DM+Serif+Display:ital@0;1&family=DM+Mono:wght@400;500&display=swap" rel="stylesheet" />

    <header class="header">
      <div class="header-inner">
        <div class="logo">
          <span class="logo-mark">◈</span>
          <span class="logo-text">Inventory</span>
        </div>
        <div class="header-badge">{{ items.length }} items</div>
      </div>
    </header>

    <main class="main">
      <section class="form-section">
        <div class="form-card">
          <h2 class="form-title">{{ editId ? 'Edit Item' : 'Add New Item' }}</h2>
          <form @submit.prevent="save" class="form">
            <div class="field">
              <label class="label">Name</label>
              <input class="input" v-model="form.name" placeholder="Item name" required />
            </div>
            <div class="field">
              <label class="label">Description</label>
              <input class="input" v-model="form.description" placeholder="Short description" />
            </div>
            <div class="field">
              <label class="label">Price</label>
              <div class="price-wrap">
                <span class="currency">₱</span>
                <input class="input price-input" v-model="form.price" type="number" step="0.01" min="0" placeholder="0.00" />
              </div>
            </div>
            <div class="form-actions">
              <button v-if="editId" type="button" class="btn btn-ghost" @click="cancelEdit">Cancel</button>
              <button type="submit" class="btn btn-primary">{{ editId ? 'Save Changes' : 'Add Item' }}</button>
            </div>
          </form>
        </div>
      </section>

      <section class="list-section">
        <div v-if="loading" class="empty-state">
          <span class="spinner">◈</span>
        </div>
        <div v-else-if="items.length === 0" class="empty-state">
          <p class="empty-text">No items yet. Add your first one.</p>
        </div>
        <div v-else class="table-wrap">
          <table class="table">
            <thead>
              <tr>
                <th>Name</th>
                <th>Description</th>
                <th class="th-price">Price</th>
                <th class="th-actions">Actions</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in items" :key="item.id" :class="{ 'row-editing': editId === item.id }">
                <td class="td-name">{{ item.name }}</td>
                <td class="td-desc">{{ item.description || '—' }}</td>
                <td class="td-price">
                  <span class="price-tag" v-if="item.price != null">
                    ₱{{ Number(item.price).toFixed(2) }}
                  </span>
                  <span class="no-price" v-else>—</span>
                </td>
                <td class="td-actions">
                  <button class="action-btn edit-btn" @click="startEdit(item)">Edit</button>
                  <button class="action-btn del-btn" @click="remove(item.id)">Delete</button>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>
    </main>
  </div>
</template>

<style>
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

:root {
  --bg: #0f0e0d;
  --surface: #1a1916;
  --border: #2e2c28;
  --accent: #d4a843;
  --accent-dim: #a07c2e;
  --text: #e8e4dc;
  --text-dim: #7a756b;
  --danger: #c0392b;
  --radius: 4px;
}

body {
  background: var(--bg);
  color: var(--text);
  font-family: 'DM Mono', monospace;
  font-size: 14px;
  min-height: 100vh;
}

.app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

/* Header */
.header {
  border-bottom: 1px solid var(--border);
  padding: 0 2rem;
  height: 56px;
  display: flex;
  align-items: center;
  position: sticky;
  top: 0;
  background: var(--bg);
  z-index: 10;
}
.header-inner {
  width: 100%;
  max-width: 1100px;
  margin: 0 auto;
  display: flex;
  align-items: center;
  justify-content: space-between;
}
.logo {
  display: flex;
  align-items: center;
  gap: 10px;
}
.logo-mark {
  color: var(--accent);
  font-size: 20px;
  animation: spin 8s linear infinite;
}
@keyframes spin { to { transform: rotate(360deg); } }
.logo-text {
  font-family: 'DM Serif Display', serif;
  font-size: 20px;
  letter-spacing: 0.02em;
  color: var(--text);
}
.header-badge {
  font-size: 11px;
  padding: 3px 10px;
  border: 1px solid var(--border);
  border-radius: 99px;
  color: var(--text-dim);
  letter-spacing: 0.05em;
}

/* Main layout */
.main {
  max-width: 1100px;
  margin: 0 auto;
  padding: 2.5rem 2rem;
  width: 100%;
  display: grid;
  grid-template-columns: 320px 1fr;
  gap: 2.5rem;
  align-items: start;
}

/* Form card */
.form-card {
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  padding: 1.75rem;
  position: sticky;
  top: 76px;
}
.form-title {
  font-family: 'DM Serif Display', serif;
  font-size: 18px;
  font-weight: 400;
  color: var(--text);
  margin-bottom: 1.5rem;
  padding-bottom: 1rem;
  border-bottom: 1px solid var(--border);
}
.form { display: flex; flex-direction: column; gap: 1.1rem; }
.field { display: flex; flex-direction: column; gap: 6px; }
.label {
  font-size: 11px;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--text-dim);
}
.input {
  background: var(--bg);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  color: var(--text);
  font-family: 'DM Mono', monospace;
  font-size: 13px;
  padding: 9px 12px;
  outline: none;
  transition: border-color 0.15s;
  width: 100%;
}
.input:focus { border-color: var(--accent); }
.input::placeholder { color: var(--text-dim); }
.price-wrap { position: relative; }
.currency {
  position: absolute;
  left: 12px;
  top: 50%;
  transform: translateY(-50%);
  color: var(--accent);
  font-size: 13px;
  pointer-events: none;
}
.price-input { padding-left: 26px; }
.form-actions { display: flex; gap: 8px; margin-top: 4px; }
.btn {
  flex: 1;
  border: none;
  border-radius: var(--radius);
  font-family: 'DM Mono', monospace;
  font-size: 12px;
  letter-spacing: 0.05em;
  padding: 10px;
  cursor: pointer;
  transition: all 0.15s;
}
.btn-primary {
  background: var(--accent);
  color: #0f0e0d;
  font-weight: 500;
}
.btn-primary:hover { background: #e8ba57; }
.btn-ghost {
  background: transparent;
  color: var(--text-dim);
  border: 1px solid var(--border);
}
.btn-ghost:hover { border-color: var(--text-dim); color: var(--text); }

/* Table */
.table-wrap {
  border: 1px solid var(--border);
  border-radius: var(--radius);
  overflow: hidden;
}
.table { width: 100%; border-collapse: collapse; }
.table thead { background: var(--surface); }
.table th {
  padding: 12px 16px;
  text-align: left;
  font-size: 10px;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: var(--text-dim);
  border-bottom: 1px solid var(--border);
  font-weight: 500;
}
.th-price, .th-actions { text-align: right; }
.table td {
  padding: 14px 16px;
  border-bottom: 1px solid var(--border);
  vertical-align: middle;
}
.table tbody tr:last-child td { border-bottom: none; }
.table tbody tr { transition: background 0.1s; }
.table tbody tr:hover { background: var(--surface); }
.row-editing { background: rgba(212, 168, 67, 0.06) !important; }
.td-name { font-weight: 500; color: var(--text); }
.td-desc { color: var(--text-dim); font-size: 13px; max-width: 240px; }
.td-price { text-align: right; }
.price-tag {
  font-size: 13px;
  color: var(--accent);
  letter-spacing: 0.02em;
}
.no-price { color: var(--text-dim); }
.td-actions { text-align: right; white-space: nowrap; }
.action-btn {
  background: none;
  border: 1px solid var(--border);
  border-radius: var(--radius);
  color: var(--text-dim);
  cursor: pointer;
  font-family: 'DM Mono', monospace;
  font-size: 11px;
  letter-spacing: 0.05em;
  padding: 5px 10px;
  transition: all 0.15s;
  margin-left: 6px;
}
.edit-btn:hover { border-color: var(--accent); color: var(--accent); }
.del-btn:hover { border-color: var(--danger); color: var(--danger); }

.empty-state {
  padding: 4rem 2rem;
  text-align: center;
  border: 1px solid var(--border);
  border-radius: var(--radius);
}
.empty-text { color: var(--text-dim); font-size: 13px; }
.spinner { font-size: 24px; color: var(--accent); display: inline-block; animation: spin 2s linear infinite; }

@media (max-width: 768px) {
  .main { grid-template-columns: 1fr; }
  .form-card { position: static; }
}
</style>
