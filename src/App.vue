<template>
  <div id="app">
    <div id="header">
      <h1 class="title">新冠肺炎各縣市每週新增病例</h1>
      <p>
        新冠肺炎疫情持續延燒，截至4月5日台灣已經有<span class="totalCase"
          >363</span
        >例。各縣市疫情曲線是向上發展還是維持平穩，儀表板報給你知。
      </p>
      <span class="credit"
        >製作：<a href="https://www.facebook.com/imDataMan/" target="_blank"
          >林佳賢</a
        ><a href="https://github.com/imdataman" target="_blank"
          ><img src="@/assets/GitHub-Mark-32px.png"/></a
      ></span>
      <span class="updateTime">更新時間：2020月4月5日</span>
    </div>
    <Chart />
    <div id="footer">
      <span class="dataSource">
        資料來源：<a href="https://data.gov.tw/dataset/118038" target="_blank"
          >衛生福利部疾病管制署</a
        >
      </span>
    </div>
  </div>
</template>

<script>
import Chart from "./components/Chart.vue";

export default {
  name: "App",
  components: {
    Chart
  }
};
</script>

<style scoped lang="scss">
body {
  margin: 0;
}

a {
  color: black;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  margin: auto;
  max-width: 950px;
}

#header,
#footer {
  margin-left: 0.5rem;
}

#footer {
  padding-bottom: 20px;
}

.updateTime {
  padding-left: 2rem;
}

.totalCase {
  color: orange;
}

.updateTime,
.credit {
  font-size: 0.75rem;
  img {
    height: 0.75rem;
    padding-left: 0.25rem;
    vertical-align: middle;
  }
}

.dataSource {
  font-size: 0.75rem;
}
</style>
