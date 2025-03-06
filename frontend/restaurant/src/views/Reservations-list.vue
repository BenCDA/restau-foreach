<script setup>
import { ref, onMounted } from "vue";
import VueCal from "vue-cal";
import "vue-cal/dist/vuecal.css";
import axios from "axios";

const reservations = ref([]);
const selectedEvent = ref(null);

const fetchReservations = async () => {
    try {
        const storedUser = localStorage.getItem("user");
        if (!storedUser) throw new Error("User not logged in");

        const user = JSON.parse(storedUser);
        const userId = user.id;

        const response = await axios.get(
            `http://localhost:8000/reservations/user/${userId}`,
            { withCredentials: true }
        );

        reservations.value = response.data.map((appt) => ({
            title: "Le Gourmet Moderne",
            start: new Date(appt.dateDebut),
            end: new Date(appt.dateFin)
        }));
    } catch (error) {
        console.error("Erreur lors du chargement des rendez-vous", error);
    }
};

const showEventDetails = (event) => {
    selectedEvent.value = event;
};

const formatDate = (date) => {
    return new Intl.DateTimeFormat("fr-FR", {
        year: "numeric",
        month: "long",
        day: "numeric",
        hour: "2-digit",
        minute: "2-digit",
    }).format(date);
};

onMounted(fetchReservations);
</script>

<template>
<div class="min-h-screen flex items-center justify-center bg-gray-900 p-6">
    <div class="container max-w-4xl bg-gray-800 p-6 rounded-xl shadow-lg text-white h-[80vh] overflow-hidden flex flex-col">
        <h2 class="text-2xl font-semibold text-center mb-4">Calendrier des réservations</h2>
        <div class="flex-1 overflow-y-auto">
            <vue-cal
                class="vuecal--blue-theme shadow-lg rounded-lg overflow-hidden"
                locale="fr"
                :events="reservations"
                :disable-views="['years', 'year']"
                @event-click="showEventDetails"
                :time="true"
                :time-from="12 * 60"
                :time-to="23 * 60"
                :time-step="30"
				

            />
        </div>

        <!-- Modal -->
        <div v-if="selectedEvent" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center">
            <div class="bg-white text-gray-900 p-6 rounded-lg shadow-xl w-96">
                <div class="flex justify-between items-center border-b pb-2">
                    <h5 class="text-lg font-bold">Détails du rendez-vous</h5>
                    <button class="text-gray-500 hover:text-gray-700" @click="selectedEvent = null">&times;</button>
                </div>
                <div class="mt-4">
                    <p><strong>Titre:</strong> {{ selectedEvent.title }}</p>
                    <p><strong>Date:</strong> {{ formatDate(selectedEvent.start) }}</p>
                </div>
                <div class="mt-4 flex justify-end">
                    <button class="bg-blue-600 text-white px-4 py-2 rounded-md hover:bg-blue-700" @click="selectedEvent = null">Fermer</button>
                </div>
            </div>
        </div>
    </div>
</div>
</template>

<style>
.vuecal {
    border-radius: 10px;
    overflow: hidden;
    font-family: 'Poppins', sans-serif;
    height: 100%;
}

.vuecal__title-bar {
    background: #1e40af !important;
    color: white !important;
}

.vuecal__event {
    background: #3b82f6 !important;
    color: white !important;
    border-radius: 8px !important;
    padding: 4px 8px !important;
}

/* Cache le texte "aucun evenement" pour le remplacer par "aucune reservation" seul moyen de changer le texte */
.vuecal__no-event {
    color: transparent; 
    position: relative;
}

.vuecal__no-event::after {
    content: "Aucune réservation";
    color: white;
	font-size: 12px;
	font-family: Arial, sans-serif;
    position: absolute;
    left: 0;
    right: 0;
    text-align: center;
}
</style>
