<template>
  <main>
    <div class="max-w-2xl mx-auto px-4 py-5 pt-10">
      <Dropdown
        placeholder="Select Account"
        v-model="selectedValue"
        :options="
              [
                  { label: 'Option A', value: 'valA', group: 'Group A' },
                  { label: 'Option B', value: 'valB', group: 'Group B' },
                  { label: 'Option C', value: 'valC', group: 'Group A' }
              ]"
        :groupBy="(option) => option.group"
      />
    </div>
  </main>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import Dropdown from './components/Dropdown.vue';

const selectedValue = ref('');

</script>
