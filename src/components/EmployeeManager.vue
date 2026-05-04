<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';

// Replace this with your actual MockAPI endpoint URL if you have one!
// Example: 'https://6xxxxxxxxxxxxxxx.mockapi.io/employees'
// Note: If the MockAPI endpoint is invalid or unreachable, the app will show errors.
// Using a placeholder URL. You must create an endpoint at mockapi.io with fields:
// name, designation, department, salary (id is auto-generated).
const API_URL = 'https://69f75beedd0c226688eda79f.mockapi.io/managementsystem'; 

const employees = ref([]);
const loading = ref(false);
const error = ref(null);

const defaultFormState = {
  id: null,
  name: '',
  designation: '',
  department: '',
  salary: ''
};

const form = ref({ ...defaultFormState });
const isEditing = ref(false);
const showForm = ref(false);

const fetchEmployees = async () => {
  loading.value = true;
  error.value = null;
  try {
    const response = await axios.get(API_URL);
    employees.value = response.data;
  } catch (err) {
    console.error('Error fetching employees:', err);
    error.value = 'Failed to load employees. Please check if your MockAPI URL is correct and accessible.';
  } finally {
    loading.value = false;
  }
};

const submitForm = async () => {
  error.value = null;
  if (!form.value.name || !form.value.designation || !form.value.department || !form.value.salary) {
    error.value = 'Please fill out all fields.';
    return;
  }

  loading.value = true;
  try {
    if (isEditing.value) {
      // Update existing employee
      const response = await axios.put(`${API_URL}/${form.value.id}`, form.value);
      const index = employees.value.findIndex(emp => emp.id === form.value.id);
      if (index !== -1) {
        employees.value[index] = response.data;
      }
    } else {
      // Create new employee
      // Removing ID so MockAPI auto-generates it
      const { id, ...newEmployeeData } = form.value;
      const response = await axios.post(API_URL, newEmployeeData);
      employees.value.push(response.data);
    }
    closeForm();
  } catch (err) {
    console.error('Error saving employee:', err);
    error.value = 'Failed to save employee. Please try again.';
  } finally {
    loading.value = false;
  }
};

const editEmployee = (employee) => {
  form.value = { ...employee };
  isEditing.value = true;
  showForm.value = true;
  error.value = null;
  window.scrollTo({ top: 0, behavior: 'smooth' });
};

const deleteEmployee = async (id) => {
  if (!confirm('Are you sure you want to delete this employee?')) return;
  
  error.value = null;
  try {
    await axios.delete(`${API_URL}/${id}`);
    employees.value = employees.value.filter(emp => emp.id !== id);
  } catch (err) {
    console.error('Error deleting employee:', err);
    error.value = 'Failed to delete employee.';
  }
};

const openAddForm = () => {
  form.value = { ...defaultFormState };
  isEditing.value = false;
  showForm.value = true;
  error.value = null;
};

const closeForm = () => {
  showForm.value = false;
  form.value = { ...defaultFormState };
  error.value = null;
};

onMounted(() => {
  fetchEmployees();
});
</script>

<template>
  <div class="employee-manager">
    <div class="d-flex justify-content-between align-items-center mb-4">
      <h2>Employee Directory</h2>
      <button v-if="!showForm" @click="openAddForm" class="btn btn-primary shadow-sm">
        <i class="bi bi-person-plus-fill"></i> Add New Employee
      </button>
    </div>

    <!-- Alert for Errors -->
    <div v-if="error" class="alert alert-danger alert-dismissible fade show shadow-sm" role="alert">
      {{ error }}
      <button type="button" class="btn-close" @click="error = null" aria-label="Close"></button>
    </div>

    <!-- Employee Form Card -->
    <div v-if="showForm" class="card shadow-sm mb-5 border-0 rounded-3">
      <div class="card-header bg-white border-bottom-0 pt-4 pb-0">
        <h4 class="mb-0 text-primary">{{ isEditing ? 'Edit Employee' : 'Add New Employee' }}</h4>
      </div>
      <div class="card-body p-4">
        <form @submit.prevent="submitForm">
          <div class="row g-3">
            <div class="col-md-6">
              <label class="form-label fw-bold">Full Name</label>
              <input type="text" class="form-control bg-light" v-model="form.name" placeholder="John Doe" required>
            </div>
            <div class="col-md-6">
              <label class="form-label fw-bold">Designation</label>
              <input type="text" class="form-control bg-light" v-model="form.designation" placeholder="Software Engineer" required>
            </div>
            <div class="col-md-6">
              <label class="form-label fw-bold">Department</label>
              <select class="form-select bg-light" v-model="form.department" required>
                <option value="" disabled>Select Department</option>
                <option value="IT">IT</option>
                <option value="HR">HR</option>
                <option value="Finance">Finance</option>
                <option value="Marketing">Marketing</option>
                <option value="Operations">Operations</option>
              </select>
            </div>
            <div class="col-md-6">
              <label class="form-label fw-bold">Salary ($)</label>
              <input type="number" class="form-control bg-light" v-model="form.salary" placeholder="75000" min="0" required>
            </div>
          </div>
          <div class="mt-4 d-flex justify-content-end gap-2">
            <button type="button" class="btn btn-outline-secondary px-4" @click="closeForm" :disabled="loading">Cancel</button>
            <button type="submit" class="btn btn-primary px-4" :disabled="loading">
              <span v-if="loading" class="spinner-border spinner-border-sm me-2" role="status" aria-hidden="true"></span>
              {{ isEditing ? 'Update Employee' : 'Save Employee' }}
            </button>
          </div>
        </form>
      </div>
    </div>

    <!-- Employee List Loading State -->
    <div v-if="loading && !showForm" class="text-center my-5">
      <div class="spinner-border text-primary" role="status">
        <span class="visually-hidden">Loading...</span>
      </div>
      <p class="mt-2 text-muted">Loading employees...</p>
    </div>

    <!-- Employee Table -->
    <div v-else class="card shadow-sm border-0 rounded-3 overflow-hidden">
      <div class="table-responsive">
        <table class="table table-hover mb-0 align-middle">
          <thead class="table-light">
            <tr>
              <th scope="col" class="ps-4">ID</th>
              <th scope="col">Name</th>
              <th scope="col">Designation</th>
              <th scope="col">Department</th>
              <th scope="col">Salary</th>
              <th scope="col" class="text-end pe-4">Actions</th>
            </tr>
          </thead>
          <tbody>
            <tr v-if="employees.length === 0">
              <td colspan="6" class="text-center py-4 text-muted">No employees found. Add one to get started!</td>
            </tr>
            <tr v-for="emp in employees" :key="emp.id">
              <td class="ps-4 fw-medium text-secondary">#{{ emp.id }}</td>
              <td class="fw-bold">{{ emp.name }}</td>
              <td><span class="badge bg-info text-dark bg-opacity-25 border border-info rounded-pill px-3">{{ emp.designation }}</span></td>
              <td>{{ emp.department }}</td>
              <td>${{ Number(emp.salary).toLocaleString() }}</td>
              <td class="text-end pe-4">
                <button @click="editEmployee(emp)" class="btn btn-sm btn-outline-primary me-2 shadow-sm" title="Edit">
                  Edit
                </button>
                <button @click="deleteEmployee(emp.id)" class="btn btn-sm btn-outline-danger shadow-sm" title="Delete">
                  Delete
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<style scoped>
.employee-manager {
  max-width: 1000px;
  margin: 0 auto;
  padding: 1rem 0;
}

.table th {
  font-weight: 600;
  text-transform: uppercase;
  font-size: 0.85rem;
  letter-spacing: 0.5px;
  color: #6c757d;
}

.form-control:focus, .form-select:focus {
  border-color: #0d6efd;
  box-shadow: 0 0 0 0.25rem rgba(13, 110, 253, 0.15);
}

.card {
  transition: all 0.3s ease;
}
</style>
