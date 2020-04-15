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
        <span class="newCase">{{
          totalCases[g] - oldTotalCases[g] || ""
        }}</span>
      </div>
      <svg :viewBox="`0, 0, ${width}, ${height}`">
        <rect
          class="background"
          x="0"
          y="0"
          :height="height - margin.bottom"
          :width="width"
        ></rect>
        <g v-for="(d, i) in stackedData[g]" :key="`${i}stack`">
          <rect
            v-for="(j, k) in d"
            :key="`${i}${k}stack`"
            :class="j[2] === '否' ? 'localCase' : 'importCase'"
            :x="xScale(j[3])"
            :y="yScale(j[1])"
            :height="yScale(j[0]) - yScale(j[1])"
            :width="barWidth"
          ></rect>
        </g>
        <rect
          class="oldCaseBar"
          v-for="(d, i) in oldWeeklyCases[g]"
          :key="`${d[0]}old`"
          v-show="d[1] !== weeklyCases[g][i][1]"
          :x="xScale(+d[0])"
          :y="yScale(+d[1])"
          :height="height - margin.bottom - yScale(+d[1])"
          :width="barWidth"
        ></rect>
        <text
          class="monthLabel"
          v-for="m in months"
          :key="`${m[0]}月`"
          :y="height - margin.bottom + 13"
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
import { max, rollups, sum, range } from "d3-array";
import { scaleLinear, scaleBand } from "d3-scale";
import { stack } from "d3-shape";

export default {
  name: "Chart",
  data() {
    return {
      grid: Array.from(Array(25).keys()),
      countyOrder: [],
      // eslint-disable-next-line prettier/prettier
      desktopLayout: [1, 2, 3, 6, 7, 8, 11, 12, 13, 16, 17, 18, 21, 22, 23, 9, 14, 19, 24, 5, 10, 15],
      // eslint-disable-next-line prettier/prettier
      mobileLayout: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21],
      // eslint-disable-next-line prettier/prettier
      counties: ["基隆市", "台北市", "新北市", "桃園市", "新竹市", "新竹縣", "苗栗縣", "台中市", "彰化縣", "雲林縣", "嘉義市", "嘉義縣", "台南市", "高雄市", "屏東縣", "宜蘭縣", "南投縣", "花蓮縣", "台東縣", "連江縣", "金門縣", "澎湖縣"],
      width: 150,
      height: 150,
      margin: { top: 20, bottom: 20, left: 0, right: 0 }
    };
  },
  props: {
    json: Array,
    oldJson: Array
  },
  computed: {
    countyMap() {
      return new Map(this.countyOrder.map((d, i) => [d, this.counties[i]]));
    },
    dataParsed() {
      let groupedData = rollups(
        this.json,
        v => sum(v, j => +j["確定病例數"]),
        d => d["縣市"],
        d => d["研判週別"]
      );
      groupedData = groupedData.map(d => [
        d[0],
        d[1].sort((a, b) => +a[0] - +b[0])
      ]);
      return groupedData;
    },
    oldDataParsed() {
      let groupedData = rollups(
        this.oldJson,
        v => sum(v, j => +j["確定病例數"]),
        d => d["縣市"],
        d => d["研判週別"]
      );
      groupedData = groupedData.map(d => [
        d[0],
        d[1].sort((a, b) => +a[0] - +b[0])
      ]);
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
    oldTotalCases() {
      return this.grid.map(g =>
        this.oldDataParsed.find(d => d[0] === this.countyMap.get(g))
          ? sum(
              this.oldDataParsed.find(d => d[0] === this.countyMap.get(g))[1],
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
    oldWeeklyCases() {
      return this.grid.map(g =>
        this.oldDataParsed.find(d => d[0] === this.countyMap.get(g))
          ? this.oldDataParsed.find(d => d[0] === this.countyMap.get(g))[1]
          : []
      );
    },
    maxWeeklyCases() {
      return max(
        this.weeklyCases.reduce((a, b) => [...a, ...b]).map(d => d[1])
      );
    },
    stackedData() {
      let groupedData = rollups(
        this.json,
        v => sum(v, j => +j["確定病例數"]),
        d => d["縣市"],
        d => d["研判週別"],
        d => d["是否為境外移入"]
      );

      groupedData = groupedData.map(d => [
        d[0],
        d[1]
          .map(j => ({ week: j[0], ...Object.fromEntries(j[1]) }))
          // 確保不會有NaN
          .map(j => ("是" in j ? j : { ...j, 是: 0 }))
          .map(j => ("否" in j ? j : { ...j, 否: 0 }))
      ]);

      groupedData = groupedData.map(d => [
        d[0],
        stack()
          .keys(["否", "是"])(d[1])
          .map(
            j => (
              j.forEach(v => {
                // 因為Vue會自己把array的property拿掉，所以這裡要自己手動新增key跟week
                v[2] = j.key;
                v[3] = v.data.week;
              }),
              j
            )
          )
      ]);

      return this.grid.map(g =>
        groupedData.find(d => d[0] === this.countyMap.get(g))
          ? groupedData.find(d => d[0] === this.countyMap.get(g))[1]
          : []
      );
    },
    weeks() {
      return [...new Set(this.json.map(d => +d["研判週別"]))].sort(
        (a, b) => a - b
      );
    },
    months() {
      return range(
        1,
        Math.floor(this.weeks[this.weeks.length - 1] / 4)
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
        this.countyOrder = this.mobileLayout;
      } else {
        this.countyOrder = this.desktopLayout;
      }
    },
    sum: sum,
    max: max
  },
  mounted() {
    this.resized();
  }
};
</script>

<style scoped lang="scss">
#chartWrapper {
  padding-top: 10px;
  display: flex;
  justify-content: space-evenly;
  flex-wrap: wrap;
}

.chart {
  width: calc(20% - 12px);
  margin-bottom: 6px;
  position: relative;
  .infoWrapper {
    position: absolute;
    left: 3px;
    top: 0;
    display: none;
    text-align: center;
    span {
      display: block;
    }
    .totalCase {
      font-weight: 500;
      line-height: 1.25rem;
      font-size: 1.25rem;
      color: $feature-color;
    }
    .newCase {
      @extend .totalCase;
      color: $new-color;
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
    .localCase {
      fill: $feature-color;
    }
    .importCase {
      fill: $import-color;
    }
    .oldCaseBar {
      fill: none;
      stroke: black;
      stroke-width: 1px;
      stroke-dasharray: 2;
    }
  }
}

.county {
  .infoWrapper {
    display: initial;
  }
  svg {
    .background {
      fill: $chart-bg-color;
    }
    text {
      display: initial;
    }
  }
}

@media only screen and (max-width: 480px) {
  .chart {
    width: calc(33.3vw - 10px);
    height: calc(33.3vw - 10px);
    display: none;
    margin-bottom: 5px;
    .infoWrapper {
      .countyName {
        font-size: 0.75rem;
      }
      .totalCase {
        font-size: 1rem;
      }
    }
  }

  .county {
    display: initial;
  }
}
</style>
