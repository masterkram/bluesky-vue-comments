<script setup lang="ts">
import {
  AppBskyFeedDefs,
} from "@atproto/api";
import { computed } from 'vue';


const props = defineProps<{
  comment: AppBskyFeedDefs.ThreadViewPost
}>();

const avatarClassName = "h-4 w-4 shrink-0 rounded-full bg-gray-300";

const author = computed(() => props.comment.post.author);

const link = computed(() => `https://bsky.app/profile/${author.value.did}/post/${props.comment.post.uri.split("/").pop()}`);

const authorLink = computed(() => `https://bsky.app/profile/${author.did}`);

const hasReplies = computed(() => props.comment.replies && props.comment.replies.length > 0);


const sortByLikes = (a: unknown, b: unknown) => {
  if (
    !AppBskyFeedDefs.isThreadViewPost(a) ||
    !AppBskyFeedDefs.isThreadViewPost(b)
  ) {
    return 0;
  }
  return (b.post.likeCount ?? 0) - (a.post.likeCount ?? 0);
};

const replies = computed(() => props.comment.replies.sort(sortByLikes).filter(reply => AppBskyFeedDefs.isThreadViewPost(reply)));
</script>

<template>
  <div class="my-4 text-sm">
    <div class="flex max-w-xl flex-col gap-2">
      <a class="flex flex-row items-center gap-2 hover:underline" :href="authorLink" target="_blank"
        rel="noreferrer noopener">

        <img v-if="author && author.avatar" :src="comment.post.author.avatar" alt="avatar" :class="avatarClassName" />
        <div v-else :class="avatarClassName" />

        <p class="line-clamp-1">
          {{ author?.displayName ?? author?.handle }}
          <span class="text-gray-500">@{{ author?.handle }}</span>
        </p>
      </a>
      <a :href="link" target="_blank" rel="noreferrer noopener">
        <p>{{ comment.post.record.text }}</p>
        <Actions :post="comment.post" />
      </a>
    </div>
    <div v-if="hasReplies" class="border-l-2 border-neutral-300 dark:border-neutral-600 pl-2">
      <Comment v-for="reply in replies" :key="reply.post.uri" :comment="reply" />
    </div>
  </div>
</template>