<template>
  <main class="p-4 bg-gray-50 min-h-screen">
    <div class="max-w-screen-xl mx-auto bg-white p-8 rounded-lg shadow-2xl">
      <h2 class="text-3xl my-6">评论</h2>
      <CommentBox @submit="addNewComment"/>
      <DividerHorizontal />
      <div v-for="comment in comments" :key="comment.id">
        <CommentItem :user="comment.user" :avatar="comment.avatar" :time="comment.time" :content="comment.content" />
        <ReplyContainer v-if="comment.replies">
          <CommentItem v-for="reply in comment.replies" :key="reply.id" :user="reply.user" :avatar="reply.avatar" :time="reply.time" :content="reply.content" />
        </ReplyContainer>
        <ReplyBox @submit="addNewComment($event, comment.id)"/>
      </div>
    </div>
  </main>

</template>

<script setup>
import CommentBox from './components/CommentBox.vue'
import DividerHorizontal from './components/DividerHorizontal.vue';
import CommentItem from './components/CommentItem.vue';
import ReplyBox from './components/ReplyBox.vue';
import ReplyContainer from './components/ReplyContainer.vue';
import dog from './assets/狗.jpg'
import kap from './assets/卡.jpg'
import cat from './assets/猫.jpg'
import berm from './assets/小熊虫.jpg'
import { onMounted, ref } from 'vue';

const comments = ref([])

async function getAllComments() {
  const res = await fetch("/api/comments");
  comments.value = await res.json();
}

onMounted(() => {
  getAllComments();
});

const addNewComment = async (content, replyTo) => {
  const res = await fetch(`/api/comments`, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      content,
      ...(replyTo && { replyTo }),
    }),
  });

  const newComment = await res.json();
  if (!replyTo) {
    comments.value.unshift(newComment);
  } else {
    comments.value.find((c) => c.id === replyTo).replies.unshift(newComment);
  }

  // 新增完评论后，自动获取新的评论列表
  // Notion API 有延迟，在添加完 page 之后，需要过一会才能获取到新的评论列表
  // setTimeout(async () => {
  //   await getAllComments();
  // }, 1000);
};
</script>

<style></style>