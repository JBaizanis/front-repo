<script setup>
/* Το component συνδέεται με το κατάλληλο endpoint στο backend για να προσπελάσει τον εκάστοτε χρήστη
   και να τον διαγράψει αφότου δωθεί η κατάλληλη έγκριση. Αν δεν δωθεί, επιστρέφει τον χρήστη στο "users" view. */
import { ref } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import { useRemoteData } from '@/composables/useRemoteData.js';
const backendEnvVar = import.meta.env.VITE_BACKEND; 

const userId = useRoute().params.id;
const router = useRouter(); // Access the router instance

const urlRef = ref(backendEnvVar+`/api/user/${userId}`);
const authRef = ref(true);
const methodRef = ref("DELETE");

const { performRequest } = useRemoteData(urlRef, authRef, methodRef);

const deleteUser = async () => {
  try {
    await performRequest();
    router.push({ name: 'users' }); // Redirect to the "users" view after successful deletion
  } catch (error) {
    console.error('Error deleting user:', error);
  }
};


</script>


<template>
  <!--Δημιουργία ενός prompt ερώτησης για την διαγραφή του χρήστη και δύο κουμπιών για την διαγραφή ή την επιστροφή αντίστοιχα. -->
  <div>
    <h2>Delete User</h2>
    <p>Are you sure you want to delete this user?</p>
    <button class="btn btn-danger" @click="deleteUser">Delete</button>
    <router-link :to="{ name: 'users' }" class="btn btn-secondary">Cancel</router-link>
  </div>
</template>
