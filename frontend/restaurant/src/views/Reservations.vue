<!-- eslint-disable vue/multi-word-component-names -->
<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';
import { useToast } from 'vue-toastification';
import flatpickr from "flatpickr";
import "flatpickr/dist/flatpickr.css";


const toast = useToast();
const errorMessage = ref('');

const name = ref('');
const selectedDate = ref(new Date().toLocaleDateString("fr-FR"));
const selectedHour = ref('12:00');
const numberOfPeople = ref(1);

const hours = [
  { value: '12:00', label: '12:00', id: '12' },
  { value: '12:30', label: '12:30', id: '12-30' },
  { value: '13:00', label: '13:00', id: '13' },
  { value: '13:30', label: '13:30', id: '13-30' },
];

onMounted(() => {
  flatpickr("#datepicker-inline", {
    dateFormat: "m/d/Y", // Format mm/dd/yyyy
    defaultDate: selectedDate.value, // Date par défaut
    onChange: (selectedDates) => {
      selectedDate.value = flatpickr.formatDate(selectedDates[0], "m/d/Y");
    },
    inline: true, // Affiche le calendrier directement sans avoir à ouvrir un pop-up
  });
});

const add = async () => {
  errorMessage.value = '';
  try {
    const storedUser = localStorage.getItem("user");
    if (!storedUser) throw new Error("User not logged in");
    const user = JSON.parse(storedUser);
    const userId = user.id;

    if (!name.value) throw new Error("Veuillez entrer votre nom.");
    if (!selectedDate.value || !selectedHour.value) throw new Error("Veuillez sélectionner une date et une heure.");
    if (numberOfPeople.value < 1) throw new Error("Veuillez entrer un nombre valide de personnes.");

    const [month, day, year] = selectedDate.value.split('/');
    const isoDate = `${year}-${month.padStart(2, '0')}-${day.padStart(2, '0')}`;
    const dateDebutString = `${isoDate}T${selectedHour.value}:00`;
    const dateDebut = new Date(dateDebutString);
    const dateFin = new Date(dateDebut.getTime() + 30 * 60 * 1000);

    const response = await axios.post('http://localhost:8000/reservations', {
      user: userId,
      name: name.value,
      numberOfPeople: numberOfPeople.value,
      dateDebut: dateDebut.toISOString(),
      dateFin: dateFin.toISOString()
    }, { withCredentials: true });

    console.log(response.data);
    toast.success("Réservation ajoutée!");

    name.value = '';
    numberOfPeople.value = 1;
    selectedHour.value = hours[0].value;
  } catch (error) {
    errorMessage.value = error.response?.data?.message || error.message;
  }
};
</script>

<template>
  <div class="min-h-screen flex items-center justify-center w-full p-6">
    <div class="bg-white shadow-lg rounded-lg p-6 max-w-md w-full">
      <h2 class="text-lg font-semibold text-gray-800 text-center mb-4">Réserver une table en 2-3 clics !</h2>

      <form @submit.prevent="add">
        <div v-if="errorMessage" class="mb-4 text-red-500 text-sm text-center">
          {{ errorMessage }}
        </div>

        <!-- Nom -->
        <div class="mb-4">
          <label for="name" class="block text-sm font-medium text-gray-700">Nom</label>
          <input type="text" id="name" v-model="name"
            class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"
            placeholder="Votre nom" required>
        </div>

        <!-- Nombre de personnes -->
        <div class="mb-4">
          <label for="people" class="block text-sm font-medium text-gray-700">Nombre de personnes</label>
          <input type="number" id="people" v-model="numberOfPeople" min="1"
            class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"
            required>
        </div>

        <!-- Sélection de la date -->
        <div class="mb-4">
          <label class="block text-sm font-medium text-gray-700">Choisissez la date</label>
          <div id="datepicker-inline" class="relative"></div> 
        </div>

        <!-- Sélection de l'heure -->
        <div class="mb-4">
          <label class="block text-sm font-medium text-gray-700">Choisissez une heure</label>
          <ul class="w-full flex flex-col mt-2 gap-2">
            <li v-for="hour in hours" :key="hour.value">
              <input type="radio" :id="hour.id" :value="hour.value" class="hidden peer" name="timetable" v-model="selectedHour" />
              <label :for="hour.id"
                class="inline-flex items-center justify-center w-full p-2 text-sm font-medium text-center bg-white border rounded-lg cursor-pointer text-blue-600 border-blue-600 hover:text-white peer-checked:border-blue-600 peer-checked:bg-blue-600 peer-checked:text-white">
                {{ hour.label }}
              </label>
            </li>
          </ul>
        </div>

        <!-- Bouton de soumission -->
        <button type="submit"
          class="w-full flex justify-center py-2 px-4 border border-transparent rounded-md shadow-sm text-sm font-medium text-white bg-amber-500 hover:bg-amber-400 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500">
          Réserver
        </button>
      </form>
    </div>
  </div>
</template>

<style scoped>

#datepicker-inline {
  margin-top: 10px;
  width: 100%;
  max-width: 400px;
}

.form-container {
  display: flex;
  justify-content: center;
}

input[type="text"] {
  cursor: pointer; 
}
</style>