<script setup>
import { ref } from "vue";

const prob = ref(99);
const spins = ref(100);
const cost = ref(1000);
const result = ref(null);
const message = ref("");
const isLimited = ref(false);

const calc = () => {
  const isPaid = localStorage.getItem("isPaid") === "true";
  message.value = "";

  if (!isPaid && getCount() >= LIMIT) {
    message.value = "無料回数を超えました";
    isLimited.value = true;
    return;
  }

  const p = Number(prob.value);
  const s = Number(spins.value);
  const c = Number(cost.value);

  if (p <= 0 || s <= 0 || c < 0) {
    message.value = "入力エラー";
    return;
  }

  const expectedHits = s / p;
  const avgPayout = 1000;
  const expectedReturn = expectedHits * avgPayout;

  result.value = Math.round(expectedReturn - c);

  if (!isPaid) {
    incrementCount();
  }
};

const LIMIT = 3;

const todayKey = () => {
  const d = new Date();
  return `count-${d.getFullYear()}-${d.getMonth()}-${d.getDate()}`;
};

const getCount = () => {
  return Number(localStorage.getItem(todayKey()) || 0);
};

const incrementCount = () => {
  localStorage.setItem(todayKey(), getCount() + 1);
};

const goToPayment = () => {
  window.location.href = "ここにStripeのPayment Link";
};
</script>

<template>
  <div class="container">
    <div class="container">
      <h1>期待値シミュレーター</h1>

      <p>
        本サービスは、確率や数値の関係を理解するための学習・計算用Webツールです。<br />
        統計・確率の考え方を可視化することを目的としており、<br />
        ギャンブルや投機行為を推奨・支援するものではありません。<br />
        本ツールは教育・学習用途として提供されています。<br />
      </p>

      <p>無料利用回数を超えた場合、100円の買い切り決済で 継続利用できます。</p>

      <!-- <p class="contact">お問い合わせ：your-email@gmail.com</p> -->

      <p>
        ※期待値は理論上の数値であり、実際の結果を保証するものではありません。
      </p>
    </div>

    <div>
      <label>確率（例：99）</label>
      <input type="number" v-model="prob" />
    </div>

    <div>
      <label>回転数</label>
      <input type="number" v-model="spins" />
    </div>

    <div>
      <label>投資金額（円）</label>
      <input type="number" v-model="cost" />
    </div>

    <button @click="calc">計算する</button>

    <div v-if="result !== null">
      <h2>
        期待値：
        <span :style="{ color: result >= 0 ? 'green' : 'red' }">
          {{ result }} 円
        </span>
      </h2>
    </div>
    <div v-if="isLimited">
      <p v-if="message" style="color: red">
        {{ message }}
      </p>
      <button @click="goToPayment">有料版を利用する（100円）</button>
    </div>

    <a href="/tokushoho.html">特定商取引法に基づく表記</a>
  </div>
</template>

<style>
.container {
  max-width: 600px;
  margin: 40px auto;
  font-family: sans-serif;
}
input {
  width: 100%;
  margin-bottom: 12px;
}
button {
  width: 100%;
  padding: 8px;
}
</style>
