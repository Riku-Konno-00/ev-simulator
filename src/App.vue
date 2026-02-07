<script setup>
import { ref, computed, onMounted, watch, reactive } from "vue";

// 追加
const addPayout = () => {
  distributions.value.push({ balls: 1000, rate: 0 });
};

// 削除
const removePayout = (index) => {
  distributions.value.splice(index, 1);
};

// 合計％
const totalRate = computed(() =>
  distributions.value.reduce((sum, d) => sum + d.rate, 0),
);
/* =====================
   入力値
===================== */

// 通常時
const probability = ref(319); // 1/319
const rotationsPer1k = ref(18); // 1Kあたり回転数

// 初当たり
const initialPayout = ref(300); // 初当たり出玉
const rushEntryRate = ref(72); // RUSH突入率（%）

// RUSH
const rushContinueRate = ref(85); // 継続率（%）

// 出玉振り分け
const distributions = ref([{ rate: 100, balls: 2000 }]);
/* =====================
   出力値
===================== */
const result = ref(null);

/* =====================
   計算系
===================== */
const spinsList = computed(() => [100, 200, probability.value]);

const resultsBySpins = computed(() => {
  if (!result.value) return [];

  const costPerSpin = 1000 / rotationsPer1k.value;

  return spinsList.value.map((spins) => {
    const investment = spins * costPerSpin;
    const profit = result.value.expectedPayout - investment;

    return {
      spins,
      investment,
      profit,
    };
  });
});
function calculate() {
  const prob = Number(probability.value);
  const rot = Number(rotationsPer1k.value);
  const initPay = Number(initialPayout.value);
  const entryRate = Number(rushEntryRate.value) / 100;
  const contRate = Number(rushContinueRate.value) / 100;

  // 1回転あたりコスト
  const costPerSpin = 1000 / rot;

  // 初当たりまでの平均回転数（確率分母）
  const expectedSpinsToHit = prob;
  const expectedInvestment = expectedSpinsToHit * costPerSpin;

  // RUSH平均出玉
  const avgRushPayout = distributions.value
    .filter((d) => d.rate > 0 && d.balls > 0)
    .reduce((sum, d) => sum + (d.rate / 100) * d.balls, 0);

  // RUSHトータル期待出玉
  const rushTotalPayout = contRate >= 1 ? 0 : avgRushPayout / (1 - contRate);

  // 初当たり1回あたり期待出玉
  const expectedPayout = initPay + entryRate * rushTotalPayout;

  const profit = expectedPayout - expectedInvestment;

  result.value = {
    expectedSpinsToHit,
    expectedInvestment,
    expectedPayout,
    profit,
    breakEvenYen: expectedPayout * 4,
  };

  showResultModal.value = true;
}

const showModal = ref(false);

onMounted(() => {
  showModal.value = true;
});

function agree() {
  localStorage.setItem("agreed", "true");
  showModal.value = false;
}

watch(showModal, (val) => {
  document.body.style.overflow = val ? "hidden" : "";
});

const showResultModal = ref(false);

function close() {
  showResultModal.value = false;
}
</script>

<template>
  <div class="container">
    <!-- Modal -->
    <div v-if="showModal" class="modal">
      <div class="modal-content">
        <h2>このサイトについて</h2>
        <p>
          本シミュレーターは、パチンコ台の期待値を確率論に基づいて計算するツールです。
        </p>

        <h2>※注意事項</h2>
        <p>実際の結果を保証するものではありません。</p>
        <p>短期的な収支を保証するものではありません。</p>
        <p>あくまで判断材料の一つとしてご利用ください。</p>

        <button @click="agree">同意して使う</button>
      </div>
    </div>

    <h2>期待値シミュレーター</h2>

    <section class="input-area">
      <!-- 基本情報 -->
      <div class="input-card">
        <h3>基本情報</h3>

        <div class="input-row">
          <label>大当たり確率</label>
          <div class="input-with-unit">
            <span>1 /</span>
            <input v-model.number="probability" />　
          </div>
        </div>

        <div class="input-row">
          <label>1K回転数</label>
          <div class="input-with-unit">
            <input v-model.number="rotationsPer1k" />
            <span>回</span>
          </div>
        </div>
      </div>

      <!-- 初当たり -->
      <div class="input-card">
        <h3>初当たり</h3>

        <div class="input-row">
          <label>初当たり出玉</label>
          <div class="input-with-unit">
            <input v-model.number="initialPayout" />
            <span>発</span>
          </div>
        </div>

        <div class="input-row">
          <label>RUSH突入率</label>
          <div class="input-with-unit">
            <input v-model.number="rushEntryRate" />
            <span>%</span>
          </div>
        </div>
      </div>

      <!-- RUSH -->
      <div class="input-card">
        <h3>RUSH性能</h3>

        <div class="input-row">
          <label>継続率</label>
          <div class="input-with-unit">
            <input v-model.number="rushContinueRate" />
            <span>%</span>
          </div>
        </div>
      </div>

      <div class="container">
        <!-- 出玉振り分けカード -->
        <div class="card">
          <div class="card-header">
            <h3>出玉振り分け</h3>
          </div>

          <div v-for="(d, index) in distributions" :key="index" class="row">
            <input type="number" v-model.number="d.rate" min="0" max="100" />
            <span>%</span>

            <input type="number" v-model.number="d.balls" placeholder="出玉" />
            <span>発</span>

            <button
              class="remove"
              @click="removePayout(index)"
              v-if="distributions.length > 1"
            >
              -
            </button>
          </div>
          <button class="add" @click="addPayout">＋ 追加</button>

          <p class="warning" v-if="totalRate !== 100">
            合計：{{ totalRate }}%（100%にしてください）
          </p>
        </div>

        <!-- 円グラフ -->
        <!-- <div class="card chart-card">
          <Pie :data="chartData" :options="chartOptions" />
        </div> -->
      </div>
    </section>

    <button @click="calculate">計算する</button>

    <!-- ResultModal -->
    <div v-if="showResultModal" class="modal">
      <div class="modal-content">
        <section v-if="result">
          <h2>計算結果</h2>
          <p>初当たり期待出玉：{{ Math.round(result.expectedPayout) }} 発</p>
          <p>損益分岐：{{ Math.round(result.breakEvenYen) }} 円</p>

          <p>
            初当たりまでの平均投資額：{{
              Math.round(result.expectedInvestment)
            }}
            円
          </p>
        </section>
        <section>
          <h3>回転数別期待値</h3>

          <div class="horizontal-scroll">
            <div v-for="r in resultsBySpins" :key="r.spins" class="result-card">
              <h4>{{ r.spins }}回転で当たり</h4>

              <p>投資額</p>
              <strong>{{ Math.round(r.investment) }}円</strong>

              <p>期待収支</p>
              <strong :style="{ color: r.profit >= 0 ? 'green' : 'red' }">
                {{ Math.round(r.profit) }}円
              </strong>
            </div>
          </div>
        </section>

        <button @click="close">閉じる</button>
      </div>
    </div>
  </div>
</template>
