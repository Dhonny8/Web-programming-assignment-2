<script setup>
import EmployeeManager from './components/EmployeeManager.vue'
</script>

<template>
  <div>
    <nav class="navbar navbar-dark bg-dark mb-4">
      <div class="container-fluid">
        <span class="navbar-brand mb-0 h1">Enterprise Employee System</span>
      </div>
    </nav>
    <main class="container">
      <EmployeeManager />
    </main>
  </div>
</template>

<style>
body {
  background-color: #f8f9fa;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}
</style>
