<template>
  <div class="relative" ref="containerRef">
    <div
      @click="toggleDropdown"
      class="flex items-center border border-gray-300 rounded px-3 py-1.5"
      :class="{ 'border-blue-400': opened || focused }"
    >
      <svg width="20px" height="30px" viewBox="0 0 24 24" fill="none">
        <path
          fill-rule="evenodd"
          clip-rule="evenodd"
          d="M17.0392 15.6244C18.2714 14.084 19.0082 12.1301 19.0082 10.0041C19.0082 5.03127 14.9769 1 10.0041 1C5.03127 1 1 5.03127 1 10.0041C1 14.9769 5.03127 19.0082 10.0041 19.0082C12.1301 19.0082 14.084 18.2714 15.6244 17.0392L21.2921 22.707C21.6828 23.0977 22.3163 23.0977 22.707 22.707C23.0977 22.3163 23.0977 21.6828 22.707 21.2921L17.0392 15.6244ZM10.0041 17.0173C6.1308 17.0173 2.99087 13.8774 2.99087 10.0041C2.99087 6.1308 6.1308 2.99087 10.0041 2.99087C13.8774 2.99087 17.0173 6.1308 17.0173 10.0041C17.0173 13.8774 13.8774 17.0173 10.0041 17.0173Z"
          fill="#0F0F0F"
        />
      </svg>
      <input
        v-model="search"
        @focus="focused = true"
        @blur="focused = false"
        class="flex-1 outline-none mx-2 text-md"
        :placeholder="placeholder"
      />

      <svg class="transition-all" :class="{ 'rotate-180': opened || focused }" width="15px" height="15px"
           viewBox="0 0 24 24" fill="none">
        <path fill-rule="evenodd" clip-rule="evenodd"
              d="M4.29289 8.29289C4.68342 7.90237 5.31658 7.90237 5.70711 8.29289L12 14.5858L18.2929 8.29289C18.6834 7.90237 19.3166 7.90237 19.7071 8.29289C20.0976 8.68342 20.0976 9.31658 19.7071 9.70711L12.7071 16.7071C12.3166 17.0976 11.6834 17.0976 11.2929 16.7071L4.29289 9.70711C3.90237 9.31658 3.90237 8.68342 4.29289 8.29289Z"
              fill="#000000"/>
      </svg>
    </div>
    <div
      v-if="opened || focused"
      class="shadow rounded bg-white absolute top-full w-full mt-1 py-1 overflow-hidden z-10"
    >
      <template v-for="(group, index) in filteredGroupedOptions">
        <!-- Displaying Group Label -->
        <div v-if="group.options.length > 0" class="font-bold px-4 py-2">{{ group.label }}</div>

        <!-- Displaying filtered options -->
        <div v-if="group.options.length === 0" class="px-4 py-2 text-gray-400">No items</div>

        <div
          v-for="option in group.options"
          :key="option.value"
          @click.prevent="handleSelect(option)"
          class="px-4 py-2 hover:bg-gray-200 cursor-pointer"
        >
          <span class="text-sm">{{ option.label }}</span>
        </div>
      </template>
    </div>
  </div>
</template>

<script setup>
import {ref, computed, watch} from 'vue';

const props = defineProps({
  placeholder: String,
  modelValue: String,
  options: {
    type: Array,
    default: () => [],
  },
  groupBy: {
    type: Function,
    default: (option) => option.group
  }
});

const emit = defineEmits(['update:modelValue']);

const getOptionLabel = (val) => {
  if (!val) return '';
  const foundOption = props.options.find((option) => option.value === val);
  return foundOption ? foundOption.label : '';
};

const opened = ref(false);
const focused = ref(false);
const search = ref(getOptionLabel(props.modelValue));

const value = computed({
  get: () => props.modelValue,
  set: (newVal) => {
    emit('update:modelValue', newVal);
  },
});

const filteredOptions = (options) => {
  return options.filter((option) => option.label.toLowerCase().includes(search.value.toLowerCase()));
};

const filteredGroupedOptions = computed(() => {
  const groups = {};

  props.options.forEach(option => {
    const groupLabel = props.groupBy(option);
    if (!groups[groupLabel]) {
      groups[groupLabel] = {label: groupLabel, options: []};
    }
    groups[groupLabel].options.push(option);
  });

  return Object.values(groups).map(group => ({
    label: group.label,
    options: filteredOptions(group.options),
  })).filter(group => group.options.length > 0);
});

const toggleDropdown = () => {
  opened.value = !opened.value;
  if (!opened.value) {
    search.value = getOptionLabel(value.value);
  }
};

const handleSelect = (option) => {
  value.value = option.value;
  search.value = option.label;
  opened.value = false;
};

watch(value, (newVal) => {
  search.value = getOptionLabel(newVal);
});

const clickOutside = (event) => {
  if (!containerRef.value) return;
  if (!containerRef.contains(event.target)) {
    opened.value = false;
  }
};

watch(focused, (newValue) => {
  if (!newValue && !opened.value) {
    search.value = getOptionLabel(value.value);
  }
});

watch(opened, (newValue) => {
  if (newValue) {
    document.addEventListener('click', clickOutside);
  } else {
    document.removeEventListener('click', clickOutside);
  }
});
</script>

<style scoped>
.relative {
  width: fit-content; /* Adjust width as needed */
}
</style>
