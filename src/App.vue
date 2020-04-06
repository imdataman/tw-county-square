<template>
  <div id="app">
    <div id="header">
      <h1 class="title">新冠肺炎各縣市每週發病病例</h1>
      <p>
        新冠肺炎疫情持續延燒，截至{{ updateDate }}台灣已經有<span
          class="totalCase"
          >{{ twTotalCase }}</span
        >例。各縣市疫情曲線是向上發展還是維持平穩，儀表板報給你知。
      </p>
      <span class="credit"
        >製作：<a href="https://www.facebook.com/imDataMan/" target="_blank"
          >林佳賢</a
        ><a href="https://github.com/imdataman" target="_blank"
          ><img src="@/assets/GitHub-Mark-32px.png"/></a
      ></span>
      <span class="updateTime">更新時間：2020月{{ updateDate }}</span>
    </div>
    <Chart :json="json" />
    <div id="footer">
      <span class="dataSource">
        資料來源：<a href="https://data.gov.tw/dataset/118038" target="_blank"
          >疾管署</a
        >
      </span>
      <span class="sourceCode"
        ><a href="https://github.com/imdataman/tw-county-square" target="_blank"
          >網頁原始碼</a
        ></span
      >
    </div>
  </div>
</template>

<script>
import json from "@/assets/Weekly_Age_County_Gender_19CoV.json";
import Chart from "./components/Chart.vue";

const updateDate = "4月5日";

export default {
  name: "App",
  components: {
    Chart
  },
  data() {
    return {
      json: json,
      updateDate: updateDate
    };
  },
  computed: {
    twTotalCase() {
      return this.json.map(d => +d["確定病例數"]).reduce((a, b) => a + b);
    }
  }
};
</script>

<style scoped lang="scss">
body {
  margin: 0;
}

a {
  text-decoration: none;
  color: Navy;
}

a:hover {
  color: orange;
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
  color: white;
  background-color: orange;
  padding: 0 5px;
  margin: 0 2px;
  border-radius: 4px;
  font-weight: 500;
}

.updateTime,
.credit,
.dataSource,
.sourceCode {
  font-size: 0.75rem;
}

.credit {
  img {
    height: 0.75rem;
    padding-left: 0.25rem;
    vertical-align: middle;
  }
}

#footer {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  grid-template-areas: "dataSource sourceCode none";
}

.sourceCode {
  text-align: center;
  grid-area: sourceCode;
}
</style>
