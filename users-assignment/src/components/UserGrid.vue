<template>
  <div>
    <div class="sorting-container">
      <button @click="sortUsers('total')">
        Twubric Score {{ sortArrow("total") }}
      </button>
      <button @click="sortUsers('friends')">
        Friends {{ sortArrow("friends") }}
      </button>
      <button @click="sortUsers('influence')">
        Influence {{ sortArrow("influence") }}
      </button>
      <button @click="sortUsers('chirpiness')">
        Chirpiness {{ sortArrow("chirpiness") }}
      </button>
      <button @click="currentSort = { field: null, ascending: true }">
        Reset Sorting
      </button>
    </div>
    <div class="date-picker-container">
      <DatePicker v-model:value="startDate" placeholder="Start Date" />
      <DatePicker v-model:value="endDate" placeholder="End Date" />
      <button
        @click="(startDate = null), (endDate = null)"
        class="reset-button"
      >
        Reset
      </button>
    </div>
    <carousel :itemsToShow="3" :navigation="true">
      <template #addons>
        <navigation />
      </template>
      <slide v-for="user in sortedAndFilteredUsers" :key="user.uid">
        <div class="user-card">
          <div class="user-info">
            <div class="name-total">
              <h3>{{ user.fullname }}</h3>
              <p>Total: {{ user.twubric.total }}</p>
            </div>
            <div class="extra-info">
              <div class="extra-info-item">
                <p>Friends:</p>
                <p>{{ user.twubric.friends }}</p>
              </div>
              <div class="extra-info-item">
                <p>Influence:</p>
                <p>{{ user.twubric.influence }}</p>
              </div>
              <div class="extra-info-item">
                <p>Chirpiness:</p>
                <p>{{ user.twubric.chirpiness }}</p>
              </div>
            </div>
            <div class="user-actions">
              <p>Start Date: {{ formatDate(user.join_date) }}</p>
              <button @click="removeUser(user.uid)">Remove</button>
            </div>
          </div>
        </div>
      </slide>
    </carousel>
  </div>
</template>

<script setup>
import { ref, computed } from "vue";
import data from "../data/user.json";
import DatePicker from "vue-datepicker-next";
import { Carousel, Slide } from "vue3-carousel";
import "vue3-carousel/dist/carousel.css";
import "vue-datepicker-next/index.css";

const users = ref(data);
const startDate = ref(null);
const endDate = ref(null);
const currentSort = ref({ field: null, ascending: true });

// Show the sort arrow beside the selection
const sortArrow = (field) => {
  if (currentSort.value.field === field) {
    return currentSort.value.ascending ? "↑" : "↓";
  }
  return "";
};

// Search the users based on the selection
const sortedAndFilteredUsers = computed(() => {
  let filtered = users.value.filter((user) => {
    // Return if there is no selection of start and end Dates
    if (!startDate.value || !endDate.value) return true;
    const start = new Date(startDate.value).getTime();
    const end = new Date(endDate.value).getTime();
    const joinDate = new Date(user.join_date * 1000).getTime();
    // Return the users who come under the below conditions
    return joinDate >= start && joinDate <= end;
  });

  // Sort the users based on the selected category
  if (currentSort.value.field) {
    filtered.sort((a, b) => {
      const multiplier = currentSort.value.ascending ? 1 : -1;
      return (
        (a.twubric[currentSort.value.field] -
          b.twubric[currentSort.value.field]) *
        multiplier
      );
    });
  }

  return filtered;
});

// Sort the users based on the selected category
const sortUsers = (criteria) => {
  if (currentSort.value.field === criteria) {
    // This will get opposite sort
    currentSort.value.ascending = !currentSort.value.ascending;
  } else {
    currentSort.value = { field: criteria, ascending: true };
  }
};

// Format the date and show in card
const formatDate = (timestamp) => {
  const date = new Date(timestamp * 1000);
  return date.toDateString();
};

// Remove user when hit remove
const removeUser = (uid) => {
  users.value = users.value.filter((user) => user.uid !== uid);
};
</script>

<style>
.sorting-container,
.date-picker-container {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin-bottom: 20px;
}

button {
  padding: 10px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

button:hover {
  background-color: #0056b3;
}
.user-card {
  border: 1px solid #ccc;
  padding: 10px;
  border-radius: 5px;
  display: flex;
  flex-direction: column;
  background-color: #f9f9f9; /* Light background for the card */
}

.user-info {
  border-bottom: 1px solid #e0e0e0; /* Light grey border at the bottom of user-info */
  margin-bottom: 10px; /* Space below the user-info section */
  padding-bottom: 10px; /* Space inside the bottom of user-info */
}

.name-total {
  display: flex;
  justify-content: space-between;
  border-bottom: 1px dashed #ddd; /* Dashed border under the name-total section */
  padding-bottom: 5px; /* Padding below the name and total */
}

.extra-info {
  display: flex;
  justify-content: space-around;
  margin-top: 10px; /* Space above the extra-info section */
}

.extra-info-item {
  flex: 1;
  text-align: center;
  padding: 10px; /* Padding around each item for spacing */
  border-right: 1px solid #ddd; /* Border between each extra-info item */
}

.extra-info-item:last-child {
  border-right: none; /* Removes the border from the last item */
}

.user-actions {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

button {
  padding: 5px 10px;
  background-color: #ff5555;
  color: white;
  border: none;
  border-radius: 3px;
  cursor: pointer;
}

button:hover {
  background-color: #cc4444; /* Darker red on hover */
}

.date-picker-container {
  display: flex;
  align-items: center;
  justify-content: space-around; /* Adjust space distribution as needed */
  padding: 20px; /* Adds some padding around the elements */
  background-color: #f4f4f4; /* Light grey background */
  border-radius: 8px; /* Rounded corners for the container */
}

.date-picker-container > * {
  margin: 0 10px; /* Space between each element */
}

.reset-button {
  padding: 10px 20px;
  background-color: #007bff; /* Bootstrap primary color */
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.reset-button:hover {
  background-color: #0056b3; /* Darker shade on hover */
}
</style>
