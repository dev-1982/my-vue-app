<template>
  <main>
    <h1>Tiny Forms</h1>
    <OrderForm />
    <SubmissionsList />
  </main>
</template>

<script setup>
import OrderForm from './components/OrderForm.vue'
import SubmissionsList from './components/SubmissionsList.vue'
</script>

<style>
main {
  display: grid;
  gap: 2rem;
  padding: 2rem;
}
</style>

