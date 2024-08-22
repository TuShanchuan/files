const barData = computed(() => {
    const filteredData = timeFilteredData.value.filter((item) => {
        // 当型号有值时，过滤型号
        return !selectedXinghao.value || item.xinghao == selectedXinghao.value;
    });
    const map = new Map();
    if (!centerX.value || !centerY.value)
        return {
            defectiveEngines: 0,
            qualifiedEngines: 0,
        };
    filteredData.forEach((engine) => {
        const [x, y] = engine.xyhw.split(',').map(Number);
        // 圆心X和圆心Y有数据的时候
        // 筛选逻辑有问题
        const distance = Math.sqrt(
            Math.pow(x - centerX.value, 2) + Math.pow(y - centerY.value, 2)
        );
        if (
            innerRadius.value &&
            distance >= innerRadius.value &&
            outerRadius.value &&
            distance <= outerRadius.value
        ) {
            map.set('defective', (map.get('defective') || 0) + 1);
        }
        // if (outerRadius.value && distance > outerRadius.value) {
        //     map.set('defective', (map.get('qualified') || 0) + 1);
        // }
        else {
            map.set('qualified', (map.get('qualified') || 0) + 1);
        }
    });
    return {
        defectiveEngines: map.get('defective') || 0,
        qualifiedEngines: map.get('qualified') || 0,
    };
});

// 缺陷类别分布-饼图-数据
const pieData = computed(() => {
    const filteredData = timeFilteredData.value.filter((item) => {
        // 当型号有值时，过滤型号
        return !selectedXinghao.value || item.xinghao == selectedXinghao.value;
    });
    console.log('timeFilteredData len:', timeFilteredData.value.length);
    const defectDataMap = new Map();
    if (!centerX.value || !centerY.value) return [];
    filteredData.forEach((engine) => {
        const [x, y] = engine.xyhw.split(',').map(Number);
        // 圆心X和圆心Y有数据的时候
        //计算缺陷位置到圆心的距离
        const distance = Math.sqrt(
            Math.pow(x - centerX.value, 2) + Math.pow(y - centerY.value, 2)
        );

        if (
            innerRadius.value &&
            distance >= innerRadius.value &&
            outerRadius.value &&
            distance <= outerRadius.value
        ) {
            console.log(
                'x, y, distance, inner, outer:',
                x,
                y,
                distance,
                innerRadius.value,
                outerRadius.value
            );
            defectDataMap.set(
                engine.quexianleibie,
                (defectDataMap.get(engine.quexianleibie) || 0) + 1
            );
        }
        // if (outerRadius.value && distance <= outerRadius.value) {
        //     defectDataMap.set(
        //         engine.quexianleibie,
        //         (defectDataMap.get(engine.quexianleibie) || 0) + 1
        //     );
        // }
    });
    const result = [];
    defectDataMap.forEach((value, key) => {
        result.push({ name: key, value });
    });
    return result;
});
