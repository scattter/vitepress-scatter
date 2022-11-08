<template>
  <div class="card">
    <slot name="custom"></slot>
    <div v-if="!$slots.custom" class="card-content">
      <h1 class="topic">{{ topic }}</h1>
      <h2 class="title">{{ title }}</h2>
      <p class="detail">{{ detail }}</p>
    </div>
  </div>
</template>
<script setup>
const props = defineProps({
  info: Object
})
const { topic, title, detail } = props.info
</script>
<style lang="scss" scoped>
.card {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  width: 100%;
  height: 100%;
  border: 2px solid var(--vp-c-bg-mute);
  border-radius: 10px;
  margin: 10px;
  padding: 24px;
  font-size: 20px;
  .topic {
    font-size: 30px;
    margin-bottom: 10px;
  }
  .title {
    line-height: 24px;
    font-size: 16px;
    font-weight: 600;
  }
  .detail {
    flex-grow: 1;
    padding-top: 8px;
    line-height: 24px;
    font-size: 14px;
    font-weight: 500;
    color: var(--vp-c-text-2);
  }
}
</style>