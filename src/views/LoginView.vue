<script setup>
/* Το component συνδέεται με το κατάλληλο endpoint και επικοινωνεί με το backend για να κάνει authenticate τα credentials του χρήστη.
   Σε περίπτωση που τα credentials είναι έγκυρα (ταιριάζουν με αυτά στην Βάση Δεδομένων), ο χρήστης συνδέεται επιτυχώς.
   Σε περίπτωση που τα credentials δεν είναι έγκυρα, ο χρήστης δεν συνδέεται και εμφανίζεται αντίστοιχο μήνυμα (Authentication failed!). */
import { onBeforeMount, ref } from 'vue';
import { useRouter } from 'vue-router';
import { useApplicationStore } from '@/stores/application.js';
const backendEnvVar = import.meta.env.VITE_BACKEND; 

const router = useRouter();
const { setUserData, persistUserData, isAuthenticated } = useApplicationStore();

const loading = ref(false);
const credentials = ref({
  username: '',
  password: ''
});
const authenticationFailed = ref(false);

const onFormSubmit = () => {
  loading.value = true;
  authenticationFailed.value = false;

  fetch('/api/auth/signin', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(credentials.value)
  })
      .then((response) => {
        if (response.ok) {
          response.json().then((data) => {
            setUserData(data);
            persistUserData();
            router.push({ name: 'home' });

            // Reload the page after successful login
            location.reload();
          });
        } else {
          authenticationFailed.value = true;
        }
      })
      .catch((err) => {
        console.warn(err);
        authenticationFailed.value = true;
      })
      .finally(() => {
        loading.value = false;
      });
};

onBeforeMount(() => {
  if (isAuthenticated === true) {
    router.push({ name: 'home' });
  }
});
</script>

<template>
  <!--Δημιουργία πεδίων και κουμπιού για την εισαγωγή των στοιχείων του χρήστη.-->
  <div class="bg-body-tertiary">
    <div class="container">
      <div class="row py-4 px-3">
        <div class="col-4">
          <div class="mb-4">
            <h1 class="fs-3">Login</h1>
          </div>
          <div class="spinner-border" role="status" v-if="loading">
            <span class="visually-hidden">Loading...</span>
          </div>
          <form v-else>
            <div class="mb-2" v-if="authenticationFailed">
              <div class="alert alert-danger" role="alert">
                Authentication failed!
              </div>
            </div>
            <div class="mb-2">
              <label for="usernameFormControl" class="form-label mb-1"
              >Email address or Username</label
              >
              <input
                  v-model="credentials.username"
                  type="text"
                  class="form-control"
                  id="usernameFormControl"
              />
            </div>
            <div class="mb-2">
              <label for="passwordFormControl" class="form-label mb-1"
              >Password</label
              >
              <input
                  v-model="credentials.password"
                  type="password"
                  class="form-control"
                  id="passwordFormControl"
              />
            </div>
            <button @click="onFormSubmit" type="submit" class="btn btn-primary">
              Login
              <span class="fst-italic" v-if="credentials.username"
              >as {{ credentials.username }}</span
              >
            </button>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>
