<script setup>
import { ref, onMounted } from 'vue'
const API = 'http://localhost:4000/api/items'
const items = ref([])
const form = ref({ name: '', description: '', price: '' })
const editId = ref(null)

async function load() {
  items.value = await fetch(API).then(r => r.json())
}

async function save() {
  if (editId.value) {
    await fetch(`${API}/${editId.value}`, {
    method: 'PUT', headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(form.value) })
    editId.value = null
  } else {
    await fetch(API, { method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(form.value) })
  }
  form.value = { name: '', description: '', price: '' }; load()
}

function startEdit(item) {
  editId.value = item.id
  form.value = { name: item.name, description: item.description, price: item.price }
}

async function remove(id) {
  await fetch(`${API}/${id}`, { method: 'DELETE' }); load()
}

onMounted(load)
</script>

<template>
  <main>
    <header class="app-header">
      <h1>Items</h1>
      <span class="item-count">{{ items.length }} record{{ items.length !== 1 ? 's' : '' }}</span>
    </header>

    <section class="form-section">
      <h2>{{ editId ? 'Edit Item' : 'New Item' }}</h2>
      <form @submit.prevent="save">
        <div class="form-row">
          <input v-model="form.name" placeholder="Name" required />
          <input v-model="form.description" placeholder="Description" />
          <input v-model.number="form.price" placeholder="Price" type="number" min="0" step="0.01" />
          <button type="submit" :class="editId ? 'btn-update' : 'btn-add'">
            {{ editId ? 'Update' : 'Add' }}
          </button>
          <button v-if="editId" type="button" class="btn-cancel" @click="editId = null; form = { name: '', description: '', price: '' }">
            Cancel
          </button>
        </div>
      </form>
    </section>

    <section class="table-section">
      <table v-if="items.length">
        <thead>
          <tr>
            <th>#</th>
            <th>Name</th>
            <th>Description</th>
            <th>Price</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(item, index) in items" :key="item.id" :class="{ editing: editId === item.id }">
            <td class="col-index">{{ index + 1 }}</td>
            <td class="col-name">{{ item.name }}</td>
            <td class="col-desc">{{ item.description }}</td>
            <td class="col-price">₱{{ Number(item.price).toFixed(2) }}</td>
            <td class="col-actions">
              <button class="btn-edit" @click="startEdit(item)">Edit</button>
              <button class="btn-delete" @click="remove(item.id)">Delete</button>
            </td>
          </tr>
        </tbody>
      </table>
      <p v-else class="empty-state">No items yet. Add one above.</p>
    </section>
  </main>
</template>

<style scoped>
* {
  box-sizing: border-box;
}

main {
  max-width: 860px;
  margin: 12px auto;
  padding: 0 20px;
  font-family: 'Segoe UI', system-ui, sans-serif;
  color: #1e293b;
}

.app-header {
  display: flex;
  align-items: baseline;
  gap: 12px;
  border-bottom: 2px solid #e2e8f0;
  padding-bottom: 10px;
  margin-bottom: 24px;
}

h1 {
  font-size: 1.5rem;
  font-weight: 700;
  margin: 0;
}

.item-count {
  font-size: 0.8rem;
  color: #64748b;
  font-weight: 400;
}

.form-section {
  background: #f8fafc;
  border: 1px solid #e2e8f0;
  border-radius: 6px;
  padding: 16px 20px;
  margin-bottom: 24px;
}

.form-section h2 {
  font-size: 0.85rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: #64748b;
  margin: 0 0 12px;
}

.form-row {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
  align-items: center;
}

input {
  flex: 1;
  min-width: 120px;
  padding: 7px 10px;
  border: 1px solid #e2e8f0;
  border-radius: 4px;
  font-size: 0.9rem;
  background: #fff;
  color: #1e293b;
  outline: none;
  transition: border-color 0.15s;
}

input:focus {
  border-color: #2563eb;
}

button {
  padding: 7px 14px;
  border: none;
  border-radius: 4px;
  font-size: 0.875rem;
  cursor: pointer;
  font-weight: 500;
  transition: opacity 0.15s;
  white-space: nowrap;
}

button:hover { opacity: 0.85; }

.btn-add    { background: #2563eb; color: #fff; }
.btn-update { background: #0d9488; color: #fff; }
.btn-cancel { background: #e2e8f0; color: #1e293b; }

.table-section {
  border: 1px solid #e2e8f0;
  border-radius: 6px;
  overflow: hidden;
}

table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.9rem;
}

thead {
  background: #f8fafc;
}

th {
  text-align: left;
  padding: 10px 14px;
  font-size: 0.78rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: #64748b;
  border-bottom: 1px solid #e2e8f0;
}

td {
  padding: 10px 14px;
  border-bottom: 1px solid #e2e8f0;
  vertical-align: middle;
}

tr:last-child td { border-bottom: none; }

tr.editing { background: #eff6ff; }

.col-index { color: #64748b; width: 40px; }
.col-price { font-weight: 600; width: 100px; }
.col-actions { width: 130px; }

.col-actions {
  display: flex;
  gap: 6px;
}

.btn-edit   { background: #f1f5f9; color: #1e293b; font-size: 0.8rem; padding: 5px 10px; }
.btn-delete { background: #fee2e2; color: #dc2626; font-size: 0.8rem; padding: 5px 10px; }

.empty-state {
  text-align: center;
  padding: 32px;
  color: #64748b;
  font-size: 0.9rem;
}
</style>