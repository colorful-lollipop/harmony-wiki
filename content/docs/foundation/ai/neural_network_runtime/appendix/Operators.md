# 附录 E: 算子列表

## 算子统计

- **总算子数**: 110+
- **分类**: 基础运算、激活函数、卷积、归一化、池化、形状操作、归约、比较等

## 算子分类列表

### 基础运算 (Element-wise)

| 算子 | 枚举值 | 说明 |
|------|--------|------|
| Add | OH_NN_OPS_ADD | 加法 |
| Sub | OH_NN_OPS_SUB | 减法 |
| Mul | OH_NN_OPS_MUL | 乘法 |
| Div | OH_NN_OPS_DIV | 除法 |
| Pow | OH_NN_OPS_POW | 幂运算 |
| Sqrt | OH_NN_OPS_SQRT | 平方根 |
| Rsqrt | OH_NN_OPS_RSQRT | 平方根倒数 |
| Exp | OH_NN_OPS_EXP | 指数 |
| Log | OH_NN_OPS_LOG | 对数 |
| Abs | OH_NN_OPS_ABS | 绝对值 |
| Neg | OH_NN_OPS_NEG | 取反 |
| Floor | OH_NN_OPS_FLOOR | 向下取整 |
| Ceil | OH_NN_OPS_CEIL | 向上取整 |
| Round | OH_NN_OPS_ROUND | 四舍五入 |
| Reciprocal | OH_NN_OPS_RECIPROCAL | 倒数 |

### 激活函数 (Activation)

| 算子 | 枚举值 | 说明 |
|------|--------|------|
| Relu | OH_NN_OPS_RELU | ReLU 激活 |
| Relu6 | OH_NN_OPS_RELU6 | ReLU6 激活 |
| Sigmoid | OH_NN_OPS_SIGMOID | Sigmoid 激活 |
| Tanh | OH_NN_OPS_TANH | Tanh 激活 |
| Gelu | OH_NN_OPS_GELU | GELU 激活 |
| Hswish | OH_NN_OPS_HSWISH | Hard Swish 激活 |
| Swish | OH_NN_OPS_SWISH | Swish 激活 |
| Softmax | OH_NN_OPS_SOFTMAX | Softmax 激活 |
| LogSoftmax | OH_NN_OPS_LOG_SOFTMAX | Log Softmax |
| LeakyRelu | OH_NN_OPS_LEAKY_RELU | Leaky ReLU |
| PRelu | OH_NN_OPS_PRELU | PReLU |
| HardSigmoid | OH_NN_OPS_HARD_SIGMOID | Hard Sigmoid |

### 卷积 (Convolution)

| 算子 | 枚举值 | 说明 |
|------|--------|------|
| Conv2D | OH_NN_OPS_CONV2D | 2D 卷积 |
| Conv2DTranspose | OH_NN_OPS_CONV2D_TRANSPOSE | 2D 转置卷积 |
| DepthwiseConv2D | OH_NN_OPS_DEPTHWISE_CONV2D_NATIVE | 深度可分离卷积 |

### 归一化 (Normalization)

| 算子 | 枚举值 | 说明 |
|------|--------|------|
| BatchNorm | OH_NN_OPS_BATCH_NORM | 批归一化 |
| LayerNorm | OH_NN_OPS_LAYERNORM | 层归一化 |
| InstanceNorm | OH_NN_OPS_INSTANCE_NORM | 实例归一化 |
| L2Normalize | OH_NN_OPS_L2_NORMALIZE | L2 归一化 |

### 池化 (Pooling)

| 算子 | 枚举值 | 说明 |
|------|--------|------|
| AvgPool | OH_NN_OPS_AVG_POOL | 平均池化 |
| MaxPool | OH_NN_OPS_MAX_POOL | 最大池化 |

### 形状操作 (Shape Manipulation)

| 算子 | 枚举值 | 说明 |
|------|--------|------|
| Reshape | OH_NN_OPS_RESHAPE | 重塑形状 |
| Squeeze | OH_NN_OPS_SQUEEZE | 去除维度 |
| Unsqueeze | OH_NN_OPS_UNSQUEEZE | 增加维度 |
| ExpandDims | OH_NN_OPS_EXPAND_DIMS | 扩展维度 |
| Transpose | OH_NN_OPS_TRANSPOSE | 转置 |
| Concat | OH_NN_OPS_CONCAT | 拼接 |
| Split | OH_NN_OPS_SPLIT | 分割 |
| Slice | OH_NN_OPS_SLICE | 切片 |
| StridedSlice | OH_NN_OPS_STRIDED_SLICE | 步长切片 |
| Gather | OH_NN_OPS_GATHER | 收集 |
| GatherND | OH_NN_OPS_GATHER_ND | N 维收集 |
| ScatterND | OH_NN_OPS_SCATTER_ND | N 维散射 |
| Tile | OH_NN_OPS_TILE | 平铺 |
| Pad | OH_NN_OPS_PAD | 填充 |
| Crop | OH_NN_OPS_CROP | 裁剪 |
| Flatten | OH_NN_OPS_FLATTEN | 展平 |
| BatchToSpaceND | OH_NN_OPS_BATCH_TO_SPACE_ND | 批次到空间 |
| SpaceToBatchND | OH_NN_OPS_SPACE_TO_BATCH_ND | 空间到批次 |
| SpaceToDepth | OH_NN_OPS_SPACE_TO_DEPTH | 空间到深度 |
| DepthToSpace | OH_NN_OPS_DEPTH_TO_SPACE | 深度到空间 |
| Stack | OH_NN_OPS_STACK | 堆叠 |
| Unstack | OH_NN_OPS_UNSTACK | 拆堆 |

### 归约 (Reduction)

| 算子 | 枚举值 | 说明 |
|------|--------|------|
| ReduceSum | OH_NN_OPS_REDUCE_SUM | 求和 |
| ReduceMean | OH_NN_OPS_REDUCE_MEAN | 求平均 |
| ReduceMax | OH_NN_OPS_REDUCE_MAX | 求最大 |
| ReduceMin | OH_NN_OPS_REDUCE_MIN | 求最小 |
| ReduceAll | OH_NN_OPS_REDUCE_ALL | 逻辑与归约 |
| ReduceProd | OH_NN_OPS_REDUCE_PROD | 求积 |
| ReduceL2 | OH_NN_OPS_REDUCE_L2 | L2 归约 |
| ArgMax | OH_NN_OPS_ARG_MAX | 最大值的索引 |
| TopK | OH_NN_OPS_TOP_K | 前 K 个值 |
| All | OH_NN_OPS_ALL | 逻辑与 |

### 比较 (Comparison)

| 算子 | 枚举值 | 说明 |
|------|--------|------|
| Equal | OH_NN_OPS_EQUAL | 等于 |
| Greater | OH_NN_OPS_GREATER | 大于 |
| GreaterEqual | OH_NN_OPS_GREATER_EQUAL | 大于等于 |
| Less | OH_NN_OPS_LESS | 小于 |
| LessEqual | OH_NN_OPS_LESS_EQUAL | 小于等于 |
| NotEqual | OH_NN_OPS_NOT_EQUAL | 不等于 |

### 逻辑运算 (Logical)

| 算子 | 枚举值 | 说明 |
|------|--------|------|
| LogicalAnd | OH_NN_OPS_LOGICAL_AND | 逻辑与 |
| LogicalOr | OH_NN_OPS_LOGICAL_OR | 逻辑或 |
| LogicalNot | OH_NN_OPS_LOGICAL_NOT | 逻辑非 |
| Select | OH_NN_OPS_SELECT | 条件选择 |

### 矩阵运算 (Matrix)

| 算子 | 枚举值 | 说明 |
|------|--------|------|
| MatMul | OH_NN_OPS_MATMUL | 矩阵乘法 |
| FullConnection | OH_NN_OPS_FULL_CONNECTION | 全连接 |

### 其他 (Others)

| 算子 | 枚举值 | 说明 |
|------|--------|------|
| Cast | OH_NN_OPS_CAST | 类型转换 |
| BiasAdd | OH_NN_OPS_BIAS_ADD | 偏置加法 |
| Eltwise | OH_NN_OPS_ELTWISE | 逐元素运算 |
| Scale | OH_NN_OPS_SCALE | 缩放 |
| LSTM | OH_NN_OPS_LSTM | 长短期记忆网络 |
| OneHot | OH_NN_OPS_ONE_HOT | One-Hot 编码 |
| Fill | OH_NN_OPS_FILL | 填充常量 |
| ConstantOfShape | OH_NN_OPS_CONSTANT_OF_SHAPE | 形状常量 |
| Range | OH_NN_OPS_RANGE | 范围 |
| Rank | OH_NN_OPS_RANK | 秩 |
| Shape | OH_NN_OPS_SHAPE | 形状 |
| Size | OH_NN_OPS_SIZE | 大小 |
| Where | OH_NN_OPS_WHERE | 条件选择 |
| DetectionPostProcess | OH_NN_OPS_DETECTION_POST_PROCESS | 检测后处理 |
| QuantDTypeCast | OH_NN_OPS_QUANT_DTYPE_CAST | 量化类型转换 |
| SparseToDense | OH_NN_OPS_SPARSE_TO_DENSE | 稀疏到稠密 |
| LRN | OH_NN_OPS_LRN | 局部响应归一化 |
| Erf | OH_NN_OPS_ERF | 误差函数 |
| Assert | OH_NN_OPS_ASSERT | 断言 |
| ResizeBilinear | OH_NN_OPS_RESIZE_BILINEAR | 双线性插值调整大小 |
| Maximum | OH_NN_OPS_MAXIMUM | 最大值 |
| Minimum | OH_NN_OPS_MINIMUM | 最小值 |
| Mod | OH_NN_OPS_MOD | 取模 |
| SquaredDifference | OH_NN_OPS_SQUARED_DIFFERENCE | 平方差 |

## 算子实现文件

算子实现位于 `frameworks/native/neural_network_runtime/ops/` 目录：

```
ops/
├── abs_builder.h/cpp
├── add_builder.h/cpp
├── avgpool_builder.h/cpp
├── conv2d_builder.h/cpp
├── ... (110+ 个算子文件)
└── ops_validation.cpp
```

## 算子注册

算子通过宏注册到 `OpsRegistry`：

```cpp
// 在 ops/*_builder.cpp 文件中
REGISTER_OPS(AddBuilder, OH_NN_OPS_ADD);
REGISTER_OPS(Conv2DBuilder, OH_NN_OPS_CONV2D);
// ...
```

**证据**: `frameworks/native/neural_network_runtime/ops_registry.h`

## 算子查询

查询设备是否支持特定算子：

```c
// 获取设备支持的算子列表
const bool* isSupported = NULL;
uint32_t opCount = 0;
OH_NNModel_GetAvailableOperations(model, deviceID, &isSupported, &opCount);

// 检查特定算子是否支持
if (isSupported[OH_NN_OPS_CONV2D]) {
    printf("Conv2D is supported\n");
}
```

## 相关跳转

- [对外 API](../04_Native_API.md)
- [API 速查](API_Quick_Reference.md)
- [目录结构](../03_Directory_Structure.md)
