<!-- src/components/SubmissionsList.vue -->
<template>
  <section class="card">
    <h2>Submissions</h2>
    <div class="toolbar">
      <input v-model.trim="query" type="text" placeholder="Search (any field)" @input="onSearch" />
      <select v-model="formIdFilter" @change="onSearch">
        <option value="">All forms</option>
        <option value="order-form-v1">Order form</option>
        <!-- other formIds can be listed dynamically -->
      </select>
    </div>

    <table class="list">
      <thead>
        <tr>
          <th>Id</th>
          <th>Form Id</th>
          <th>Submitted</th>
          <th>Data (flattened preview)</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="s in paged" :key="s.id">
          <td>{{ s.id }}</td>
          <td>{{ s.formId }}</td>
          <td>{{ formatDate(s.submittedAt) }}</td>
          <td class="data-cell">{{ summarize(s.data) }}</td>
        </tr>
        <tr v-if="paged.length === 0">
          <td colspan="4">No results</td>
        </tr>
      </tbody>
    </table>

    <div class="pager">
      <button :disabled="page===1" @click="page--">Prev</button>
      <span>Page {{ page }}</span>
      <button :disabled="page*pageSize>=filtered.length" @click="page++">Next</button>
    </div>
  </section>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { API_BASE } from '../api'
const items = ref([])
const query = ref('')
const formIdFilter = ref('')
const page = ref(1)
const pageSize = 10

onMounted(load)

async function load() {
  
const res = await fetch(`${API_BASE}/submissions?limit=1000`)
  items.value = await res.json()
}

function onSearch() { page.value = 1 }

const filtered = computed(() => {
  const q = query.value.toLowerCase()
  return items.value.filter(s => {
    if (formIdFilter.value && s.formId !== formIdFilter.value) return false
    if (!q) return true
    // simple deep-search on values
    const values = Object.values(flatten(s.data)).map(v => String(v).toLowerCase())
    return values.some(v => v.includes(q))
  })
})

const paged = computed(() => filtered.value.slice((page.value - 1) * pageSize, page.value * pageSize))

function flatten(obj, prefix = '', out = {}) {
  for (const [k, v] of Object.entries(obj || {})) {
    const key = prefix ? `${prefix}.${k}` : k
    if (v && typeof v === 'object' && !Array.isArray(v)) flatten(v, key, out)
    else out[key] = v
  }
  return out
}

function summarize(data) {
  const f = flatten(data)
  const pairs = Object.entries(f).slice(0, 6).map(([k, v]) => `${k}: ${v}`)
  return pairs.join(' | ')
}

function formatDate(d) {
  try { return new Date(d).toLocaleString() } catch { return d }
}
</script>

<style scoped>
.card { max-width: 960px; margin: 2rem auto; padding: 1.5rem; border: 1px solid #e5e7eb; border-radius: 12px; background: #fff; }
.toolbar { display: flex; gap: .75rem; margin-bottom: 1rem; }
.list { width: 100%; border-collapse: collapse; }
.list th, .list td { border-bottom: 1px solid #e5e7eb; padding: .5rem .75rem; text-align: left; }
.data-cell { white-space: nowrap; overflow: hidden; text-overflow: ellipsis; max-width: 480px; }
.pager { display: flex; gap: 1rem; align-items: center; justify-content: flex-end; margin-top: 1rem; }
</style>
