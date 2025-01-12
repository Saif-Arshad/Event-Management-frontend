<script setup>

import { ref, onMounted } from "vue";
import axios from "axios";
const loading = ref(false);
import { useToast } from "vue-toastification";

const events = ref([]); 
const toast = useToast();

function handleDetailClick(eventId) {
    const authToken = localStorage.getItem("authToken");

    if (!authToken) {
      toast.error("Please log in to see event details.");
      return;
    }

    window.location.href = `#/events/${eventId}`;

  }

const fetchEvents = async () => {
  loading.value = true;

  try {
    const response = await axios.get(
      `${import.meta.env.VITE_BACKEND_URL}/api/events/all-events`,);
    console.log( response.data)
    events.value = response.data.events || [];
  } catch (err) {
    toast.error(err.response?.data?.message || "Failed to fetch events.");
  } finally {
    loading.value = false;
  }
};
function formatDate(date) {
   const options = { day: 'numeric', month: 'long', year: 'numeric' };
   return new Date(date).toLocaleDateString('en-US', options); // Adjust locale as needed
 }
onMounted(() => {
  fetchEvents();
});
</script>

<template>
   <div 
   class="relative min-h-screen w-full main--landing">
    <div class="absolute inset-0 h-full w-full "></div>
    <div class="grid min-h-screen px-8">
     <div class="container relative z-10 my-auto mx-auto grid place-items-center text-center">
 
  <h1 class="text-white text-5xl lg:max-w-3xl font-extrabold leading-tight">
    Eventitude: 
    <br>
    Empowering Connections Through Events
  </h1>
  <p class="text-white mt-1 mb-12 w-full md:max-w-full lg:max-w-2xl text-lg">
    Discover, create, and join events that bring people together. With 
    Eventitude, users can host events, join communities, and spark 
    conversations by asking and answering questions. Your next adventure starts here!
  </p>

</div>
    </div>
  </div>

  <div class="bg-black p-5 w-full ">
 <h1 class="text-white text-5xl lg:max-w-3xl text-center  mx-auto font-extrabold leading-tight">
   Latest Events
  </h1>
     <div v-if="loading" class="text-center text-gray-500 mt-6">
      Loading...
    </div>
  <div v-else class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-2 2xl:grid-cols-2 gap-4 mt-10">
  <div
    v-for="event in events"
    :key="event.event_id"
    class="p-4 rounded-xl shadow-md  bg-neutral-900 border border-neutral-700"
  >
    <h3 class="text-2xl text-white capitalize lg:text-3xl xl:text-4xl font-bold mb-2">
      {{ event.name }}
    </h3>
    <p class="text-sm text-gray-400 mb-6">{{ event.description }}</p>
    <p class="text-sm text-gray-300">
      <strong>Location:</strong> {{ event.location }}
    </p>
    <p class="text-sm text-gray-300 mt-1">
      <strong>Max Attendees:</strong> {{ event.max_attendees }}
    </p>
    <div class="flex flex-col md:flex-row md:items-center justify-between mt-1 space-y-2 md:space-y-0">
  <p class="text-sm text-gray-300">
    <strong>Start Date:</strong> {{ formatDate(event.start_date) }}
  </p>
  <p class="text-sm text-gray-300">
    <strong>Close Registration:</strong> {{ formatDate(event.close_registration) }}
  </p>


</div>
<div class="flex items-center justify-end mt-9">
<a
  href="#"
  @click.prevent="handleDetailClick(event.event_id)"
  class="px-6 py-2 bg-[#FA7D3B] text-white rounded-full hover:bg-[#e77026] transition inline-block text-center"
>
  Detail
</a>

</div>


   
  </div>
</div>
  </div>

</template>

<style>
.main--landing{
  background-image: url(../../assets/event.jpeg);
}
</style>
