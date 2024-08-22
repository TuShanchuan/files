<template>
    <TaskSetting class="task-setting !h-full" noresult>
        <template #task-content>
            <div class="item">
                <TaskTitle name="统计范围" />
                <el-date-picker
                    class="date-picker"
                    v-model="timeRange.start"
                    type="date"
                    placeholder="开始时间"
                    :default-value="new Date()"
                    :teleported="false"
                />
                <el-date-picker
                    class="date-picker"
                    v-model="timeRange.end"
                    type="date"
                    placeholder="结束时间"
                    :default-value="new Date()"
                    :teleported="false"
                />
            </div>
            <div class="item">
                <TaskTitle name="数据输入" />
                <el-input
                    class="input-align-right"
                    v-model="centerX"
                    placeholder="请输入"
                >
                    <template #prepend>圆心X：</template>
                </el-input>
                <el-input
                    class="input-align-right"
                    v-model="centerY"
                    placeholder="请输入"
                >
                    <template #prepend>圆心Y：</template>
                </el-input>
                <el-input
                    class="input-align-right"
                    v-model="innerRadius"
                    placeholder="请输入"
                >
                    <template #prepend>内半径：</template>
                </el-input>
                <el-input
                    class="input-align-right"
                    v-model="outerRadius"
                    placeholder="请输入"
                >
                    <template #prepend>外半径：</template>
                </el-input>
                <div class="mt-4 relative">
                    <el-select v-model="xinghao1">
                        <el-option
                            v-for="item in xinghaoOptions"
                            :key="item.value"
                            :label="item.label"
                            :value="item.value"
                        />
                    </el-select>
                    <div
                        class="
                            absolute
                            leading-9
                            pl-5
                            top-0
                            left-0
                            text-[#909399]
                        "
                    >
                        型号:
                    </div>
                </div>
                <div
                    class="
                        mt-4
                        cursor-pointer
                        user-select-none
                        w-full
                        h-9
                        leading-9
                        text-center text-[#121A2F]
                        bg-[#73EFFF]
                        rounded
                    "
                    @click="updateChart"
                >
                    更新数据
                </div>
            </div>
            <div class="item">
                <TaskTitle name="查询输入" />
                <el-input
                    class="input-align-right"
                    v-model="bianhao"
                    placeholder="请输入"
                >
                    <template #prepend>编号：</template>
                </el-input>
                <el-input
                    class="input-align-right"
                    v-model="xinghao2"
                    placeholder="请输入"
                >
                    <template #prepend>型号：</template>
                </el-input>
                <div class="mt-4 relative">
                    <el-select v-model="position">
                        <el-option
                            v-for="item in positionOptions"
                            :key="item.value"
                            :label="item.label"
                            :value="item.value"
                        />
                    </el-select>
                    <div
                        class="
                            absolute
                            leading-9
                            pl-5
                            top-0
                            left-0
                            text-[#909399]
                        "
                    >
                        位置:
                    </div>
                </div>
                <div
                    class="
                        mt-4
                        cursor-pointer
                        user-select-none
                        w-full
                        h-9
                        leading-9
                        text-center text-[#121A2F]
                        bg-[#73EFFF]
                        rounded
                    "
                    @click="updateTable"
                >
                    查询
                </div>
            </div>
        </template>
        <template #result-content>
            <slot name="result-content"></slot>
        </template>
    </TaskSetting>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import TaskSetting from '@components/task-setting-template/index.vue';
import TaskTitle from '@components/task-setting-template/task-title.vue';

const emit = defineEmits(['updateChart', 'updateTable']);

defineProps({
    xinghaoOptions: Array,
});

const timeRange = defineModel('timeRange');
const positionOptions = defineModel('positionOptions');

const centerX = ref(128);
const centerY = ref(128);
const innerRadius = ref(5);
const outerRadius = ref(10);
const xinghao1 = ref();
const bianhao = ref();
const xinghao2 = ref();
const position = ref();

const randomizeCircleCenter = () => {
    const randomIntX = Math.floor(Math.random() * 3);
    const randomIntY = Math.floor(Math.random() * 3);
    centerX.value += randomIntX;
    centerX.value += randomIntY;
};

const updateChart = () => {
    emit('updateChart', {
        selectedXinghao: xinghao1.value,
        innerRadius: innerRadius.value,
        outerRadius: outerRadius.value,
        centerX: centerX.value,
        centerY: centerY.value,
    });
};

const updateTable = () => {
    emit('updateTable', {
        bianhao: bianhao.value,
        xinghao: xinghao2.value,
        position: position.value,
    });
};

onMounted(() => {
    randomizeCircleCenter();
});
</script>

<style scoped lang="less">
.item {
    display: flex;
    flex-direction: column;
    margin-top: 20px;

    :deep(.date-picker) {
        --el-date-editor-width: 100%;
    }
}

.task-setting {
    max-width: 300px; /* 根据需要调整最大宽度 */
    margin-left: auto; /* 确保其位于右侧 */
}

:deep(.el-input) {
    .el-input-group__prepend {
        padding-right: 0;
    }

    .el-input__inner {
        color: #fff;

        &::placeholder {
            color: #666;
        }
    }
}
</style>
