<template>
  <div class="event-detail bg-neutral-800 min-h-screen flex flex-col items-center justify-center px-4">
    <!-- Loading State -->
    <div v-if="loading" class="text-center text-gray-500 text-lg">
      Loading...
    </div>

    <!-- Event Details -->
    <div v-else class="grid md:grid-cols-3 px-6 h-full w-full md:max-h-screen md:overflow-y-hidden">
      <!-- Event Info -->
      <div class="w-full md:row-span-2 md:col-span-2 bg-neutral-800 rounded-lg p-6 sm:p-10">
        <h1 class="text-3xl md:text-5xl text-white font-bold mb-4 capitalize">
          {{ event.name }}
        </h1>
        <p class="text-gray-300 mb-6">{{ event.description }}</p>
        <div class="text-gray-200 space-y-3">
          <p><strong class="text-gray-100">Location:</strong> {{ event.location }}</p>
          <p><strong class="text-gray-100">Max Attendees:</strong> {{ event.max_attendees }}</p>
          <p><strong class="text-gray-100">Total Attendees:</strong> {{ event.attendee_count }}</p>
          <div class="grid md:grid-cols-2 gap-3">
            <p><strong class="text-gray-100">Event Start Date:</strong> {{ formatDate(event.start_date) }}</p>
            <p><strong class="text-gray-100">Close Registration:</strong> {{ formatDate(event.close_registration) }}</p>
          </div>
          <div class="flex items-center justify-end mt-10">
            <button
              v-if="event.is_joined"
              class="px-6 py-2 bg-gray-500 mt-6 text-white rounded-full cursor-not-allowed"
              disabled
            >
              Joined Already
            </button>
            <button
              v-else
              @click="joinEvent"
              class="px-6 py-2 bg-[#FA7D3B] mt-6 text-white rounded-full hover:bg-[#e77026] transition inline-block text-center"
            >
              Join Event
            </button>
          </div>
        </div>
      </div>

      <div class="w-full bg-neutral-800 md:border-l h-screen  relative pl-6">
        <h1 class="text-3xl text-white w-full font-bold my-4 capitalize">Ask Questions</h1>
        <!-- Questions List -->
      <div class="mb-4 space-y-3 max-h-[80vh] overflow-y-auto">
  <div
    v-for="question in questions"
    :key="question.question_id"
    class="flex items-center gap-6"
  >
    <p
      :class="[
        'flex-1 text-white capitalize rounded-md',
        event.creator_id === question.asked_by ? 'bg-green-700 p-2 px-4' : 'bg-neutral-900 p-2 px-4'
      ]"
    >
      {{ question.question }}
    </p>

    <!-- Vote Button and Count -->
    <div class="flex items-center gap-2">
      <!-- Vote Count -->
      <span class="text-white text-sm font-medium">{{ question.votes }}</span>

      <!-- Vote Icon/Button -->
      <button
        v-if="question.user_voted"
        class="flex items-center justify-center p-2 text-gray-400 bg-gray-700 rounded-full cursor-not-allowed"
        disabled
      >
        <svg xmlns="http://www.w3.org/2000/svg" class="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" />
        </svg>
      </button>
      <button
        v-else
        @click="vote(question.question_id)"
        class="flex items-center justify-center p-2 text-green-500 bg-green-800 rounded-full hover:bg-green-700"
      >
        <svg xmlns="http://www.w3.org/2000/svg" class="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 15l7-7 7 7" />
  </svg>
      </button>
    </div>
  </div>
</div>

        <div class="absolute bottom-5 w-full left-6 py-7 right-0 bg-neutral-800">
          <div class="flex gap-5">
            <input
              v-model="newQuestion"
              type="text"
              class="bg-neutral-900 p-3 w-full rounded-lg text-white placeholder-gray-400 focus:outline-none focus:ring-2 focus:ring-[#FA7D3B]"
              placeholder="Write Question"
            />
            <button
              @click="askQuestion"
              class="px-6 py-2 bg-[#FA7D3B] text-white rounded-full hover:bg-[#e77026]"
            >
              Ask
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>



<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";
import { useToast } from "vue-toastification";

const eventId = ref(window.location.hash.split("/").pop());
const loading = ref(false);
const event = ref({});
const questions = ref([]);
const newQuestion = ref("");
const toast = useToast();

function formatDate(date) {
  const options = { day: "numeric", month: "long", year: "numeric" };
  return new Date(date).toLocaleDateString("en-US", options);
}

// Fetch Event Details
const fetchEvent = async () => {
  loading.value = true;
  try {
    const token = localStorage.getItem("authToken");
    const headers = token ? { Authorization: `Bearer ${token}` } : {};
    const response = await axios.get(
      `${import.meta.env.VITE_BACKEND_URL}/api/events/${eventId.value}`,
      { headers }
    );
    event.value = response.data.event || {};
    questions.value = response.data.questions || [];
  } catch (err) {
    console.error("Error fetching event:", err);
    toast.error("Failed to fetch event details. Please try again.");
  } finally {
    loading.value = false;
  }
};

// Join Event
const joinEvent = async () => {
  const token = localStorage.getItem("authToken");
  if (!token) {
    toast.error("Please log in to join the event.");
    return;
  }

  try {
    await axios.post(
      `${import.meta.env.VITE_BACKEND_URL}/api/events/attend`,
      { event_id: eventId.value },
      { headers: { Authorization: `Bearer ${token}` } }
    );
    toast.success("You successfully joined the event.");
    fetchEvent(); // Refresh event details
  } catch (err) {
    console.error("Error joining event:", err);
    toast.error("Failed to join the event. Please try again.");
  }
};

// Ask Question
const askQuestion = async () => {
  if (!newQuestion.value.trim()) {
    toast.error("Question cannot be empty!");
    return;
  }

  const token = localStorage.getItem("authToken");
  if (!token) {
    toast.error("Please log in to ask a question.");
    return;
  }

  try {
    const response = await axios.post(
      `${import.meta.env.VITE_BACKEND_URL}/api/events/question`,
      { question: newQuestion.value, event_id: eventId.value },
      { headers: { Authorization: `Bearer ${token}` } }
    );
    questions.value.push(response.data.data);
    newQuestion.value = "";
    toast.success("Question submitted successfully.");
  } catch (err) {
    console.error("Error asking question:", err);
    toast.error("Failed to submit the question. Please try again.");
  }
};

// Vote on a Question
const vote = async (question_id) => {
  const token = localStorage.getItem("authToken");
  if (!token) {
    toast.error("Please log in to vote.");
    return;
  }

  try {
    await axios.post(
      `${import.meta.env.VITE_BACKEND_URL}/api/events/vote`,
      { question_id },
      { headers: { Authorization: `Bearer ${token}` } }
    );
    toast.success("Vote recorded successfully.");
    fetchEvent(); // Refresh questions with updated votes
  } catch (err) {
    console.error("Error voting on question:", err);
    toast.error("Failed to vote. Please try again.");
  }
};

onMounted(() => {
  fetchEvent();
});
</script>




