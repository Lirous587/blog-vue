<template>
  <div class="flex items-center" style="width: 100%">
    <div style="width: 10%" class="flex justify-center items-center">
      <el-icon size="24px">
        <component :is="modelValue"></component>
      </el-icon>
    </div>
    <div style="width: 90%">
      <el-select
        :modelValue="modelValue"
        class="ml-2"
        filterable
        placeholder="请选择图标"
        @change="hanlerChange"
      >
        <el-option
          v-for="item in icons"
          :key="item"
          :label="item"
          :value="item"
        >
          <div class="flex items-center justify-between">
            <el-icon :size="16">
              <component :is="item"></component>
            </el-icon>
            <span class="text-gray-500"> {{ item }}</span>
          </div>
        </el-option>
      </el-select>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from "vue";

const IconList = ref({});
const icons = computed(() => Object.keys(IconList.value));

const loadIcons = async () => {
  const module = await import("@element-plus/icons-vue");
  IconList.value = module;
};

loadIcons();

defineProps({
  modelValue: String,
});

const emit = defineEmits(["update:modelValue"]);

const hanlerChange = (icon) => {
  emit("update:modelValue", icon);
};
</script>
