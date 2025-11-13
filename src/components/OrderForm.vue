<!-- src/components/OrderForm.vue -->
<template>
  <section class="card">
    <h2>Place an order</h2>
    <form @submit.prevent="onSubmit" novalidate>
      <!-- Text -->
      <div class="field">
        <label for="name">Name</label>
        <input id="name" v-model.trim="form.name" type="text" />
        <p v-if="errors.name" class="error">{{ errors.name }}</p>
      </div>

      <!-- Drop-down -->
      <div class="field">
        <label for="product">Product</label>
        <select id="product" v-model="form.product">
          <option value="">Select a product</option>
          <option value="Widget A">Widget A</option>
          <option value="Widget B">Widget B</option>
          <option value="Widget Pro">Widget Pro</option>
        </select>
        <p v-if="errors.product" class="error">{{ errors.product }}</p>
      </div>

      <!-- Date -->
      <div class="field">
        <label for="deliveryDate">Delivery date</label>
        <input id="deliveryDate" v-model="form.deliveryDate" type="date" />
        <p v-if="errors.deliveryDate" class="error">{{ errors.deliveryDate }}</p>
      </div>

      <!-- Radio -->
      <div class="field">
        <span>Payment method</span>
        <div class="radio-group">
          <label><input type="radio" value="card" v-model="form.paymentMethod" /> Card</label>
          <label><input type="radio" value="invoice" v-model="form.paymentMethod" /> Invoice</label>
        </div>
        <p v-if="errors.paymentMethod" class="error">{{ errors.paymentMethod }}</p>
      </div>

      <!-- Checkbox -->
      <div class="field">
        <label>
          <input type="checkbox" v-model="form.acceptTerms" />
          I accept the terms
        </label>
        <p v-if="errors.acceptTerms" class="error">{{ errors.acceptTerms }}</p>
      </div>

      <button class="primary" type="submit" :disabled="submitting">Submit</button>
    </form>
  </section>
</template>

<script setup>
import { reactive, ref } from 'vue'
import { API_BASE } from '../api'
const submitting = ref(false)
const form = reactive({
  name: '',
  product: '',
  deliveryDate: '',
  paymentMethod: '',
  acceptTerms: false,
})
const errors = reactive({})

function validate() {
  Object.keys(errors).forEach(k => delete errors[k])
  if (!form.name || form.name.length < 2) errors.name = 'Name is required (min 2 chars).'
  if (!form.product) errors.product = 'Please select a product.'
  if (!form.deliveryDate) errors.deliveryDate = 'Please choose a delivery date.'
  else if (new Date(form.deliveryDate) < new Date().setHours(0,0,0,0))
    errors.deliveryDate = 'Delivery date cannot be in the past.'
  if (!form.paymentMethod) errors.paymentMethod = 'Please select payment method.'
  if (!form.acceptTerms) errors.acceptTerms = 'You must accept the terms.'
  return Object.keys(errors).length === 0
}

async function onSubmit() {
  if (!validate()) return
  submitting.value = true
  try {
    

const res = await fetch(`${API_BASE}/submissions`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      // Backend-agnostic payload: formId + data map
      body: JSON.stringify({
        formId: 'order-form-v1',
        data: { ...form },
        submittedAt: new Date().toISOString()
      })
    })
    if (!res.ok) throw new Error('Failed to submit')
    // Clear fields on success
    Object.assign(form, { name: '', product: '', deliveryDate: '', paymentMethod: '', acceptTerms: false })
    alert('Submitted!')
  } catch (e) {
    alert(e.message)
  } finally {
    submitting.value = false
  }
}
</script>

<style scoped>
.card { max-width: 480px; margin: 2rem auto; padding: 1.5rem; border: 1px solid #e5e7eb; border-radius: 12px; box-shadow: 0 2px 12px rgba(0,0,0,0.06); background: #fff; }
.field { display: flex; flex-direction: column; gap: .5rem; margin-bottom: 1rem; }
label { font-weight: 600; }
input[type="text"], input[type="date"], select { padding: .5rem .75rem; border: 1px solid #cbd5e1; border-radius: 8px; }
.radio-group { display: flex; gap: 1rem; margin-top: .25rem; }
.error { color: #b91c1c; font-size: .875rem; }
.primary { background: #2563eb; color: #fff; border: none; padding: .6rem 1rem; border-radius: 8px; cursor: pointer; }
.primary:disabled { opacity: .6; cursor: not-allowed; }
</style>
