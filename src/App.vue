<template>
  <div class="main" id="mainPage">
    <head-bar @newChatEvent="handleChatEvent" />
    <main-page :goNewChat="goNewChat" />
  </div>
</template>

<script>
import mainPage from "@/views/mainPage/index.vue";
import headBar from "@/views/mainPage/headBar.vue";
export default {
  components: {
    mainPage,
    headBar
  },
  data() {
    return {
      goNewChat: false,
    }
  },
  methods: {
    handleChatEvent() {
      this.goNewChat = !this.goNewChat
    }
  }
}
</script>

<style>
.main {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
</style>