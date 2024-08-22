<template>
    <div class="model-inference pl-[75px] pr-[28px] relative flex items-center">
        <Thumbnail
            ref="thumbnailRef"
            v-model:imgSrcList="imageList"
            v-model:currentImgIndex="currentIndex"
        />
        <div class="flex flex-1 pt-10 h-full pl-5">
            <div class="leftAdjust flex-1 pb-12">
                <input
                    type="file"
                    ref="fileInput1"
                    @change="handleFileChange1"
                    accept="image/*"
                    style="display: none"
                />
                <input
                    type="file"
                    ref="folderInput1"
                    @change="handleFolderChange"
                    webkitdirectory
                    multiple
                    style="display: none"
                />
                <canvas
                    id="labelBox2"
                    name="labelBox2"
                    ref="container2"
                    class="container2"
                ></canvas>
                <el-text type="primary" style="left: 33px">输入</el-text>
                <!-- 新增调节栏 -->
                <div class="adjustment-controls2 absolute bottom-0">
                    <div class="flex items-center">
                        <label class="whitespace-nowrap leading-[40px]"
                            >对比度：</label
                        >
                        <input
                            type="range"
                            min="0"
                            max="200"
                            v-model="contrast2"
                            @input="updateImage2"
                        />
                        <span class="value2">{{ contrast2 - 100 }}</span>
                    </div>
                    <div class="flex items-center">
                        <label class="whitespace-nowrap leading-[40px]"
                            >亮度：</label
                        >
                        <input
                            type="range"
                            min="0"
                            max="200"
                            v-model="brightness2"
                            @input="updateImage2"
                        />
                        <span class="value2">{{ brightness2 - 100 }}</span>
                    </div>
                    <div class="flex items-center">
                        <label class="whitespace-nowrap leading-[40px]"
                            >色差：</label
                        >
                        <input
                            type="range"
                            min="0"
                            max="200"
                            v-model="saturation2"
                            @input="updateImage2"
                        />
                        <span class="value2">{{ saturation2 - 100 }}</span>
                    </div>
                    <!-- 新增重置按钮 -->
                    <button
                        class="whitespace-nowrap h-7"
                        @click="resetAdjustments2"
                    >
                        重置
                    </button>
                </div>
            </div>
            <div class="right flex-1 relative pb-12">
                <canvas
                    id="labelBox"
                    name="labelBox"
                    ref="container"
                    class="container"
                ></canvas>
                <el-text type="primary" style="left: 33px">输出</el-text>
                <!-- 显示处理状态 -->
                <div
                    class="
                        label-selector
                        absolute
                        -top-7
                        left-1/2
                        flex
                        -translate-x-1/2
                    "
                >
                    <select v-model="selectedLabel">
                        <option
                            v-for="label in labels"
                            :key="label"
                            :value="label"
                        >
                            {{ label }}
                        </option>
                    </select>
                    <input
                        type="text"
                        v-model="newLabel"
                        placeholder="添加标签"
                        class="new-label-input"
                    />
                    <button @click="addNewLabel" class="add-label-button">
                        添加
                    </button>
                    <button
                        v-if="deletable"
                        @click="removeLabel(label)"
                        class="remove-label-btn"
                    >
                        删除
                    </button>
                </div>
                <!-- 新增调节栏 -->
                <div class="adjustment-controls absolute bottom-0">
                    <div class="flex items-center">
                        <label class="whitespace-nowrap leading-[40px]"
                            >对比度：</label
                        >
                        <input
                            type="range"
                            min="0"
                            max="200"
                            v-model="contrast"
                            @input="updateImage"
                        />
                        <span class="value">{{ contrast - 100 }}</span>
                    </div>
                    <div class="flex items-center">
                        <label class="whitespace-nowrap leading-[40px]"
                            >亮度：</label
                        >
                        <input
                            type="range"
                            min="0"
                            max="200"
                            v-model="brightness"
                            @input="updateImage"
                        />
                        <span class="value">{{ brightness - 100 }}</span>
                    </div>
                    <div class="flex items-center">
                        <label class="whitespace-nowrap leading-[40px]"
                            >色差：</label
                        >
                        <input
                            type="range"
                            min="0"
                            max="200"
                            v-model="saturation"
                            @input="updateImage"
                        />
                        <span class="value">{{ saturation - 100 }}</span>
                    </div>
                    <!-- 新增重置按钮 -->
                    <button
                        class="whitespace-nowrap h-7"
                        @click="resetAdjustments"
                    >
                        重置
                    </button>
                </div>
            </div>
        </div>
        <OperationBtns class="operation" :operationList="operationList" />
        <TaskSetting
            class="task-setting"
            :annotation_list="annotation_list"
            :confidenceSelect="confidenceSelect"
            :scale="scale"
            :modelSelect="modelSelect"
            @update:confidenceSelect="handleConfidenceChange"
            @update:scale="handleScaleChange"
            @update:modelSelect="handleModelSelect"
            @delete-annotation="deleteAnnotation"
        ></TaskSetting>
        <div v-if="isProcessing" class="processing-status">
            <el-progress
                class="w-[370px]"
                :percentage="progress"
                :stroke-width="15"
                status="success"
                striped
                striped-flow
                :duration="50"
                :show-text="false"
                color="#73efff"
            />
            <div class="text-center text-white mt-5 text-[24px]">
                正在处理中...
            </div>
        </div>
        <button @click="goBack" class="back-button">返回主界面</button>
    </div>
</template>
<script setup>
import { useRouter } from 'vue-router';
import { ref, onMounted, watch } from 'vue';
import axios from 'axios';
import OperationBtns from '@components/operation-btns.vue';
import TaskSetting from './components/task-setting.vue';
import fileIcon from '@assets/images/common/operation-icon/选择文件夹.png';
import fileActiveIcon from '@assets/images/common/operation-icon/选择文件夹-active.png';
import prevIcon from '@assets/images/common/operation-icon/prev.png';
import prevActiveIcon from '@assets/images/common/operation-icon/prev-active.png';
import nextIcon from '@assets/images/common/operation-icon/next.png';
import nextActiveIcon from '@assets/images/common/operation-icon/next-active.png';
import referenceIcon from '@assets/images/common/operation-icon/推理.png';
import referenceActiveIcon from '@assets/images/common/operation-icon/推理-active.png';
import saveIcon from '@assets/images/common/operation-icon/save.png';
import saveActiveIcon from '@assets/images/common/operation-icon/save-active.png';
import outlineIcon from '@assets/images/common/operation-icon/画框.png';
import outlineActiveIcon from '@assets/images/common/operation-icon/画框-active.png';
import deleteIcon from '@assets/images/common/operation-icon/删除.png';
import deleteActiveIcon from '@assets/images/common/operation-icon/删除.png';
import img from '@assets/images/home/bg2.jpg';
import img2 from '@assets/images/home/bg2.jpg';
import CanvasSelect from 'canvas-select';
import Thumbnail from '@components/thumbnail.vue';

const thumbnailRef = ref(null);

const router = useRouter(); // 定义路由

let processTimer;
const confidenceSelect = ref(0.5); // 默认值
const scale = ref(1); // 默认值
const modelSelect = ref({}); // 默认空对象

const imgSrc1 = ref(img); // 初始值设置为 null，因为初始时没有选择图片
const imgSrc2 = ref(img2); // 假设输出图像的初始值仍然为预设的图

const fileInput1 = ref(null);
const folderInput1 = ref(null);

// 调节栏的值
const contrast = ref(100);
const brightness = ref(100);
const saturation = ref(100);

const contrast2 = ref(100);
const brightness2 = ref(100);
const saturation2 = ref(100);

const cls = ref(null); // 初始化 cls
const positions = ref(null); // 初始化 positions

const imageList = ref([]); // 存储所有图片路径
const processedImageList = ref([]); // 存储所有处理后的图片路径

const currentIndex = ref(0); // 当前显示的图片索引

const clsList = ref(null);
const positionsList = ref(null);

const fileInputRef = ref();

const isProcessing = ref(false); // 用于存储处理状态

const annotationList = ref([]); // 记录单张图的标注结果
const annotation_list = ref([]); // 取出单张图的结果
const annotationListAll = ref([]); // 记录所有图的标注结果
const file_name_lists = ref([]); // 记录所有图的文件名
const options = ref([]); // 保存canvas需要的标注框信息
const imageWidth = ref(1000); // 图像宽度
const imageHeight = ref(1080); // 图像高度
const selectedLabel = ref('气孔');
const labels = ref(['气孔', '夹杂', '裂纹']); // 初始标签
const newLabel = ref('');
const deletable = ref(true); // 控制是否显示删除按钮
let instance = null;
let instance2 = null;
const single_file_name = ref(''); // 单个图像的名字

watch(currentIndex, () => {
    imgSrc1.value = imageList.value[currentIndex.value];
    imgSrc2.value = imageList.value[currentIndex.value];
    annotationList.value = annotationListAll.value[currentIndex.value];
    annotation_list.value = annotationList.value.annotation_list;
});

const handleConfidenceChange = (value) => {
    confidenceSelect.value = value;
};

const handleScaleChange = (value) => {
    scale.value = value;
};

const handleModelSelect = (file) => {
    modelSelect.value = file;
};

function handleSelectFile() {
    fileInputRef.value.click();
}

function handleFileChange1(event) {
    const file = event.target.files[0];
    console.log('file:', file);
    single_file_name.value = file.name;
    if (file) {
        const reader = new FileReader();
        reader.onload = (e) => {
            imgSrc1.value = e.target.result;
            const image = new Image();
            image.onload = () => {
                imageWidth.value = reader.width;
                imageHeight.value = reader.height;
            };
            image.src = e.target.result;
        };
        reader.readAsDataURL(file);
    }
}

async function handleFolderChange(event) {
    // 先清空imageList等的值
    imageList.value = [];
    imgSrc2.value = img2;
    annotationListAll.value = [];
    positionsList.value = [];
    clsList.value = [];
    const files = Array.from(event.target.files).filter((file) =>
        file.type.startsWith('image/')
    ); // 只保留图像文件

    if (!modelSelect.value || !modelSelect.value.name) {
        ElMessage({
            message: '请先选择检测模型！',
            type: 'error',
        });
        imgSrc2.value = img2;
        // return;
    }

    if (!files.length) {
        ElMessage({
            message: '未获取到文件夹数据',
            type: 'error',
        });
        return;
    }

    const formData = new FormData();
    files.forEach((file) => {
        formData.append('images', file);
    });
    formData.append('model', modelSelect.value);
    formData.append('iou', confidenceSelect.value);
    formData.append('scale', scale.value);

    // 提取文件夹名称
    const filePath = files[0].webkitRelativePath || files[0].name; // 获取第一个文件的相对路径或名称
    const folderName = filePath.split('/')[0]; // 获取文件夹名称
    console.log('Folder Name:', folderName);

    formData.append('folder_name', folderName);

    try {
        isProcessing.value = true; // 开始处理
        processTimer && clearInterval(processTimer);
        progress.value = 0;
        processTimer = setInterval(() => {
            if (progress.value < 80) {
                progress.value++;
            } else {
                processTimer && clearInterval(processTimer);
            }
        }, 1000);
        const response = await axios.post(
            'http://127.0.0.1:8000/process_folder',
            formData,
            {
                headers: {
                    'Content-Type': 'multipart/form-data',
                },
            }
        );

        imageList.value = files.map((file) => URL.createObjectURL(file)); // 存储所有图片路径
        currentIndex.value = 0; // 重置当前索引
        imgSrc1.value = imageList.value[currentIndex.value]; // 显示第一张图片
        imgSrc2.value = imageList.value[currentIndex.value]; // 显示第一张图片

        // 接收所有标注数据
        annotationListAll.value = response.data.annotationRequestList;
        annotationList.value = annotationListAll.value[currentIndex.value];
        annotation_list.value = annotationList.value.annotation_list;

        //接收其他数据
        positionsList.value = response.data.positionsAll;
        clsList.value = response.data.clsAll;

        console.log(imageList.value);
        console.log(annotationListAll.value);

        // 处理后端返回的所有图像URL
        const processedImageUrls = response.data.processedImageUrls;
        processedImageList.value = processedImageUrls.map(
            (item) => item.output_image
        );
    } catch (error) {
        console.error('Error processing the images:', error);
    } finally {
        isProcessing.value = false; // 处理完成
        processTimer && clearInterval(processTimer);
    }
}

// 归一化
const normalized = (coor, imageWidth, imageHeight) => {
    const centerX = coor[0] / imageWidth;
    const centerY = coor[1] / imageHeight;
    const width = coor[2] / imageWidth;
    const height = coor[3] / imageHeight;
    return [centerX, centerY, width, height];
};

const convertToYOLONoNormalized = (coor, imageWidth, imageHeight) => {
    const [x_min, y_min] = coor[0];
    const [x_max, y_max] = coor[1];
    const centerX = (x_min + x_max) / 2;
    const centerY = (y_min + y_max) / 2;
    const width = x_max - x_min;
    const height = y_max - y_min;
    return [centerX, centerY, width, height];
};

// 监听imgSrc的变化，更新canvas
watch(imgSrc2, () => {
    updateCanvasRight();
});

// 监听imgSrc的变化，更新canvas
watch(imgSrc1, () => {
    updateCanvasLeft();
});

const updateCanvasRight = () => {
    if (instance) {
        // 获取读到图片的长和宽
        const img = new Image();
        img.src = imgSrc2.value;
        img.onload = () => {
            imageWidth.value = img.width;
            imageHeight.value = img.height;
            console.log('Image Width:', imageWidth.value);
            console.log('Image Height:', imageHeight.value);
            const annotationForActual = annotation_list.value.map(
                (annotation) => {
                    return {
                        label: annotation.label,
                        coor: normalized(
                            annotation.coor,
                            imageWidth.value,
                            imageHeight.value
                        ),
                    };
                }
            );
            options.value = convertAnnotationsToOptions(annotationForActual); // 这里用的是左上和右下点的坐标
            instance.setData(options.value);
            instance.setImage(imgSrc2.value);
            instance.resize();
            instance.fitZoom();
            instance.update();
            console.log('instance.dataset: ', instance.dataset);
            console.log('options: ', options.value);
        };
    }
};

const updateCanvasLeft = () => {
    if (instance2) {
        // 获取读到图片的长和宽
        const img = new Image();
        img.src = imgSrc1.value;
        img.onload = () => {
            imageWidth.value = img.width;
            imageHeight.value = img.height;
            console.log('Image Width:', imageWidth.value);
            console.log('Image Height:', imageHeight.value);
            instance2.setImage(imgSrc1.value);
            instance2.resize();
            instance2.fitZoom();
            instance2.update();
            console.log('instance.dataset: ', instance.dataset);
            console.log('options: ', options.value);
        };
    }
};

// 函数用于更新后端的label.txt文件
async function updateLabelFile() {
    try {
        const response = await fetch('http://localhost:8000/update_labels/', {
            // 假设后端运行在8000端口
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify({ labels: labels.value }), // 发送当前标签列表
        });

        if (!response.ok) {
            throw new Error(`Error: ${response.statusText}`);
        }

        const data = await response.json();
        console.log(data); // 打印后端响应
    } catch (error) {
        console.error('Error updating label file:', error);
    }
}

// 添加新标签
function addNewLabel() {
    if (newLabel.value && !labels.value.includes(newLabel.value)) {
        labels.value.push(newLabel.value); // 添加新标签到列表
        selectedLabel.value = newLabel.value; // 更新选中的标签为新标签
        newLabel.value = ''; // 清空输入框
        updateLabelFile(); // 更新文件
    }
}

// 删除标签的方法
function removeLabel(labelToRemove) {
    if (newLabel.value && labels.value.includes(newLabel.value)) {
        labels.value = labels.value.filter((label) => label !== newLabel.value);
        newLabel.value = ''; // 清空输入框
        selectedLabel.value = labels.value.length > 0 ? labels.value[0] : ''; // 更新选中的标签为第一个标签
        updateLabelFile(); // 更新文件
    } else {
        console.error('标签不需要删除', labelToRemove);
    }
}

// 将annotationList转成标签需要的格式
function convertAnnotationsToOptions(annotationList) {
    const options = annotationList.map((annotation) => {
        return {
            label: annotation.label,
            labelFillStyle: '#fff', // 默认填充样式，你可以根据实际情况调整
            textFillStyle: '#000', // 默认文本填充样式，你可以根据实际情况调整
            coor: convertYoloToActualCoordinates(
                annotation.coor,
                imageWidth.value,
                imageHeight.value
            ),
            // 根据具体情况，你可能需要设置其他属性，例如 type
            // 这里假设所有标注项都是相同类型
            type: 1, // 假设类型为1，你可以根据实际情况调整
        };
    });
    return options;
}

// YOLO坐标转换为屏幕坐标
function convertYoloToActualCoordinates(yoloCoor, imgWidth, imgHeight) {
    const [x_center, y_center, width, height] = yoloCoor;
    const x1 = (x_center - width / 2) * imgWidth;
    const y1 = (y_center - height / 2) * imgHeight;
    const x2 = (x_center + width / 2) * imgWidth;
    const y2 = (y_center + height / 2) * imgHeight;
    return [
        [x1, y1],
        [x2, y2],
    ];
}

async function handleAnalyze() {
    // 清空imageList
    imageList.value = [];
    // 清空缓存数据
    processedImageList.value = [];
    imgSrc2.value = img2;
    imageList.value = [];
    annotationListAll.value = [];
    positionsList.value = [];
    clsList.value = [];

    if (!imgSrc1.value) {
        alert('请先选择一个输入图像!');
        return;
    }

    if (!modelSelect.value || !modelSelect.value.name) {
        console.log(modelSelect.value);
        console.log(modelSelect.value.name);
        alert('请先选择检测模型！');
        return;
    }
    console.log(modelSelect.value);
    console.log(modelSelect.value.name);

    const formData = new FormData();
    formData.append('image', imgSrc1.value);
    formData.append('model', modelSelect.value);
    formData.append('iou', confidenceSelect.value);
    formData.append('scale', scale.value);
    formData.append('file_name', single_file_name.value);

    try {
        isProcessing.value = true; // 开始处理
        processTimer && clearInterval(processTimer);
        progress.value = 0;
        processTimer = setInterval(() => {
            if (progress.value < 80) {
                progress.value++;
            } else {
                processTimer && clearInterval(processTimer);
            }
        }, 1000);
        const response = await axios.post(
            'http://127.0.0.1:8000/process',
            formData,
            {
                headers: {
                    'Content-Type': 'multipart/form-data',
                },
            }
        );

        // 假设后端返回处理后的图像URL
        // 0613改 imgSrc2.value = img2
        imgSrc2.value = imgSrc1.value;
        // imgSrc2.value = response.data.processedImageUrl;
        cls.value = response.data.cls;
        positions.value = response.data.positions;

        //接受标注数据
        annotationListAll.value = response.data.annotationRequestList;
        annotationList.value = annotationListAll.value[currentIndex.value];
        annotation_list.value = annotationList.value.annotation_list;

        console.log(cls.value);
        console.log(positions.value);
    } catch (error) {
        console.error('Error processing the image:', error);
    } finally {
        isProcessing.value = false; // 处理完成
        processTimer && clearInterval(processTimer);
    }
}

function handlePrevImage() {
    if (currentIndex.value > 0) {
        currentIndex.value -= 1;
        imgSrc1.value = imageList.value[currentIndex.value];
        if (annotationListAll.value) {
            annotationList.value = annotationListAll.value[currentIndex.value];
            annotation_list.value = annotationList.value.annotation_list;
        }
        imgSrc2.value = imageList.value[currentIndex.value];
        resetAdjustments();
        resetAdjustments2();
        thumbnailRef.value.slidePrev?.();
    } else {
        alert('已经是第一张图片');
    }
}

function handleNextImage() {
    if (currentIndex.value < imageList.value.length - 1) {
        currentIndex.value += 1;
        imgSrc1.value = imageList.value[currentIndex.value];
        if (annotationListAll.value) {
            annotationList.value = annotationListAll.value[currentIndex.value];
            annotation_list.value = annotationList.value.annotation_list;
        }
        imgSrc2.value = imageList.value[currentIndex.value];
        resetAdjustments();
        resetAdjustments2();
        thumbnailRef.value.slideNext?.();
    } else {
        alert('已经是最后一张图片');
    }
}

// 保存数据
async function handleSave() {
    // 临时变量，保存YOLO格式的标注数据，传给后端
    let annotationAll_trans = annotationListAll.value;
    let annotation_sql = annotationAll_trans;
    for (let i = 0; i < annotationAll_trans.length; i++) {
        const formData = new FormData();
        let annotation_single = annotationAll_trans[i];
        // let file_name = file_name_lists.value[i];
        formData.append('annotation', JSON.stringify(annotation_single));
        //formData.append('file_name', file_name);
        try {
            const response = await axios.post(
                'http://127.0.0.1:8000/defect_save_mysql',
                formData,
                {
                    headers: {
                        'Content-Type': 'multipart/form-data',
                    },
                }
            );
            console.log(response.data.message);
        } catch (error) {
            console.error('结果保存失败:', error);
        }
    }
    if (Object.keys(imageList.value).length === 0) {
        // 单张图片情况
        const formData = new FormData();
        // annotationListAll.value[currentIndex.value].annotation_list = annotation_list;
        // 使用 JSON 序列化和反序列化进行深拷贝
        console.log('annotationAll_trans:', annotationListAll.value);
        annotationAll_trans = JSON.parse(
            JSON.stringify(annotationListAll.value)
        );
        annotationAll_trans.forEach((item) => {
            item.annotation_list = item.annotation_list.map((annotation) => {
                return {
                    label: annotation.label,
                    coor: normalized(
                        [
                            annotation.coor[0],
                            annotation.coor[1],
                            annotation.coor[2],
                            annotation.coor[3],
                        ],
                        imageWidth.value,
                        imageHeight.value
                    ),
                };
            });
        });
        formData.append('annoList', JSON.stringify(annotationAll_trans));
        formData.append('image', imgSrc1.value);
        try {
            const response = await axios.post(
                'http://127.0.0.1:8000/defect_save_singleimg',
                formData,
                {
                    headers: {
                        'Content-Type': 'multipart/form-data',
                    },
                }
            );
            console.log(response.data.message);
            alert(`结果保存成功，存储路径为${response.data.savePath}`);
        } catch (error) {
            console.error('结果保存失败:', error);
        } finally {
            isProcessing.value = false; // 处理完成
        }
        console.log('No images to save');
    } else {
        const formData = new FormData();
        // 使用 JSON 序列化和反序列化进行深拷贝
        annotationAll_trans = JSON.parse(
            JSON.stringify(annotationListAll.value)
        );
        console.log('annotationAll_trans:', annotationListAll.value);
        annotationAll_trans.forEach((item) => {
            item.annotation_list = item.annotation_list.map((annotation) => {
                return {
                    label: annotation.label,
                    coor: normalized(
                        [
                            annotation.coor[0],
                            annotation.coor[1],
                            annotation.coor[2],
                            annotation.coor[3],
                        ],
                        imageWidth.value,
                        imageHeight.value
                    ),
                };
            });
        });
        formData.append('annoList', JSON.stringify(annotationAll_trans));

        for (let key in imageList.value) {
            const blobUrl = imageList.value[key];
            const blob = await fetch(blobUrl).then((r) => r.blob());
            formData.append('imgs', new File([blob], `image_${key}.jpg`));
        }

        try {
            const response = await axios.post(
                'http://127.0.0.1:8000/defect_save',
                formData,
                {
                    headers: {
                        'Content-Type': 'multipart/form-data',
                    },
                }
            );
            console.log(response.data.message);
            alert(`结果保存成功，存储路径为${response.data.savePath}`);
        } catch (error) {
            console.error('结果保存失败:', error);
        } finally {
            isProcessing.value = false; // 处理完成
        }
    }

    // 将结果保存到数据库
    // 使用 JSON 序列化和反序列化进行深拷贝
    console.log('annotationAll_trans:', annotationListAll.value);
    annotationAll_trans = JSON.parse(JSON.stringify(annotationListAll.value));
    annotationAll_trans.forEach((item) => {
        item.annotation_list = item.annotation_list.map((annotation) => {
            return {
                label: annotation.label,
                coor: normalized(
                    [
                        annotation.coor[0],
                        annotation.coor[1],
                        annotation.coor[2],
                        annotation.coor[3],
                    ],
                    imageWidth.value,
                    imageHeight.value
                ),
            };
        });
    });
}

// 观察选择标签的变换, 选择标签后，将标签赋值给当前选中的标注框
watch(selectedLabel, (newLabel) => {
    if (instance) {
        console.log(`Selected label changed to: ${newLabel}`);
    }
});

const deleteAnnotation = (index) => {
    if (instance) {
        instance.deleteByIndex(index);
        // 修改labelDict中的标注
        const file_name = imageList.value[index];
        labelDict.value[file_name] = annotationList.value;
        annotation_list.value.splice(index, 1);

        annotationListAll.value[currentIndex.value].annotation_list =
            annotation_list;
    }
};

onMounted(() => {
    instance = new CanvasSelect('.container', imgSrc2.value);
    instance2 = new CanvasSelect('.container2', imgSrc1.value);
    instance2.isCursor = false;
    instance2.createType = 0;
    instance.labelUp = true;
    instance.setData(options.value);
    instance.isCursor = false;
    instance.on('add', (info) => {
        info.labelFillStyle = '#fff';
        info.labelFont = '8px sans-serif';
        info.label = selectedLabel.value;
        instance.update();
    });

    instance.on('updated', (info) => {
        const newAnnotations = info.map((item) => {
            const yoloCoor = convertToYOLONoNormalized(
                item.coor,
                imageWidth.value,
                imageHeight.value
            ); // 转换成没有归一化的yolo坐标
            return { label: item.label, coor: yoloCoor };
        });
        //更新
        annotationListAll.value[currentIndex.value].annotation_list =
            newAnnotations;
        annotation_list.value = newAnnotations;
    });

    const canvasElement = document.querySelector('.container');
    // canvasElement.addEventListener('click', handleMouseClick);
});

const handleMouseClick = (event) => {
    if (instance) {
        const rect = event.target.getBoundingClientRect();
        const x = event.clientX - rect.left;
        const y = event.clientY - rect.top;
        const clickedAnnotation = annotation_list.value.find((item) => {
            const [centerX, centerY, width, height] = item.coor;
            const x_min = (centerX - width / 2) * imageWidth.value;
            const y_min = (centerY - height / 2) * imageHeight.value;
            const x_max = (centerX + width / 2) * imageWidth.value;
            const y_max = (centerY + height / 2) * imageHeight.value;
            return x >= x_min && x <= x_max && y >= y_min && y <= y_max;
        });

        if (clickedAnnotation) {
            instance.createType = 0;
        } else {
            instance.createType = 1;
        }
    }
};

// 调整图像
const updateImage = () => {
    const img = document.querySelector('.container');
    if (img) {
        img.style.filter = `
            contrast(${contrast.value}%)
            brightness(${brightness.value}%)
            saturate(${saturation.value}%)
        `;
    }
};

const updateImage2 = () => {
    const img = document.querySelector('.container2');
    if (img) {
        img.style.filter = `
            contrast(${contrast2.value}%)
            brightness(${brightness2.value}%)
            saturate(${saturation2.value}%)
        `;
    }
};

//重置图像参数
const resetAdjustments = () => {
    contrast.value = 100;
    brightness.value = 100;
    saturation.value = 100;
    updateImage();
};

const resetAdjustments2 = () => {
    contrast2.value = 100;
    brightness2.value = 100;
    saturation2.value = 100;
    updateImage2();
};

watch([contrast, brightness, saturation], () => {
    updateImage();
});

watch([contrast2, brightness2, saturation2], () => {
    updateImage2();
});

function isLabel() {
    if (instance.createType == 0) {
        instance.createType = 1;
    } else {
        instance.createType = 0;
    }
}

const operationList = [
    {
        icon: fileIcon,
        activeIcon: fileActiveIcon,
        name: '选择文件',
        callback() {
            const choice = confirm(
                '请选择导入方式：\n确定：单张图片\n取消：文件夹'
            );
            if (choice) {
                fileInput1.value.click(); // 触发文件选择对话框
            } else {
                folderInput1.value.click(); // 触发文件夹选择对话框
            }
        },
    },
    {
        icon: prevIcon,
        activeIcon: prevActiveIcon,
        name: '上一张',
        callback: handlePrevImage,
    },
    {
        icon: nextIcon,
        activeIcon: nextActiveIcon,
        name: '下一张',
        callback: handleNextImage,
    },
    {
        icon: outlineIcon,
        activeIcon: outlineActiveIcon,
        name: '标注',
        callback: isLabel,
    },
    {
        icon: referenceIcon,
        activeIcon: referenceActiveIcon,
        name: '开始推理',
        callback: handleAnalyze,
    },
    {
        icon: saveIcon,
        activeIcon: saveActiveIcon,
        name: '保存结果',
        callback: handleSave,
    },
];
const goBack = () => {
    router.push('/home'); // 跳转到主界面，假设主界面的路径是'/'
};
</script>
<style scoped lang="less">
.model-inference {
    position: relative;
    width: 100%;
    height: 100%;
    display: flex;
    padding-left: 75px;
    padding-right: 28px;

    .leftAdjust,
    .right {
        height: 100%;
        min-width: 0;
        flex: 1;
        position: relative;
        // flex: 1;
        // display: flex;
        // flex-direction: column;
        // align-items: center;

        canvas,
        canvas {
            user-select: none;
            -webkit-user-drag: none;
            height: 100%;
        }

        .el-text {
            position: absolute;
            left: 5px;
            top: 5px;
            cursor: default;
        }

        .label-selector select {
            width: 80px;
            height: 30px;
            padding: 5px;
            font-size: 14px;
        }
    }

    .leftAdjust canvas,
    .right canvas {
        width: 100%;
        height: 100%;
    }

    .leftAdjust {
        margin-right: 15px; /* 确保左右两部分没有额外的边距影响 */
    }

    .operation {
        position: absolute;
        left: 0;
        top: 50%;
        transform: translateY(-50%);
    }

    .task-setting {
        position: absolute;
        right: 0;
        top: 50%;
        transform: translateY(-50%);
    }
    .processing-status {
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
    }

    .new-label-input {
        width: 150px;
        height: 30px;
        padding: 5px;
        font-size: 14px;
    }

    .add-label-button {
        width: 50px;
        height: 30px;
        padding: 5px;
        font-size: 14px;
        color: #ffffff;
        background-color: #0000ff;
    }

    .remove-label-btn {
        width: 50px;
        height: 30px;
        padding: 5px;
        font-size: 14px;
        color: #ffffff;
        background-color: #0000ff;
    }
    .adjustment-controls {
        display: flex;
        margin-top: 0px;
        margin-left: 5%;
        z-index: 1000;
    }

    .adjustment-controls2 {
        display: flex;
        margin-top: 0px;
        margin-left: 5%;
        z-index: 1000;
    }

    .adjustment-controls .value {
        margin-left: 5px;
        margin-top: 5px;
        font-weight: bold;
        color: #dacbcbe8;
        width: 20px;
        z-index: 1000;
    }

    .adjustment-controls2 .value2 {
        margin-left: 5px;
        margin-top: 5px;
        font-weight: bold;
        color: #dacbcbe8;
        width: 20px;
        z-index: 1000;
    }

    .adjustment-controls label {
        color: white; /* 将字体颜色设置为白色 */
        margin-left: 20px;
        z-index: 1000;
    }

    .adjustment-controls2 label {
        color: white; /* 将字体颜色设置为白色 */
        margin-left: 20px;
        z-index: 1000;
    }

    .adjustment-controls input {
        z-index: 1000;
        padding: 5px;
        margin-left: 5px;
    }

    .adjustment-controls2 input {
        z-index: 1000;
        padding: 5px;
        margin-left: 5px;
    }

    .adjustment-controls button {
        margin-top: 6px;
        padding: 3px;
        background-color: #007bff;
        color: white;
        border: none;
        border-radius: 5px;
        cursor: pointer;
        margin-left: 15px;
        z-index: 1000;
    }

    .adjustment-controls2 button {
        margin-top: 6px;
        padding: 3px;
        background-color: #007bff;
        color: white;
        border: none;
        border-radius: 5px;
        cursor: pointer;
        margin-left: 15px;
        z-index: 1000;
    }

    .adjustment-controls button:hover {
        background-color: #0056b3;
    }

    .adjustment-controls2 button:hover {
        background-color: #0056b3;
    }
}
</style>
