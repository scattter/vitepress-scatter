<template>
  <div class="home">
    <div class="banner">WELCOME</div>
    <div class="cards">
      <card class="home-card" :class="{'custom-card': item.type && item.type === 'custom'}" v-for="item in cards" :key="item.title" :info="item">
        <template v-if="item.type && item.type === 'custom'" v-slot:custom>
          <p class="title">Learning Plan</p>
          <p>1. solidity</p>
          <p>2. 50 days 50 projects</p>
          <p>3. update before docs to website</p>
        </template>
      </card>
    </div>
  </div>
</template>
<script setup>
import card from './card.vue'
const cards = [
  {
    type: 'custom card',
  },
  {
    topic: '🐕',
    title: 'zkk',
    detail: 'a boy'
  },
  {
    topic: '🦊',
    title: 'sww',
    detail: 'a cool girl'
  },
]
</script>
<style lang="scss" scoped>
.home {
  padding: 5px 10px;
  .banner {
    width: 100%;
    height: 100px;
    background-color: bisque;
    border-radius: 10px;
    text-align: center;
    line-height: 100px;
    color: brown;
    font-size: 30px;
    font-weight: bold;
    font-family: Menlo,serif;
  }
  .cards {
    display: flex;
    justify-content: space-between;
    margin: 0 -10px;
    @media  screen and (max-width: 400px) {
      flex-direction: column;
      .home-card {
        width: auto;
      }
    }
    @media  screen and (min-width: 400px) {
      flex-direction: row;
    }
    .home-card {
      height: auto;
    }
    .custom-card {
      p:nth-of-type(n+2) {
        margin: 2px 0;
        color: var(--vp-c-text-2);
      }
      .title {
        font-weight: 600;
        margin-bottom: 10px;
      }
    }
  }
}
</style>