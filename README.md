# 🖼️ Crawl & Label: 自动化Bing图像爬取 + VLM标注生成PASCAL VOC数据集

![n8n Workflow](https://img.shields.io/badge/n8n-workflow-blue?style=for-the-badge&logo=n8n)
![VLM](https://img.shields.io/badge/VLM-GLM--4.1V--9B--Thinking-green?style=for-the-badge)
![PASCAL VOC](https://img.shields.io/badge/PASCAL%20VOC-XML--Labeling-orange?style=for-the-badge)

> 一键自动从 Bing 搜索图片，调用大模型（如 GLM-4.1V）进行视觉语义理解，并输出标准 PASCAL VOC XML 标注文件，构建高质量目标检测训练数据集。

---

## 🚀 功能亮点

| 特性 | 说明 |
|------|------|
| 🔍 **智能爬取** | 自动翻页抓取 Bing 图片（支持“大图”过滤、自定义关键词、数量、页数） |
| 🧠 **AI标注** | 使用 [GLM-4.1V-9B-Thinking](https://huggingface.co/THUDM/GLM-4.1V-9B-Thinking) 等多模态大模型，精准识别物体并生成边界框（xmin, ymin, xmax, ymax） |
| 📁 **结构化输出** | 输出 `.jpg` 图像 + `.xml` 标注文件，完全符合 [PASCAL VOC](http://host.robots.ox.ac.uk/pascal/VOC/) 标准格式 |
| ⚙️ **全自动流程** | 无需人工干预，从关键词到数据集一键完成 |
| 💾 **灵活配置** | 支持自定义路径、文件名、类别名称、最小尺寸、每页数量等参数 |
| 🗂️ **打包归档** | 最终自动压缩为 `data.zip`，便于传输与训练 |

---

## 🛠️ 技术栈

| 组件 | 说明 |
|------|------|
| **核心引擎** | [n8n](https://n8n.io/) - 低代码工作流平台 |
| **图像来源** | Bing Images（通过网页解析，无官方API依赖） |
| **视觉语言模型** | [GLM-4.1V-9B-Thinking](https://github.com/THUDM/GLM-4-Vision)（硅基流动 SiliconFlow） |
| **标注格式** | PASCAL VOC XML（兼容 TensorFlow、PyTorch、LabelImg、CVAT） |
| **部署方式** | Docker / Linux / macOS / Windows（n8n本地运行） |
| **依赖服务** | [SiliconFlow API](https://siliconflow.cn/)（需注册获取 Key） |

---

## 📥 安装与部署

### 1. 准备环境

- 安装 [n8n](https://docs.n8n.io/getting-started/installation/)（推荐使用 Docker）：
  ```bash
  docker run -it --rm \
    --name n8n \
    -p 5678:5678 \
    -v ~/.n8n:/home/node/.n8n \
    n8nio/n8n
  ```

- 注册 [SiliconFlow](https://siliconflow.cn/) 并获取 **API Key**（免费额度可用）

### 2. 导入工作流

1. 打开 n8n 工作流编辑器：[http://localhost:5678](http://localhost:5678)
2. 点击右上角 **"Import from JSON"**
3. 粘贴本项目中的 [`Crawl&Label_workflow.json`](Crawl&Label_workflow.json) 内容
4. 点击 **"Import"**

### 3. 配置凭证

进入 **Credentials** → 新建 **HTTP Header Auth**：

- **Name**: `Header Auth account 2` （必须与工作流中一致）
- **Header Name**: `Authorization`
- **Header Value**: `Bearer YOUR_SILICONFLOW_API_KEY`

> ✅ **重要**：`Header Auth account 2` 名称**必须完全匹配**，否则 VLM 调用将失败！

### 4. 配置存储路径（Linux/macOS 示例）

在工作流顶部的 **"输入要求"** 节点中，修改路径为你本地可写目录：

```json
"path": "/home/yourname/images",
"filename": "my_dataset_001"
```

> 💡 **Windows 用户**：请使用 `C:\\Users\\YourName\\images` 或挂载 Docker 卷。  
> 💡 **Docker 用户**：建议挂载宿主机目录，例如：`-v ~/images:/home/node/images`

### 5. 运行工作流

点击左上角 **“Execute Workflow”** 按钮，即可开始自动执行：

- 自动创建文件夹 `/your/path/my_dataset_001/`
- 下载指定数量的 Bing 大图
- 每张图调用 VLM 生成对应 XML 标注
- 输出 ZIP 压缩包 `data.zip`

---

## 📝 配置参数详解（修改“输入要求”节点）

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `keyword` | `"变压器"` | 搜索关键词（中文/英文均可，如 `"cat"`, `"dog"`, `"行人"`） |
| `filename` | `"crawl_test2"` | 输出文件夹名称 |
| `type` | `"是变压器/不是变压器"` | **关键字段！** VLM 会根据此字符串判断目标类别。**请用 `/` 分隔正负样本**，如：`"猫/狗"` 或 `"汽车/非汽车"` |
| `num_img` | `30` | 总共要下载并标注的图片数量 |
| `path` | `"/home/node/images"` | 本地存储路径（确保有读写权限） |
| `images_per_page` | `35` | 每次请求返回的图片数（Bing限制，建议 20~50） |
| `total_pages_to_scrape` | `2` | 爬取总页数 → 总图片数 = `images_per_page × total_pages_to_scrape` |

> ⚠️ **重要提示**：  
> - `type` 字段是 VLM 的**唯一类别指令**，直接影响标注准确性。  
> - 推荐格式：`"目标类/背景类"`，例如：  
>   - `"红色变压器/其他设备"`  
>   - `"人/非人"`  
>   - `"红绿灯/非红绿灯"`  
>   - `"手机/非手机"`  
> - **不要使用空格或特殊符号**，避免模型混淆。

---

## 📂 输出结构示例

```
/home/yourname/images/crawl_test2/
├── 0.jpg
├── 0.xml
├── 1.jpg
├── 1.xml
├── ...
├── 29.jpg
├── 29.xml
└── data.zip ← 自动生成的完整数据集压缩包
```

### XML 标注样例（PASCAL VOC 标准）

```xml
<annotation>
	<folder>crawl_test2</folder>
	<filename>0.jpg</filename>
	<path>/home/yourname/images/crawl_test2/0.jpg</path>
	<source>
		<database>Unknown</database>
	</source>
	<size>
		<width>1280</width>
		<height>720</height>
		<depth>3</depth>
	</size>
	<segmented>0</segmented>
	<object>
		<name>是变压器</name>
		<pose>Unspecified</pose>
		<truncated>0</truncated>
		<difficult>0</difficult>
		<bndbox>
			<xmin>210</xmin>
			<ymin>150</ymin>
			<xmax>320</xmax>
			<ymax>280</ymax>
		</bndbox>
	</object>
</annotation>
```

> ✅ **兼容性**：该格式可直接导入 [LabelImg](https://github.com/tzutalin/labelImg)、[CVAT](https://cvat.org/)、[YOLOv8](https://docs.ultralytics.com/modes/detect/#train) 等工具训练模型。

---

## 🌟 高级技巧与优化建议

### ✅ 提升标注准确率
- 在 `type` 中增加描述词，如：`"红色变压器/其他设备"` → 指导模型关注颜色与形态
- 在 prompt 中加入：“只标注清晰可见的物体，遮挡超过50%忽略”

### ✅ 增强鲁棒性（可选）
- 添加 `retryOnFail` 和超时设置（已配置）
- 加入图像尺寸过滤（可扩展 `Filter1` 节点，检查 `width` 和 `height`）
- 增加图像去重（基于 MD5 哈希）

### ✅ 扩展方向
| 方向 | 说明 |
|------|------|
| 🔄 多类别支持 | 修改 `type` 为 `"猫/狗/鸟"`，VLM 可输出多个 `<object>` |
| 📊 自动统计 | 在末尾添加节点统计成功/失败数量，写入 `summary.json` |
| 🤖 集成 Label Studio | 将 XML 转换为 Label Studio JSON 格式 |
| 📦 Docker镜像 | 将整个 n8n + workflow 打包为可复用镜像 |
| 📈 数据增强 | 后续接入 Albumentations/Augmentor 对数据集做增强 |

---

## 📜 许可证

MIT License - See [LICENSE](LICENSE) for details.

---

## 🙏 致谢

- [n8n](https://n8n.io/) —— 强大的开源自动化平台
- [SiliconFlow](https://siliconflow.cn/) —— 提供高性能国产多模态模型
- [GLM-4-Vision](https://github.com/THUDM/GLM-4-Vision) —— 优秀的开源视觉语言模型
- [PASCAL VOC](http://host.robots.ox.ac.uk/pascal/VOC/) —— 标注标准奠基者
