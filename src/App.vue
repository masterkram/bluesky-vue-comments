<script setup lang="ts">
import HelloWorld from './components/HelloWorld.vue'
import TheWelcome from './components/TheWelcome.vue'
import CommentSection from './components/Comments/CommentSection.vue';
</script>

<template>
  <header>
    <div class="text-center font-bold text-3xl mt-8">
      Vue bluesky comments
    </div>
  </header>

  <main class="mx-auto max-w-2xl">
    <CommentSection :uri="`https://bsky.app/profile/mark.speaker.blue/post/3lbunendh4s2d`" />
  </main>
</template>