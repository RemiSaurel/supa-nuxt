<script setup>
const client = useSupabaseClient();
const countries = ref([]);

async function getCountries() {
  const { data } = await client.from("countries").select();
  countries.value = data;
}

onMounted(() => {
  getCountries();
});
</script>

<template>
  <UContainer class="max-w-4xl flex flex-col gap-4 mt-8">
    <UCard v-for="(country, i) in countries">
      <template #header> Pays n° {{ i + 1 }} </template>

      {{ country.name }}
    </UCard>
  </UContainer>
</template>
