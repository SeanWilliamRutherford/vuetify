<template>
  <v-autocomplete
    v-model="model"
    :items="variables"
    :custom-filter="customFilter"
    item-props
    auto-select-first
    base-color="disabled"
    chips
    clearable
    multiple
    persistent-clear
    placeholder="Search SASS API"
    prepend-inner-icon="mdi-database-search-outline"
    variant="outlined"
    return-object
  >
    <template #chip="{ props }">
      <v-chip
        v-bind="props"
        color="primary"
        label
        variant="flat"
      />
    </template>
  </v-autocomplete>

  <app-markup
    v-if="model.length > 0"
    :code="code"
    class="mb-6"
    language="scss"
    resource="src/styles/main.scss"
  />
</template>

<script setup>
  // Utilities
  import { computed, ref } from 'vue'

  const files = import.meta.glob('../../../../api-generator/dist/api/*.json')

  const variables = ref([])
  const model = ref([])

  const code = computed(() => {
    const $parsed = model.value.map(variable => {
      return `  ${variable.title}: ${variable.value}`
    }).join(',\n')

    return `@use 'vuetify' with (\n${$parsed},\n);`
  })

  async function getVariables (name) {
    return import(`../../../../api-generator/dist/api/${name}.json`)
  }

  function customFilter (value, query, item) {
    console.log(value)
    return (
      item.props.title.toLowerCase().indexOf(query.toLowerCase()) > -1 ||
      item.props.subtitle.toLowerCase().indexOf(query.toLowerCase()) > -1
    )
  }

  async function fetchApiData () {
    try {
      for (const file in files) {
        const name = file.split('/').pop().split('.')[0]

        if (!name.startsWith('V')) continue

        const component = await getVariables(name)

        for (const variable in component.sass) {
          variables.value.push({
            title: variable,
            value: component.sass[variable]?.default || null,
            subtitle: name,
          })
        }
      }
    } catch (err) {}
  }

  fetchApiData()
</script>
