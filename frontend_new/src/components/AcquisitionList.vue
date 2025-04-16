<template>
  <div class="grid grid-cols-8 h-screen bg-[#ffff]">
    <!-- Sidebar Component -->
    <Sidebar @sidebarToggle="handleSidebarToggle" />
    <div>
      <DownloadButton />
    </div>
    <!-- Acquisition List Section -->
    <div
      :class="[
        'col-start-2 col-span-6 flex flex-col h-screen p-8 bg-[#ffff] rounded-lg flex-grow mx-auto justify-start items-center mt-8',
        { shifted: isSidebarOpen },
      ]"
    >
      <h2 class="title text-3xl font-bold text-center text-[#231f20] mb-6">
        Acquisition List
      </h2>
      <div
        class="button-list flex flex-col gap-4 items-center w-full max-w-2xl"
      >
        <button
          v-for="property in properties"
          :key="property.name"
          class="property-button w-full p-4 bg-[#ffffff] text-[#2c3e50] rounded-lg font-semibold shadow-md hover:bg-[#967444] hover:text-white transition-transform transform hover:scale-105 duration-300 border border-gray-300"
          @click="handlePropertySelection(property)"
          :disabled="loading === property.id"
        >
          {{ property.name }}
        </button>
      </div>
    </div>
    <div v-if="loading" class="spinner-overlay">
      <div class="spinner-container">
        <div class="spinner-large" />
      </div>
    </div>
  </div>
</template>

<script>
import Sidebar from "./Sidebar.vue";
import DownloadButton from "./DownloadButton.vue";
import { supabase } from "../lib/supabase.js";

export default {
  name: "AcquisitionList",
  components: {
    Sidebar,
    DownloadButton,
  },
  data() {
    return {
      properties: [
        {
          id: "lim",
          name: "Light Industrial - Multi Tenant",
          route: "/acquisition-list/LightIndustrialMT",
          table: "light_industrial_multitenant",
        },
        {
          id: "iub",
          name: "Light Industrial - User Buildings",
          route: "/acquisition-list/LightIndustrialUB",
          table: "industrial_user_buildings",
        },
        {
          id: "ios",
          name: "IOS - Existing",
          route: "/acquisition-list/IOSExisting",
          table: "ios_existing",
        },
        {
          id: "",
          name: "IOS - Developable Land",
          route: "/acquisition-list/IOSDevelopment",
          table: "",
        },
      ],
      isSidebarOpen: true, // ✅ Sidebar starts
      loading: null,
    };
  },
  methods: {
    selectProperty(property) {
      this.$router.push(property.route);
    },
    handleSidebarToggle(isOpen) {
      this.isSidebarOpen = isOpen;
    },
    async createAcquisitionList(listType) {
      // Call your API to create the acquisition list
      const apiUrl =
        "https://ry57ivn9ae.execute-api.us-east-1.amazonaws.com/dev/sub-acq-list";

      const response = await fetch(apiUrl, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ list_type: listType }),
      });

      if (!response.ok) {
        const errorData = await response.json();
        throw new Error(errorData.error || "Failed to create acquisition list");
      }

      return await response.json();
    },
    async handlePropertySelection(property) {
      try {
        this.loading = true;
        // Check if table has data
        const { count, error } = await supabase
          .from(property.table)
          .select("*", { count: "exact", head: true });

        if (error) throw error;

        if (count === 0) {
          // Table exists but has no data - call API to create data
          await this.createAcquisitionList(property.id);
        }
        this.loading = false;
        // Navigate to the route after data is confirmed to exist
        this.selectProperty(property);
      } catch (error) {
        console.error("Error checking table:", error);
      } finally {
        this.loading = false;
      }
    },
  },
};
</script>

<style scoped>
/* Acquisition List Section */
.col-start-2 {
  font-family: "Sentinel – Book Aa", serif;
  flex: 1;
  transition: transform 0.3s ease-in-out; /* ✅ Smooth transition both ways */
  transform: translateX(0px); /* ✅ Sidebar starts open */
}

/* Shift back when sidebar is closed */
.shifted {
  transform: translateX(120px); /* ✅ Moves back when sidebar closes */
}

/* Button List - Expand Width */
.button-list {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  align-items: center;
  width: 100%;
  width: 500px;
  font-family: "Sentinel – Book Aa", serif;
}

/* Property Buttons */
.property-button {
  width: 100%; /* Make buttons take full width */
  max-width: 900px; /* Adjust as needed */
  padding: 1.5rem;
  border-radius: 12px;
  font-weight: 600;
  font-size: 1.2rem;
  cursor: pointer;
  text-align: center;
  background-color: #231f20;
  color: #ffff;
  border: 1px solid #d8d2c4;
  transition: background-color 0.3s ease, transform 0.2s ease;
}

.property-button:hover {
  background-color: #967444;
  color: white;
  transform: scale(1.05);
  border: none;
}

/* Spinner */
.spinner-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9999;
}

.spinner-container {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.spinner-large {
  width: 3rem;
  height: 3rem;
  border: 4px solid rgba(255, 255, 255, 0.3);
  border-radius: 50%;
  border-top-color: white;
  animation: spin 1s ease-in-out infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}
</style>
