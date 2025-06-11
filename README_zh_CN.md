# 3Dmol.js + 思源笔记 技术要点总结

## 1. 3Dmol.js 引入和图层控制

### 基本引入

```html
<!-- 3Dmol.js CDN -->
<script src="https://3dmol.org/build/3Dmol-min.js"></script>
```

###  关键CSS - 防止WebGL逃逸到图层外

```css
/* 3Dmol.js 容器样式 - 关键修复 */
#molecule-viewer {
    position: relative !important;
    width: 100% !important;
    height: 100% !important;
    overflow: hidden !important;
}

/* 确保3Dmol.js的canvas元素正确定位 */
#molecule-viewer canvas {
    position: absolute !important;
    top: 0 !important;
    left: 0 !important;
    width: 100% !important;
    height: 100% !important;
    z-index: 1 !important;
}

/* 防止3Dmol.js的WebGL元素逃逸 */
#molecule-viewer > div {
    position: relative !important;
    width: 100% !important;
    height: 100% !important;
    overflow: hidden !important;
}
```

### 基本初始化

```javascript
const config = { 
    backgroundColor: 'white',
    width: rect.width,
    height: rect.height
};

this.viewer = $3Dmol.createViewer(element, config);
```

---

## 2. 3Dmol.js 核心API

### 分子加载和显示

```javascript
// 加载分子数据
this.viewer.addModel(xyzData, "xyz");

// 设置显示样式
this.viewer.setStyle({}, {stick: {}, sphere: {scale: 0.3}}); // 球棍模型
this.viewer.setStyle({}, {sphere: {scale: 0.8}});           // 球型模型  
this.viewer.setStyle({}, {line: {}});                       // 线型模型

// 设置背景颜色
this.viewer.setBackgroundColor('white');

// 缩放到合适大小并渲染
this.viewer.zoomTo();
this.viewer.render();
```

###  视图状态保存和恢复（核心功能）

```javascript
// 获取当前视图状态 - 直接访问内部对象
getCurrentViewState() {
    const rotationGroup = this.viewer.rotationGroup;
    const modelGroup = this.viewer.modelGroup;
    
    return {
        // 旋转组的位置和旋转
        rotationPosition: {
            x: rotationGroup.position.x,
            y: rotationGroup.position.y,
            z: rotationGroup.position.z
        },
        rotationQuaternion: {
            x: rotationGroup.quaternion.x,
            y: rotationGroup.quaternion.y,
            z: rotationGroup.quaternion.z,
            w: rotationGroup.quaternion.w
        },
        // 模型组的位置
        modelPosition: {
            x: modelGroup.position.x,
            y: modelGroup.position.y,
            z: modelGroup.position.z
        },
        // 相机参数
        cameraZ: this.viewer.CAMERA_Z
    };
}

// 应用视图状态 - 直接设置内部对象
applyViewState(viewState) {
    // 设置旋转组位置和旋转
    this.viewer.rotationGroup.position.set(
        viewState.rotationPosition.x,
        viewState.rotationPosition.y,
        viewState.rotationPosition.z
    );
    
    this.viewer.rotationGroup.quaternion.set(
        viewState.rotationQuaternion.x,
        viewState.rotationQuaternion.y,
        viewState.rotationQuaternion.z,
        viewState.rotationQuaternion.w
    );
    
    // 设置模型组位置
    this.viewer.modelGroup.position.set(
        viewState.modelPosition.x,
        viewState.modelPosition.y,
        viewState.modelPosition.z
    );
    
    // 设置相机距离
    this.viewer.CAMERA_Z = viewState.cameraZ;
    
    this.viewer.render();
}
```

### 原子标签显示

```javascript
// 显示元素符号标签
this.viewer.addPropertyLabels("elem", {}, {
    fontColor: 'black',
    font: 'Arial',
    fontSize: 14,
    showBackground: true,
    backgroundColor: 'white',
    backgroundOpacity: 0.8,
    alignment: 'center'
});

// 显示原子序号标签（带位置偏移）
this.viewer.addPropertyLabels("serial", {}, {
    fontColor: 'blue',
    font: 'Arial',
    fontSize: 12,
    showBackground: true,
    backgroundColor: 'lightyellow',
    backgroundOpacity: 0.7,
    alignment: 'center',
    screenOffset: {x: 15, y: -10} // 偏移避免重叠
});

// 移除所有标签
this.viewer.removeAllLabels();
```

### 其他常用API

```javascript
// 清除所有内容
this.viewer.clear();

// 重置视图到默认位置
this.viewer.zoomTo();

// 手动渲染（状态改变后需要调用）
this.viewer.render();
```

---

## 3. 思源笔记 API

###  获取挂件块ID

```javascript
getBlockId() {
    try {
        // 获取当前挂件所在的块ID
        const id = window.frameElement?.parentElement?.parentElement?.getAttribute("data-node-id");
        return id || 'default';
    } catch (error) {
        console.warn('无法获取块ID，使用默认ID');
        return 'default';
    }
}
```

### 文件操作API

#### 保存文件

```javascript
async putFileContent(path, content) {
    const formData = new FormData();
    formData.append("path", path);
    formData.append("file", new Blob([content], { type: 'application/json' }));
    
    try {
        const response = await fetch("/api/file/putFile", {
            method: "POST",
            body: formData,
        });
        
        if (response.ok) {
            return true;
        } else {
            throw new Error("保存文件失败");
        }
    } catch (error) {
        console.error("写入文件错误:", error);
        throw error;
    }
}
```

#### 读取文件

```javascript
async getFileContent(path) {
    try {
        const response = await fetch("/api/file/getFile", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
            },
            body: JSON.stringify({ path }),
        });
        
        if (response.ok) {
            return await response.text();
        } else if (response.status === 404) {
            return null; // 文件不存在
        } else {
            throw new Error("读取文件失败");
        }
    } catch (error) {
        console.error("读取文件错误:", error);
        return null;
    }
}
```

### 文件路径规范

```javascript
// 推荐的数据存储路径
this.dataPath = "/data/widgets/molecule-viewer/";

// 完整文件路径示例
const filePath = `${this.dataPath}${this.blockId}.json`;
// 结果: /data/widgets/molecule-viewer/20241210123456-abc123.json
```

---

## 4. 数据持久化结构

### JSON数据格式

```javascript
const data = {
    input: inputData,              // 原始分子数据
    format: this.currentFormat,    // 文件格式 (XYZ/ORCA/GAUSSIAN)
    style: this.currentStyle,      // 显示样式 (stick/sphere/line)
    background: this.currentBackground, // 背景颜色
    viewState: viewState,          // 完整的视图状态
    isViewLocked: this.isViewLocked, // 是否锁定视图
    showingAtomLabels: this.showingAtomLabels, // 原子标签状态
    timestamp: Date.now()          // 保存时间戳
};
```

---

## 5. 关键技术难点解决方案

### ❌ 错误方法：使用getView/setView

```javascript
// 这些API不可靠，经常失效
const view = this.viewer.getView(); // 返回的数据不完整
this.viewer.setView(view);          // 无法正确恢复
```

### ✅ 正确方法：直接操作内部对象

```javascript
// 直接访问和设置3Dmol.js的内部变换对象
this.viewer.rotationGroup.position.set(x, y, z);
this.viewer.rotationGroup.quaternion.set(x, y, z, w);
this.viewer.modelGroup.position.set(x, y, z);
this.viewer.CAMERA_Z = z;
```

###  CSS防逃逸的核心原理

- 3Dmol.js使用WebGL canvas渲染，默认可能溢出容器
- 通过`position: relative`和`overflow: hidden`限制范围
- 强制设置canvas的`position: absolute`确保定位正确
- `z-index: 1`防止层级问题

---

## 6. 最佳实践

1. **初始化顺序**：先创建viewer → 加载模型 → 设置样式 → 渲染
2. **状态管理**：每次状态改变后都要调用`render()`
3. **错误处理**：文件操作和3D渲染都需要try-catch包装
4. **性能优化**：避免频繁的`render()`调用，可以用防抖
5. **用户体验**：提供加载状态和错误提示