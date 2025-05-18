<template>
  <div class="legacy-layout">
    <header>
      <legacy-header />
    </header>
    <main class="container">
      <div class="row">
        <div class="col-3 scrollable">
          <chat-list />
        </div>
        <div class="col scrollable">
          <slot />
        </div>
      </div>
    </main>
    <footer>
      <slot name="footer" />
    </footer>
  </div>
</template>
<script>
import LegacyHeader from "@/components/LegacyHeader.vue";
import ChatList from "@/pages/ChatList.vue";

export default {
  name: "LegacyWithChatList",
  components: {
    LegacyHeader,
    ChatList,
  },
};
</script>
<style scoped>
.legacy-layout {
  height: 100vh;
}
.scrollable {
  overflow-y: auto;
  height: 90vh;
}
</style>
