<template>
  <div class="flex flex-col justify-center items-center px-15">
    <div class="flex" style="width: 100%">
      <el-card shadow="never" style="width: 100%">
        <template #header>
          <div class="flex justify-between">
            <span class="text-sm">关键字排行</span>
            <div>
              <el-check-tag
                v-for="(item, index) in options"
                :key="index"
                :checked="currentTag == item.value"
                @click="handerlChoose(item.value)"
                type="primary"
                style="margin-right: 8px"
              >
                {{ item.text }}
              </el-check-tag>
            </div>
          </div>
        </template>

        <div
          ref="el"
          id="chart"
          style="height: 300px; width: 100%"
          class=""
        ></div>
      </el-card>
    </div>

    <div class="flex mt-10" style="width: 100%">
      <el-card shadow="never" style="width: 100%; height: 350px">
        <template #header>
          <div class="flex justify-between items-center">
            <span class="text-sm">用户访问量</span>
            <el-tag type="danger" effect="plain"> 用户访问量 </el-tag>
          </div>
        </template>
        <el-row :gutter="20">
          <el-col :span="8" v-for="(item, index) in visitedCount" :key="index">
            <el-card
              shadow="never"
              class="border-0 bg-light-400 flex justify-center items-center"
            >
              <div
                class="flex flex-col justify-center items-center"
                style="height: 200px"
              >
                <span class="text-xl mb-2">{{ item.count }}</span>
                <span class="text-sm text-gray-500">{{ item.period }}</span>
              </div>
            </el-card>
          </el-col>
        </el-row>
      </el-card>
    </div>
  </div>

  <div class="background" :style="{ left: $store.state.adminAsideWidth }"></div>
</template>

<script setup>
import { ref, onMounted, onBeforeMount, reactive, nextTick } from "vue";
import * as echarts from "echarts/core";
import { BarChart } from "echarts/charts";
import {
  TooltipComponent,
  GridComponent,
  LegendComponent,
} from "echarts/components";
import { CanvasRenderer } from "echarts/renderers";

import { getIndexPanel } from "~/api/panel.js";
import { useResizeObserver } from "@vueuse/core";

// 注册需要的组件
echarts.use([
  BarChart,
  TooltipComponent,
  GridComponent,
  LegendComponent,
  CanvasRenderer,
]);

const options = [
  {
    text: "年排行",
    value: "year",
  },
  {
    text: "月排行",
    value: "month",
  },
  {
    text: "周排行",
    value: "week",
  },
];
const visitedCount = ref([
  {
    period: "用户年访问量",
    count: 0,
  },
  {
    period: "用户月访问量",
    count: 0,
  },
  {
    period: "用户周访问量",
    count: 0,
  },
]);

const keywordRank = reactive({
  year: {},
  month: {},
  week: {},
});
const currentTag = ref("year");

const getData = async () => {
  await getIndexPanel().then((res) => {
    let ipSet = res.ipSet;
    let rankList = res.rankList;

    visitedCount.value[0].count = ipSet.year;
    visitedCount.value[1].count = ipSet.month;
    visitedCount.value[2].count = ipSet.week;
    for (const time in rankList) {
      keywordRank[time] = rankList[time];
    }
  });
};

const handerlChoose = (tag) => {
  currentTag.value = tag;
  changeTimeTag(tag);
};

let myChart = null;

const el = ref(null);

function changeTimeTag(tag) {
  const tagData = keywordRank[tag];
  const xList = tagData.x;
  const yList = tagData.y;

  let option = {
    xAxis: {
      data: xList,
    },
    yAxis: {},
    legend: {
      data: ["搜索次数"],
    },
    series: [
      {
        name: "搜索次数",
        type: "bar",
        data: yList,
        label: {
          show: true,
          position: "top",
        },
        itemStyle: {
          color: "#59c4e6",
        },
        barWidth: 30,
        selectedMode: true,
        select: {
          itemStyle: {
            color: "#edafda",
          },
        },
      },
    ],
  };
  myChart.showLoading();
  myChart.setOption(option);
  myChart.hideLoading();
}

onBeforeMount(() => {
  if (myChart) {
    echarts.dispose(myChart);
  }
});

onMounted(async () => {
  await getData();
  let chartDom = document.getElementById("chart");
  if (chartDom) {
    myChart = echarts.init(chartDom);
    handerlChoose("year");
  }
  useResizeObserver(el, (entries) => {
    myChart.resize();
  });
});
</script>

<style scoped>
:deep(.el-card__body) {
  height: 100%;
}

.background {
  @apply fixed;
  right: 0;
  top: 0;
  bottom: 0;
  pointer-events: none;
  background: linear-gradient(
    to right top,
    rgba(97, 65, 104, 0.069),
    rgba(114, 92, 40, 0.081),
    rgba(23, 58, 185, 0.098)
  );
  transition: all 0.5s ease;
}
</style>
