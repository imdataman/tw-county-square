<template>
  <div id="app">
    <div id="header">
      <h2 class="title">新冠肺炎各縣市每週發病病例</h2>
      <p>
        新冠肺炎疫情持續延燒，截至{{ updateDate }}台灣已經有<span
          class="totalCase"
          >{{ twTotalCase }}</span
        >例，單日增加<span class="newCase">{{
          twTotalCase - oldTwTotalCase
        }}</span
        >例。各縣市疫情曲線是向上發展還是維持平穩，儀表板報給你知。
      </p>
      <div id="legend">
        <div class="importLegend"></div>
        <span>境外移入</span>
        <div class="localLegend"></div>
        <span>本土傳染</span>
        <div class="lastDay"></div>
        <span>前日值</span>
      </div>
    </div>
    <Chart :json="json" :oldJson="oldJson" />
    <div id="footer">
      <span class="dataSource">
        資料來源：<a href="https://data.gov.tw/dataset/118038" target="_blank"
          >疾管署</a
        >
      </span>
      <span class="sourceCode"
        ><span class="credit"
          >製作：<a href="https://www.facebook.com/imDataMan/" target="_blank"
            >林佳賢</a
          ><a
            href="https://github.com/imdataman/tw-county-square"
            target="_blank"
            ><img src="@/assets/GitHub-Mark-32px.png"/></a></span
      ></span>
    </div>
  </div>
</template>

<script>
import json from "@/assets/Weekly_Age_County_Gender_19CoV.json";
import oldJson from "@/assets/old.json";
import Chart from "./components/Chart.vue";

const updateDate = "4月11日";

export default {
  name: "App",
  components: {
    Chart
  },
  data() {
    return {
      json: json,
      oldJson: oldJson,
      updateDate: updateDate
    };
  },
  computed: {
    twTotalCase() {
      return this.json.map(d => +d["確定病例數"]).reduce((a, b) => a + b);
    },
    oldTwTotalCase() {
      return this.oldJson.map(d => +d["確定病例數"]).reduce((a, b) => a + b);
    }
  },
  created() {
    document.querySelector("body").classList.add("bodyStyle");
  },
  destroyed() {
    document.querySelector("body").classList.remove("bodyStyle");
  }
};
</script>

<style>
.bodyStyle {
  margin: 0;
  background-color: floralwhite;
}
</style>

<style scoped lang="scss">
$feature-color: darkorange;

h2 {
  margin: 1.25rem 0 0.75rem;
}

a {
  text-decoration: none;
  color: Navy;
  &:hover {
    color: $feature-color;
  }
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
  padding-bottom: 10px;
}

.totalCase {
  color: white;
  background-color: $feature-color;
  padding: 0 5px;
  margin: 0 2px;
  border-radius: 4px;
  font-weight: 500;
}

.newCase {
  @extend .totalCase;
  background-color: crimson;
}

.updateTime,
.credit,
.dataSource,
.sourceCode {
  font-size: 0.75rem;
}

#legend {
  display: flex;
  padding-bottom: 0.5rem;
  span {
    text-align: center;
    padding-right: 1rem;
  }
  .importLegend {
    @extend .totalCase;
    width: 10px;
    height: 20px;
    display: inline-block;
    background-color: #ffc966;
  }
  .localLegend {
    @extend .importLegend;
    background-color: $feature-color;
  }
  .lastDay {
    padding: 0 5px;
    margin: 0 2px;
    width: 10px;
    height: 20px;
    border-radius: 4px;
    border-width: 1px;
    border-style: dashed;
  }
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
