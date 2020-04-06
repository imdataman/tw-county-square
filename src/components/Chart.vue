<template>
  <div id="chartWrapper">
    <div
      class="chart"
      v-for="g in grid"
      :key="g"
      :class="{ county: countyMap.get(g) }"
    >
      <div class="infoWrapper">
        <span class="countyName">{{ countyMap.get(g) }}</span>
        <span class="totalCase">{{ `${totalCases[g]}` }}</span>
      </div>
      <svg :viewBox="`0, 0, ${width}, ${height}`">
        <rect
          class="background"
          :x="xScale(+weeks[0])"
          :y="yScale(maxWeeklyCases) - margin.top - 2"
          :height="yScale(0)"
          :width="width"
        ></rect>
        <rect
          class="caseBar"
          v-for="d in weeklyCases[g]"
          :key="d[0]"
          :x="xScale(+d[0])"
          :y="yScale(+d[1])"
          :height="yScale(0) - yScale(+d[1])"
          :width="barWidth"
        ></rect>
        <text
          class="monthLabel"
          v-for="m in months"
          :key="`${m[0]}月`"
          :y="yScale(0) + 13"
          :x="xScale(m[1] + 2)"
        >
          {{ m[0] === 2 ? `${m[0]}月` : m[0] }}
        </text>
        <text
          class="caseLabel"
          :x="
            weeklyCases[g].find(d => d[1] === max(weeklyCases[g], d => d[1]))
              ? xScale(
                  +weeklyCases[g].find(
                    d => d[1] === max(weeklyCases[g], d => d[1])
                  )[0]
                ) +
                barWidth / 2
              : 0
          "
          :y="
            max(weeklyCases[g], d => d[1])
              ? yScale(max(weeklyCases[g], d => d[1])) - 2
              : 0
          "
        >
          {{ max(weeklyCases[g], d => d[1]) }}
        </text>
      </svg>
    </div>
  </div>
</template>

<script>
import json from "@/assets/Weekly_Age_County_Gender_19CoV.json";
import { max, rollups, sum, range } from "d3-array";
import { scaleLinear, scaleBand } from "d3-scale";

export default {
  name: "Chart",
  data() {
    return {
      grid: Array.from(Array(25).keys()),
      // eslint-disable-next-line prettier/prettier
      countyOrder: [1, 2, 3, 6, 7, 8, 11, 12, 13, 16, 17, 18, 21, 22, 23, 9, 14, 19, 24, 5, 10, 15],
      // eslint-disable-next-line prettier/prettier
      counties: ["基隆市", "台北市", "新北市", "桃園市", "新竹市", "新竹縣", "苗栗縣", "台中市", "彰化縣", "雲林縣", "嘉義市", "嘉義縣", "台南市", "高雄市", "屏東縣", "宜蘭縣", "南投縣", "花蓮縣", "台東縣", "連江縣", "金門縣", "澎湖縣"],
      countyData: json,
      width: 150,
      height: 150,
      margin: { top: 20, bottom: 20, left: 0, right: 0 }
    };
  },
  computed: {
    countyMap() {
      return new Map(this.countyOrder.map((d, i) => [d, this.counties[i]]));
    },
    dataParsed() {
      const groupedData = rollups(
        this.countyData,
        v => sum(v, j => +j["確定病例數"]),
        d => d["縣市"],
        d => d["診斷週別"]
      );

      return groupedData;
    },
    totalCases() {
      return this.grid.map(g =>
        this.dataParsed.find(d => d[0] === this.countyMap.get(g))
          ? sum(
              this.dataParsed.find(d => d[0] === this.countyMap.get(g))[1],
              d => d[1]
            )
          : 0
      );
    },
    weeklyCases() {
      return this.grid.map(g =>
        this.dataParsed.find(d => d[0] === this.countyMap.get(g))
          ? this.dataParsed.find(d => d[0] === this.countyMap.get(g))[1]
          : []
      );
    },
    maxWeeklyCases() {
      return max(
        this.weeklyCases.reduce((a, b) => [...a, ...b]).map(d => d[1])
      );
    },
    weeks() {
      return [...new Set(this.countyData.map(d => +d["診斷週別"]))].sort(
        (a, b) => a - b
      );
    },
    months() {
      return range(
        1,
        Math.floor(this.weeks[this.weeks.length - 1] / 4) + 1
      ).map(d => [d + 1, d * 4]);
    },
    xScale() {
      return scaleBand()
        .domain(this.weeks)
        .range([this.margin.left, this.width])
        .padding(0.1);
    },
    barWidth() {
      return this.xScale.bandwidth();
    },
    yScale() {
      return scaleLinear()
        .domain([0, this.maxWeeklyCases])
        .nice()
        .range([this.height - this.margin.bottom, this.margin.top]);
    }
  },
  created() {
    window.addEventListener("resize", this.resized);
  },
  destroyed() {
    window.removeEventListener("resize", this.resized);
  },
  methods: {
    resized() {
      if (window.innerWidth <= 480) {
        // eslint-disable-next-line prettier/prettier
        this.countyOrder = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21];
      } else {
        // eslint-disable-next-line prettier/prettier
        this.countyOrder = [1, 2, 3, 6, 7, 8, 11, 12, 13, 16, 17, 18, 21, 22, 23, 9, 14, 19, 24, 5, 10, 15];
      }
    },
    sum: sum,
    max: max
  }
};
</script>

<style scoped lang="scss">
#chartWrapper {
  padding-top: 20px;
  display: flex;
  justify-content: space-evenly;
  flex-wrap: wrap;
}

.chart {
  display: inline-block;
  width: calc(20% - 12px);
  margin-bottom: 8px;
  vertical-align: top;
  .infoWrapper {
    position: absolute;
    left: 3px;
    top: 0;
    display: none;
    text-align: center;
    span {
      display: block;
    }
    .countyName {
      font-weight: 500;
      font-size: 1rem;
      color: black;
    }
    .totalCase {
      line-height: 1.25rem;
      color: orange;
      font-size: 1.25rem;
    }
  }
  svg {
    display: block;
    .background {
      fill: none;
    }
    text {
      text-anchor: middle;
      display: none;
    }
    .monthLabel,
    .caseLabel {
      font-size: 0.75rem;
    }
    .caseBar {
      fill: orange;
    }
  }
}

.chart:before {
  content: "";
  float: left;
  padding-top: 100%;
}

.county {
  position: relative;
  .infoWrapper {
    display: initial;
  }
  svg {
    .background {
      fill: floralwhite;
    }
    text {
      display: initial;
    }
  }
}

@media only screen and (max-width: 480px) {
  .chart {
    width: calc(33.3vw - 14px);
    height: calc(33.3vw - 14px);
    display: none;
    margin-bottom: 10px;
    .infoWrapper {
      .countyName {
        font-size: 0.75rem;
      }
      .totalCase {
        line-height: 1rem;
        font-size: 1rem;
      }
    }
  }

  .county {
    display: inline-block;
  }
}
</style>
