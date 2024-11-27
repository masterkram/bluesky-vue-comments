<script setup lang="ts">
import { ref, computed, watchEffect, watch, type Ref } from "vue";
import Comment from "./Comment.vue";
import Heart from "./icons/Heart.vue";
import Reply from "./icons/Reply.vue";
import Repost from "./icons/Repost.vue";

import {
  AppBskyFeedDefs,
  AppBskyFeedPost,
  type AppBskyFeedGetPostThread,
} from "@atproto/api";

const visibleCount = ref(3);

const thread: Ref<Thread> = ref();
const error = ref();

const showMore = () => {
  visibleCount.value += 5
};

const props = defineProps<{
  uri: string
}>();

const formatUri = (uri: string): string => {
  if (!uri.startsWith("at://") && uri.includes("bsky.app/profile/")) {
    const match = uri.match(/profile\/([\w.]+)\/post\/([\w]+)/);
    if (match) {
      const [, did, postId] = match;
      return `at://${did}/app.bsky.feed.post/${postId}`;
    }
  }
  return uri;
};

const getPostThread = async (uri: string) => {
  const atUri = formatUri(uri);
  const params = new URLSearchParams({ uri: atUri });

  const res = await fetch(
    "https://public.api.bsky.app/xrpc/app.bsky.feed.getPostThread?" +
    params.toString(),
    {
      method: "GET",
      headers: {
        "Accept": "application/json",
      },
      cache: "no-store",
    },
  );

  if (!res.ok) {
    console.error(await res.text());
    throw new Error("Failed to fetch post thread");
  }

  const data = (await res.json()) as AppBskyFeedGetPostThread.OutputSchema;

  if (!AppBskyFeedDefs.isThreadViewPost(data.thread)) {
    throw new Error("Could not find thread");
  }

  return data.thread;
};

// Function to fetch the thread data
const fetchThreadData = async (uri: string) => {
  try {
    const threadResult = await getPostThread(uri);
    thread.value = threadResult;
  } catch (err) {
    console.log(err);
    error.value = "Error loading comments";
  }
};

watchEffect(() => { fetchThreadData(props.uri) }, [props.uri]);

if (props.uri) {
  const [, , did, _, rkey] = formatUri(props.uri).split("/");
  const postUrl = `https://bsky.app/profile/${did}/post/${rkey}`;
}

type Reply = {
  post: {
    uri: string;
    likeCount?: number;
    repostCount?: number;
    replyCount?: number,
  };
};

type Thread = {
  replies: Reply[];
  post: {
    likeCount?: number;
    repostCount?: number;
    replyCount?: number;
  };
};

const sortByLikes = (a: unknown, b: unknown) => {
  if (
    !AppBskyFeedDefs.isThreadViewPost(a) ||
    !AppBskyFeedDefs.isThreadViewPost(b)
  ) {
    return 0;
  }
  return (b.post.likeCount ?? 0) - (a.post.likeCount ?? 0);
};

const sortedReplies = ref();

watch([thread], () => {
  sortedReplies.value = thread.value.replies.sort(sortByLikes)
});

const moreButtonEnabled = computed(() => visibleCount.value < sortedReplies.value.length);

const posts = computed(() => sortedReplies.value.slice(0, visibleCount.value).filter(reply => AppBskyFeedDefs.isThreadViewPost(reply)));

const noComments = computed(() => (!thread.value.replies || thread.value.replies.length === 0));
</script>

<template>
  <div v-if="uri">
    <div v-if="error">{{ error }}</div>
    <div v-else-if="!thread">loading comments...</div>
    <div v-else-if="noComments">no comments yet...</div>
    <div v-else class="mt-20">
      <a :href="postUrl" target="_blank">
        <p class="flex items-center hover:underline gap-2 text-lg">
          <span class="flex items-center">
            <Heart />

            <span class="ml-1">{{ thread.post.likeCount ?? 0 }} likes</span>
          </span>
          <span class="flex items-center">
            <Repost />
            <span class="ml-1">{{ thread.post.repostCount ?? 0 }} reposts</span>
          </span>
          <span class="flex items-center">
            <Reply />
            <span class="ml-1">{{ thread.post.replyCount ?? 0 }} replies</span>
          </span>
        </p>
      </a>
      <h2 class="mt-6 text-xl font-bold">Commentss</h2>
      <p class="mt-2 text-sm">
        Reply on Bluesky
        <a :href="postUrl" class="underline" target="_blank" rel="noreferrer noopener">
          here</a>
        to join the conversation.
      </p>
      <div class="mt-2 border-b border-neutral-300 dark:border-neutral-600" />
      <div class="mt-2 space-y-8">
        <Comment v-for="reply in posts" :key="reply.post.uri" :comment="reply" />
        <button v-if="moreButtonEnabled" @click="showMore" class="mt-2 text-sm text-sky-400 underline">
          Show more comments
        </button>
      </div>
    </div>
  </div>
  <div v-else>
    Provide include url to post as prop
  </div>
</template>