<template>
    <el-table stripe  :data="state.list" border size="small" width="100%" :height="`${state.height}px`" >
        <el-table-column prop="Name" :label="$t('status.tunnelName')" width="120"></el-table-column>
        <el-table-column prop="Label" :label="$t('status.tunnelLabel')"></el-table-column>
        <el-table-column prop="ProtocolType" :label="$t('status.tunnelProtocol')" width="60"></el-table-column>
        <el-table-column prop="BufferSize" :label="$t('status.tunnelBuffer')" width="100">
            <template #default="scope">
                <el-select v-model="scope.row.BufferSize" placeholder="Select" size="small" @change="handleSave">
                    <el-option v-for="(item,index) in state.bufferSize" :key="index" :label="item" :value="index"/>
                </el-select>
            </template>
        </el-table-column>
        <el-table-column property="Reverse" :label="$t('status.tunnelReverse')" width="64">
            <template #default="scope">
                <el-switch :disabled="scope.row.DisableReverse" v-model="scope.row.Reverse" @change="handleSave" inline-prompt :active-text="$t('status.tunnelYes')" :inactive-text="$t('status.tunnelNo')" />
            </template>
        </el-table-column>
        <el-table-column property="SSL" :label="$t('status.tunnelSSL')" width="60">
            <template #default="scope">
                <el-switch :disabled="scope.row.DisableSSL" v-model="scope.row.SSL" @change="handleSave" inline-prompt :active-text="$t('status.tunnelYes')" :inactive-text="$t('status.tunnelNo')" />
            </template>
        </el-table-column>
        <el-table-column property="Disabled" :label="$t('status.tunnelDisanbled')" width="64">
            <template #default="scope">
                <el-switch v-model="scope.row.Disabled" @change="handleSave" inline-prompt :active-text="$t('status.tunnelYes')" :inactive-text="$t('status.tunnelNo')" style="--el-switch-on-color: red; --el-switch-off-color: #ddd" />
            </template>
        </el-table-column>
        <el-table-column prop="Order" :label="$t('status.tunnelSort')" width="104" fixed="right">
            <template #header>
                <div class="flex">
                    <strong>{{ $t('status.tunnelSort') }}</strong><span class="flex-1"></span><Sync name="TunnelTransports" v-if="state.isSelf"></Sync>
                </div>
            </template>
            <template #default="scope">
                <div>
                    <el-input-number v-model="scope.row.Order" :min="1" :max="255" @change="handleOrderChange" size="small" />
                </div>
            </template>
        </el-table-column>
    </el-table>
</template>
<script>
import { getTunnelTransports, setTunnelTransports } from '@/apis/tunnel';
import { injectGlobalData } from '@/provide';
import { ElMessage } from 'element-plus';
import { computed,onMounted,reactive, watch } from 'vue'
import { Delete,Plus,Top,Bottom } from '@element-plus/icons-vue';
import { useI18n } from 'vue-i18n';
import Sync from '../sync/Index.vue'
export default {
    props:['machineId','height'],
    components:{Delete,Plus,Top,Bottom,Sync},
    setup(props) {
        const {t} = useI18n();
        const globalData = injectGlobalData();
        const state = reactive({
            list:[],
            height:computed(()=>props.height),
            bufferSize:globalData.value.bufferSize,
            machineid:props.machineId || globalData.value.config.Client.Id,
            isSelf:computed(()=>{
                return state.machineid === globalData.value.config.Client.Id;
            })
        });

        const getData = ()=>{
            getTunnelTransports(state.machineid).then((res)=>{
                state.list = res.sort((a,b)=>a.Order - b.Order);
            });
        }

        const handleOrderChange = ()=>{
            handleSave(state.list);
        }    
        const handleSave = ()=>{
            state.list = state.list.slice().sort((a,b)=>a.Order - b.Order);
            setTunnelTransports({
                machineid:state.machineid,
                data:state.list
            }).then(()=>{
                ElMessage.success(t('common.oper'));
            }).catch((err)=>{
                console.log(err);
                ElMessage.error(t('common.operFail'));
            });
        }
        onMounted(()=>{
            getData();
        });

        return {state,handleOrderChange,handleSave}
    }
}
</script>
<style lang="stylus" scoped>
</style>