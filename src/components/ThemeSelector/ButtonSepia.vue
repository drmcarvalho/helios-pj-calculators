<template>
  <a href="#" @click.prevent="changeThemeMode">
    <svg
      xmlns="http://www.w3.org/2000/svg"
      width="24"
      height="24"
      viewBox="0 0 24 24"
      fill="none"
      stroke="currentColor"
      stroke-width="2"
      stroke-linecap="round"
      stroke-linejoin="round"
      class="feather feather-coffee preferred selected"
      data-v-2b778c3e=""
    >
      <path d="M18 8h1a4 4 0 0 1 0 8h-1" data-v-2b778c3e=""></path>
      <path
        d="M2 8h16v9a4 4 0 0 1-4 4H6a4 4 0 0 1-4-4V8z"
        data-v-2b778c3e=""
      ></path>
      <line x1="6" y1="1" x2="6" y2="4" data-v-2b778c3e=""></line>
      <line x1="10" y1="1" x2="10" y2="4" data-v-2b778c3e=""></line>
      <line x1="14" y1="1" x2="14" y2="4" data-v-2b778c3e=""></line>
    </svg>
  </a>
</template>

<script>
export default {
  methods: {
    changeThemeMode() {
      this.$emit('changeThemeMode', 'sepia');
    },
  },
};
</script>

<style></style>
