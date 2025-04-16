<template>
  <button class="download-btn" @click="downloadReport">
    Download Delta Report
  </button>
</template>

<script>
//import { fetchDeltaReport } from "@/services/apiService";

export default {
  methods: {
    downloadReport() {
      fetchDeltaReport();
    },
  },
};
</script>

<style scoped>
.download-btn {
  position: fixed;
  bottom: 20px;
  right: 10px;
  background-color: #231f20;
  font-family: "Sentinel – Book Aa", serif;
  color: white;
  border: 1px solid #d8d2c4;
  padding: 10px 15px;
  cursor: pointer;
  box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
  border-style: none;
}
.download-btn:hover {
  background-color: #000000; /* black on hover */
  color: #ffffff !important; /* ensure text is white */
  transform: scale(1.05); /* optional: a little zoom effect */
  transition-duration: 300ms;
}
</style>
