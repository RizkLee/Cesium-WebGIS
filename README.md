# Cesium Vue 三维WebGIS应用

基于 Vue 3 + Vite + Cesium 构建的三维地理信息系统应用。

## 功能特性

- 🌍 三维地球场景展示
- 🗺️ 多种影像底图切换（天地图影像、ArcGIS影像）
- ⛰️ 地形数据加载与切换
- 🎯 相机控制（飞行到指定城市、重置视图）
- 📍 地标标注（北京、上海、成都）
- 📦 三维实体添加（立方体、圆柱、点标注）
- 📊 实时相机位置信息显示
- 🎨 美观的控制面板界面

## 技术栈

- Vue 3 - 渐进式JavaScript框架
- Vite - 下一代前端构建工具
- Cesium - 三维地球和地图可视化库
- vite-plugin-cesium - Cesium的Vite插件

## 本地开发

### 前置要求

- Node.js >= 16.0.0
- npm 或 pnpm

### 安装依赖

```bash
npm install
```

### 配置环境变量

1. 复制 `.env.example` 文件为 `.env`：
```bash
cp .env.example .env
```

2. 编辑 `.env` 文件，填入你的API密钥：
```env
VITE_CESIUM_ION_TOKEN=your_cesium_ion_token_here
VITE_TIANDITU_TOKEN=your_tianditu_token_here
```

**获取API密钥：**
- Cesium Ion Token: 在 [Cesium Ion](https://cesium.com/ion/) 注册账号后获取
- 天地图 Token: 在 [天地图开放平台](https://console.tianditu.gov.cn/) 注册申请

### 启动开发服务器

```bash
npm run dev
```

访问 http://localhost:5173 查看应用。

### 构建生产版本

```bash
npm run build
```

构建产物将生成在 `dist` 目录下。

## 部署到 Cloudflare Pages

### 方法一：通过 Cloudflare Dashboard

1. 将代码推送到 GitHub 仓库

2. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)

3. 进入 **Pages** > **创建项目** > **连接到 Git**

4. 选择你的 GitHub 仓库

5. 配置构建设置：
   - **构建命令**: `npm run build`
   - **构建输出目录**: `dist`
   - **Node 版本**: `16` 或更高

6. 设置环境变量：
   - 在项目设置中，找到 **环境变量** 部分
   - 添加以下变量：
     - `VITE_CESIUM_ION_TOKEN`: 你的 Cesium Ion Token
     - `VITE_TIANDITU_TOKEN`: 你的天地图 Token

7. 点击 **保存并部署**

### 方法二：使用 Wrangler CLI

```bash
# 安装 Wrangler
npm install -g wrangler

# 登录
wrangler login

# 部署
wrangler pages deploy dist
```

部署前确保已在 Cloudflare Dashboard 中设置好环境变量。

## 使用说明

### 相机控制
- **飞行到北京/上海/成都**: 点击相应按钮快速飞往指定城市
- **重置视图**: 返回到中国全景视图
- 使用鼠标拖拽、滚轮缩放、右键旋转控制相机

### 地形设置
- 勾选"启用地形"显示真实地形起伏
- 取消勾选显示平滑椭球体

### 影像底图
- 通过下拉菜单切换不同的影像底图
- 天地图影像：高清卫星影像
- ArcGIS影像：全球影像服务

### 实体添加
- **添加立方体**: 在当前视图中心添加红色半透明立方体
- **添加圆柱**: 在当前视图中心添加蓝色半透明圆柱
- **添加点标注**: 在当前视图中心添加带坐标的黄色标注点
- **清除所有实体**: 删除所有添加的实体（保留初始地标）

## 注意事项

⚠️ **重要**: 
- 不要将 `.env` 文件提交到 Git 仓库
- 确保 `.gitignore` 包含了 `.env` 文件
- 在 Cloudflare Pages 部署时，必须在项目设置中配置环境变量

## 项目结构

```
cesium-vue/
├── public/              # 静态资源
├── src/
│   ├── assets/         # 资源文件
│   ├── components/     # Vue组件
│   │   └── CesiumViewer.vue  # 主要的Cesium视图组件
│   ├── App.vue         # 根组件
│   └── main.js         # 入口文件
├── index.html          # HTML模板
├── vite.config.js      # Vite配置
├── package.json        # 项目配置
├── .env.example        # 环境变量示例
└── README.md           # 项目说明
```

## 许可证

MIT License

## 致谢

- [Cesium](https://cesium.com/) - 强大的三维地球引擎
- [Vue.js](https://vuejs.org/) - 渐进式JavaScript框架
- [Vite](https://vitejs.dev/) - 快速的前端构建工具
- [天地图](https://www.tianditu.gov.cn/) - 国家地理信息公共服务平台
