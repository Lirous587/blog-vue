<template>
  <div v-loading="tableLoading">
    <div class="flex flex-col mx-3 mt-3">
      <div class="mb-2" style="z-index: 999">
        <el-button circle @click="dialogVisibleRef = true">
          <el-icon size="25px">
            <Search />
          </el-icon>
        </el-button>
      </div>
      <essayEdit
        mode="edit"
        v-model:previewRef="previewRef"
        v-model:editContent="form.content"
      />
    </div>

    <div class="bottom-3 fixed" style="z-index: 999">
      <el-button
        type="primary"
        size="large"
        @click="updateEssayPre"
        class="ml-3"
        >修改文章</el-button
      >
    </div>
  </div>

  <!-- 修改文章的窗口 -->
  <el-dialog
    v-model="dialogVisibleRef"
    :v-close-on-click-modal="true"
    :show-close="false"
    append-to-body
    width="80%"
  >
    <!-- 数据列表 -->
    <el-card shadow="always">
      <template #header>
        <el-input
          v-model="searchInput"
          placeholder="搜索文档"
          style="height: 50px"
        >
          <template #prefix>
            <el-icon size="20px" class="mx-2">
              <search />
            </el-icon>
          </template>
          <template #suffix>
            <el-button type="primary" @click="getEssayList">搜索</el-button>
          </template>
        </el-input>
      </template>
      <el-table
        v-loading="essayListLoading"
        :data="tableDate"
        border
        stripe
        size="large"
      >
        <el-table-column label="文章名" align="center" prop="name" />
        <el-table-column label="分类名" align="center" prop="kind" />
        <el-table-column label="文章路由" align="center" prop="router" />
        <el-table-column label="文章介绍" align="center">
          <template #default="{ row }">
            <el-text line-clamp="2">
              {{ row.introduction }}
            </el-text>
          </template>
        </el-table-column>
        <el-table-column label="操作" align="center" width="200px">
          <template #default="{ row }">
            <div>
              <el-button
                type="primary"
                size="small"
                text
                @click="chooseEssay(row)"
                >选择文章</el-button
              >
              <el-button
                type="primary"
                size="small"
                text
                @click="deletedEssyConfirm(row)"
                >删除文章</el-button
              >
            </div>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </el-dialog>

  <!-- 修改文章的抽屉 -->
  <el-drawer
    v-model="drawerVisiableRef"
    title="修改文章"
    append-to-body
    destroy-on-close
    size="45%"
  >
    <el-form :model="form" label-width="80px" :inline="false">
      <el-form-item label="分类">
        <el-select v-model="form.kind" placeholder="选择分类">
          <el-option
            v-for="item in classifyList"
            :key="item.name"
            :label="item.name"
            :value="item.name"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="文章名">
        <el-input v-model="form.name" placeholder="文章名" />
      </el-form-item>
      <el-form-item label="路由">
        <el-input v-model="form.router" placeholder="路由" />
      </el-form-item>
      <el-form-item label="介绍">
        <el-input v-model="form.introduction" placeholder="介绍" />
      </el-form-item>
      <el-form-item label="关键词">
        <dynamicAddTag v-model:tags="form.keywords"> </dynamicAddTag>
      </el-form-item>

      <el-form-item label="文章图片">
        <uploadImg v-model:imgUrl="form.imgUrl" ref="uploadImgRef"></uploadImg>
      </el-form-item>

      <el-form-item label="广告图片">
        <uploadImg
          v-model:imgUrl="form.advertiseImg"
          ref="uploadImgRef2"
        ></uploadImg>
      </el-form-item>

      <el-form-item label="广告信息">
        <el-input v-model="form.advertiseMsg" placeholder="广告信息" />
      </el-form-item>

      <el-form-item label="广告链接">
        <el-input v-model="form.advertiseHref" placeholder="介绍" />
      </el-form-item>

      <el-form-item>
        <el-button
          type="primary"
          size="large"
          style="width: 100%"
          @click="handelUpdate"
          class="mt-5"
          :loading="btnLoading"
          >修改</el-button
        >
      </el-form-item>
    </el-form>
  </el-drawer>
</template>

<script setup>
import { ref, reactive, watch, defineAsyncComponent } from "vue";
import { toast, listenScreen } from "~/composables/util.js";
import { ElMessageBox } from "element-plus";
import { updateEssayMsg, deleteEssay } from "~/api/manager.js";
import { getEssayMsg } from "~/api/user.js";
import { searchData } from "~/api/keyword.js";
import { useCommonForm, useCommonData } from "~/composables/useCommon.js";
const essayEdit = defineAsyncComponent(() =>
  import("~/components/essayEdit.vue")
);
const dynamicAddTag = defineAsyncComponent(() =>
  import("~/components/dynamicAddTag.vue")
);
const uploadImg = defineAsyncComponent(() =>
  import("~/components/uploadImg.vue")
);
const { classifyList } = useCommonData();

const uploadImgRef = ref(null);
const uploadImgRef2 = ref(null);

const {
  form,
  btnLoading,
  tableLoading,
  drawerVisiableRef,
  id,
  handelUpdate,
  handelDelete,
} = useCommonForm({
  form: reactive({
    content: "",
    kind: "",
    name: "",
    router: "",
    introduction: "",
    keywords: [],
    imgUrl: "",
    advertiseMsg: "",
    advertiseImg: "",
    advertiseHref: "",
  }),
  update: {
    needCustomizeDispose: (form) => {
      return new Promise((resolve, reject) => {
        updateEssayMsg(form)
          .then(() => uploadImgRef.value.submitUpload())
          .then(() => uploadImgRef2.value.submitUpload())
          .then(() => {
            resolve();
          })
          .catch((error) => reject(error));
      });
    },
  },
  delete: deleteEssay,
});

const dialogVisibleRef = ref(false);
const previewRef = ref({});

//搜索数据
const searchInput = ref("");
let tableDate = ref([]);
const essayListLoading = ref(false);

function getEssayList() {
  tableDate.value = [];
  if (searchInput.value == "") {
    toast("请输入搜索内容", "warning");
    return;
  }
  essayListLoading.value = true;
  searchData({ keyword: searchInput.value })
    .then((res) => {
      tableDate.value = res;
      if (!res) {
        toast("没有相关文章", "warning");
      } else {
        toast(`查找到${tableDate.value.length}篇相关文章`);
      }
    })
    .finally(() => {
      essayListLoading.value = false;
    });
}

const { handelOnKeyUp } = listenScreen({
  onKeyUp: {
    visiable: dialogVisibleRef,
    getData: getEssayList,
  },
});

watch(
  () => dialogVisibleRef.value,
  (data) => {
    data
      ? document.addEventListener("keyup", handelOnKeyUp)
      : document.removeEventListener("keyup", handelOnKeyUp);
  }
);

// 选择文章 加载数据
const updatePermission = ref(false);
const chooseEssay = (essay) => {
  updatePermission.value = true;
  tableLoading.value = true;
  getEssayMsg(essay.id).then((res) => {
    for (let k in res) {
      form[k] = res[k];
    }
    tableLoading.value = false;
  });
  dialogVisibleRef.value = false;
};

// 删除文章
const deletedEssyConfirm = async (essay) => {
  id.value = essay.id;
  try {
    await ElMessageBox.confirm("确定要删除文章?", "删除文章", {
      confirmButtonText: "确定",
      cancelButtonText: "取消",
      type: "warning",
    });
    handelDelete(id);
  } catch (error) {
    toast("取消删除文章", "warning");
  }
};

// 打开更新文章前处理
function updateEssayPre() {
  if (updatePermission.value) {
    drawerVisiableRef.value = true;
    return;
  }
  toast("请先选择文章", "warning");
}
</script>
