<template>
  <essayList :essayList="tableData" :listUse="listUse"></essayList>
</template>

<script setup>
import { reactive, defineAsyncComponent, watch } from "vue";
import { getEssayList } from "~/api/user.js";
import { useCommonGetData } from "~/composables/useCommon.js";
import { useRoute } from "vue-router";
import { router } from "~/router";
const route = useRoute();
const essayList = defineAsyncComponent(() =>
  import("~/components/essayList.vue")
);

const { tableData, currentPage, totalPages, loading, getDataList } =
  useCommonGetData({
    form: {
      page: 1,
      pageSize: 5,
    },
    getDataList: getEssayList,
    loadingText: "文章列表渲染中",
  });

const changePage = (p) => {
  currentPage.value = p;
};

const listUse = reactive({
  loading,
  currentPage,
  totalPages,
});

const getdata = async (page) => {
  changePage(page ? parseInt(page) : 1);
  await getDataList();
  if (page > totalPages.value) {
    router.push("/404");
  }
};

getdata(route.query?.page ? parseInt(route.query?.page) : 1);

watch(
  () => route.query.page,
  (page) => {
    getdata(page);
  }
);
</script>
