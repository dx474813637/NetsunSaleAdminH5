<template>
    <el-table 
        v-loading="loading" 
        :data="skuList" 
        style="width: 100%" 
        border 
        :maxHeight="maxHeight"
        :highlight-current-row="isRadioGroup"
        @current-change="handleCurrentTableChange"
        > 
        <el-table-column label="ID" :width="isRadioGroup ?70:50" >
            <template #default="{ row }">
                <div class="u-flex">
                    <span>{{ row.id }}</span> 
                    <span class="u-m-l-10" v-if="isRadioGroup && currentRow && currentRow.id == row.id ">
                        <el-Icon color="#ff0000">
                            <i-ep-CircleCheck></i-ep-CircleCheck>
                        </el-Icon>
                    </span>
                </div>
                
            </template>
        </el-table-column>
        <el-table-column prop="name" label="商品名" />
        <el-table-column prop="sku" label="商品规格" >
            <template #default="{ $index }">
                <el-tree :data="skuList[$index].sku" :props="defaultProps"  />
            </template>
        </el-table-column>
        <el-table-column prop="uptime" label="更新时间" width="200" />
        <el-table-column label="操作" width="90" align="center" v-if="isEditBtn"> 
            <template #default="{row}">
                <div class="u-flex u-flex-center">
                    <!-- <el-button 
                    link 
                    type="primary" 
                    size="small" 
                    @click="router.push({name: 'sku_edit', params: {id: row.id}})"
                    >编辑</el-button>   -->

                    <el-popconfirm 
                        title="移除确认" 
                        @confirm="deleteSku(row.id)"
                        confirm-button-text="确认"
                        cancel-button-text="取消"
                        >
                        <template #reference>
                            <el-button  
                                link 
                                type="danger" 
                                size="small"  
                                >删除</el-button> 
                        </template>
                    </el-popconfirm>


                
                </div>
                
            </template>
            
        </el-table-column>
        <template #empty>
            <div class="u-flex u-flex-center u-p-t-20 u-p-b-20">
                <el-empty description="无数据" />
            </div>
        </template>
    </el-table>
    <div class="list-page-box u-p-t-20 u-p-b-20">
        <el-pagination
            v-model:current-page="curP"
            :page-size="pageSize"
            small
            background
            layout="prev, pager, next, slot"
            :total="total"
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
        >
            <span class="u-p-l-10">共 {{ total }} 条数据</span>
        </el-pagination>
    </div> 
</template>

<script setup lang='ts'>
import { reactive,ref,computed, inject, onMounted } from 'vue'
import router from "@/router/guard" 
import { ElMessage } from "element-plus";
import { deepClone } from '@/utils';
import useProductSku from '@/hook/useProductSku'
const {
    sku2treeData
} = useProductSku()
const props = defineProps({
    isRadioGroup: {
        type: Boolean,
        default: false
    },
    isEditBtn: {
        type: Boolean,
        default: false
    },
    maxHeight: {
        type: [String, Number],
        default: '80vh'
    }
});
const emit = defineEmits(["setCurrentRow"]);
const currentRow = ref()
const $api = inject('$api')
const skuList = ref([])
const loading = ref(false)
const curP = ref(1)
const total = ref(0)
const pageSize = ref(20)
const paramsObj = computed(() => {
    return {
        p: curP.value
    }
})
const defaultProps = {
  children: 'children',
  label: 'label',
}
onMounted(async () => {
    refreshData()
})
async function refreshData() {
    
    loading.value = true; 
    await getData()
    loading.value = false;
}
const getData = async () => { 
    const res = await $api.login_sku({params: paramsObj.value, loading: false}) 
    skuList.value = res.list.map(ele => ({...ele, sku: sku2treeData(ele.sku)}))
    total.value = +res.total
}
 
const handleSizeChange = (val: number) => {
  console.log(`${val} items per page`)
}
const handleCurrentChange = async (val: number) => {
  console.log(`current page: ${val}`)
  await refreshData()
} 

const handleCurrentTableChange = (val: User | undefined) => { 
    if(!props.isRadioGroup) return 
    currentRow.value = val
    emit('setCurrentRow', {value: deepClone(currentRow.value)})
}
const deleteSku = async (id) => {
    const res = await $api.del_sku({params: {id}});
    if(res.code == 1){
        ElMessage.success(res.msg)
        refreshData()
    }
}

</script>
<style lang='scss' scoped>
// 
.el-tree {
    background-color: transparent;
}
</style>