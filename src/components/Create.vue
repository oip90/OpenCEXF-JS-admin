<template>
  <div v-if="type === 'bots_botconfig_list'" class="p-8" style="min-width: 50vw;">
    <BotConfig :data="props.data" @close="emit('close')"/>
  </div>
  <div v-else-if="type === 'auth_group_list'" class="p-8" style="min-width: 50vw;">
    <UserGroups />
  </div>
  <div v-else class="detail-data">
    <div v-for="field in fieldsIds" class="detail-data-item">
      <template v-if="props.data.fields[field].type === 'boolean'">
        <v-checkbox
          color="primary"
          :label="props.data.fields[field].attributes.label"
          v-model="values[field]"
        ></v-checkbox>
      </template>
      <template v-else-if="props.data.fields[field].attributes.label === 'User'">
        <v-autocomplete
          variant="underlined"
          color="primary"
          label="Users"
          clearable
          :items="users"
          item-title="text"
          item-value="value"
          v-model="values[field]"
          @update:search="fetchUsers"
        ></v-autocomplete>
      </template>
      <template v-else-if="props.data.fields[field].type === 'choice'">
        <v-select
          variant="underlined"
          color="primary"
          item-title="text"
          item-value="value"
          :items="props.data.fields[field].attributes.choices"
          :label="props.data.fields[field].attributes.label"
          v-model="values[field]"
        ></v-select>
      </template>
      <template v-else-if="props.data.fields[field].type === 'datetime'">
        <v-date-picker variant="underlined" color="primary" v-model="values[field]"></v-date-picker>
      </template>
      <template v-else>
        <v-text-field 
          variant="underlined"
          color="primary"
          :label="props.data.fields[field].attributes.label"
          v-model="values[field]"
          :hint="props.data.fields[field].attributes.hint"
        ></v-text-field>
      </template>
    </div>
    <div class="grid grid-cols-2 gap-5">
      <v-btn color="rgba(0, 0, 0, 0.52)" variant="outlined" @click="emit('close')">Close</v-btn>
      <v-btn color="primary" variant="flat" @click="save">Save</v-btn>
    </div>
  </div>
  <v-alert
    class="alert-block"
    v-if="alert"
    color="pink"
    dark
    border="top"
    icon="mdi-home"
    transition="scale-transition"
  >
  {{ alertText }}
</v-alert>
  </template>
  <script setup>
  import { onMounted, ref, defineEmits, computed } from 'vue';
  import { useRoute } from 'vue-router'
  import localConfig from "@/local_config"
  import axios from '../plugins/axios'
  import { splitAndReplace, endsWithList, removeListSuffix, findErrMessage } from "../plugins/helpers"
  import UserGroups from '@/components/UserGroups.vue'
  import BotConfig from '@/components/BotConfig.vue'
  const alert = ref(false)
  const alertText = ref('')
  const apiKey = localConfig.api
  const props = defineProps({
    data: {
      type: Object,
      required: true,
    },
    type: {
      type: String,
      required: false,
    },
  })

  const emit = defineEmits(['close'])
  
  const values = ref({})
  const route = useRoute()
  const param = ref(route.params.page)

  const users = ref([])

  const fetchUsers = async (searchValue) => {
    users.value = []
    try {
      const response = await axios.get(`${apiKey}auth/user/`,{ params: { search: searchValue } })
      response.data.results.map($ => {
        users.value.push({value: $.id, text: $.email})
      })
    } catch (error) {
      showAlert(error)    
    }
  }

  const fieldsIds = computed(() => 
    Object
      .keys(props.data.fields)
      .filter(field => 
        !(
          props.data.fields[field].attributes.read_only === true ||
          props.data.fields[field].attributes.label === 'Precisions' ||
          field === '_label'
        )
      )
  )

  onMounted(async () => {
    users.value = []
    await fetchUsers()
  });

  const save = async () => {
    let pathSepar = splitAndReplace(removeListSuffix(param.value))
    if(endsWithList(param.value)) 
      try {
        await axios.post(`${apiKey}${pathSepar[0]}/${pathSepar[1]}/`, values.value)
        location.reload()
      } catch (error) {
        showAlert(error)        
      }
  }
const showAlert = (err) => {
  const alertMessage = findErrMessage(err)
  if(alertMessage) {
    alert.value = true
    alertText.value = alertMessage
    setTimeout(() => {
      alert.value = false
      alertText.value = ''
    }, 13000)
  }
}
</script>
  
<style scoped>
.detail-data {
  width: 500px;
  padding: 30px;
  margin: 0 auto
}
.detail-data-item {
  margin-bottom: 20px;
}
.hidden {
  display: none;
}
.alert-block {
  position: fixed !important;
  bottom: 0 !important;
  right: 0 !important;
  width: 520px !important;
  z-index: 22 !important;
}
</style>