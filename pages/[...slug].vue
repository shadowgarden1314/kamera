<script setup lang="ts">
import photosList from '~/constants/photos.json'

const toast = useToast()
const user = useUserStore()
const route = useRoute()
const dataList = ref<Array<Object>>([])
const loading = ref<boolean>(false)
const handleButton = ref<boolean>(true)
const pageInfo = reactive({
  total: 0,
  totalPage: 0,
  pageNum: 1,
  pageSize: 10,
})

const dataHandle = async () => {
  loading.value = true
  try {
    const res = await $fetch('/api/getImageList', {
      method: 'post',
      headers: {
        Authorization: `${user.tokenName} ${user.token}`,
      },
      body: { pageNum: pageInfo.pageNum, pageSize: pageInfo.pageSize, type: route.path.replace('/', '') },
    })
    if (res?.code === 200) {
      if (pageInfo.pageNum <= res?.data.totalPage) {
        if (pageInfo.pageNum === res?.data.totalPage) {
          handleButton.value = false
        }
        pageInfo.pageNum++
        if (dataList.value.length === 0) {
          dataList.value = res?.data.data
        } else {
          dataList.value = dataList.value.concat(res?.data.data)
        }
        pageInfo.total = res?.data.total
        pageInfo.totalPage = res?.data.totalPage
      }
    }
  } catch (e) {
    toast.add({ title: '加载失败！', timeout: 2000, color: 'red' })
  } finally {
    loading.value = false
  }
}

onUnmounted(() => {
  dataList.value = []
  pageInfo.total = 0
  pageInfo.totalPage = 0
  pageInfo.pageNum = 1
})

definePageMeta({
  layout: 'default',
  validate: async (route) => {
    try {
      return !!photosList.some(i => i.url === route.path)
    } catch (e) {
      console.log(e)
      return false
    }
  },
})
</script>

<template>
  <Waterfall :dataList="dataList" :loading="loading" :handleButton="handleButton" @dataHandle="dataHandle" />
</template>
