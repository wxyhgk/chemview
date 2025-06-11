# 3Dmol.js + ˼Դ�ʼ� ����Ҫ���ܽ�

## 1. 3Dmol.js �����ͼ�����

### ��������

```html
<!-- 3Dmol.js CDN -->
<script src="https://3dmol.org/build/3Dmol-min.js"></script>
```

###  �ؼ�CSS - ��ֹWebGL���ݵ�ͼ����

```css
/* 3Dmol.js ������ʽ - �ؼ��޸� */
#molecule-viewer {
    position: relative !important;
    width: 100% !important;
    height: 100% !important;
    overflow: hidden !important;
}

/* ȷ��3Dmol.js��canvasԪ����ȷ��λ */
#molecule-viewer canvas {
    position: absolute !important;
    top: 0 !important;
    left: 0 !important;
    width: 100% !important;
    height: 100% !important;
    z-index: 1 !important;
}

/* ��ֹ3Dmol.js��WebGLԪ������ */
#molecule-viewer > div {
    position: relative !important;
    width: 100% !important;
    height: 100% !important;
    overflow: hidden !important;
}
```

### ������ʼ��

```javascript
const config = { 
    backgroundColor: 'white',
    width: rect.width,
    height: rect.height
};

this.viewer = $3Dmol.createViewer(element, config);
```

---

## 2. 3Dmol.js ����API

### ���Ӽ��غ���ʾ

```javascript
// ���ط�������
this.viewer.addModel(xyzData, "xyz");

// ������ʾ��ʽ
this.viewer.setStyle({}, {stick: {}, sphere: {scale: 0.3}}); // ���ģ��
this.viewer.setStyle({}, {sphere: {scale: 0.8}});           // ����ģ��  
this.viewer.setStyle({}, {line: {}});                       // ����ģ��

// ���ñ�����ɫ
this.viewer.setBackgroundColor('white');

// ���ŵ����ʴ�С����Ⱦ
this.viewer.zoomTo();
this.viewer.render();
```

###  ��ͼ״̬����ͻָ������Ĺ��ܣ�

```javascript
// ��ȡ��ǰ��ͼ״̬ - ֱ�ӷ����ڲ�����
getCurrentViewState() {
    const rotationGroup = this.viewer.rotationGroup;
    const modelGroup = this.viewer.modelGroup;
    
    return {
        // ��ת���λ�ú���ת
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
        // ģ�����λ��
        modelPosition: {
            x: modelGroup.position.x,
            y: modelGroup.position.y,
            z: modelGroup.position.z
        },
        // �������
        cameraZ: this.viewer.CAMERA_Z
    };
}

// Ӧ����ͼ״̬ - ֱ�������ڲ�����
applyViewState(viewState) {
    // ������ת��λ�ú���ת
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
    
    // ����ģ����λ��
    this.viewer.modelGroup.position.set(
        viewState.modelPosition.x,
        viewState.modelPosition.y,
        viewState.modelPosition.z
    );
    
    // �����������
    this.viewer.CAMERA_Z = viewState.cameraZ;
    
    this.viewer.render();
}
```

### ԭ�ӱ�ǩ��ʾ

```javascript
// ��ʾԪ�ط��ű�ǩ
this.viewer.addPropertyLabels("elem", {}, {
    fontColor: 'black',
    font: 'Arial',
    fontSize: 14,
    showBackground: true,
    backgroundColor: 'white',
    backgroundOpacity: 0.8,
    alignment: 'center'
});

// ��ʾԭ����ű�ǩ����λ��ƫ�ƣ�
this.viewer.addPropertyLabels("serial", {}, {
    fontColor: 'blue',
    font: 'Arial',
    fontSize: 12,
    showBackground: true,
    backgroundColor: 'lightyellow',
    backgroundOpacity: 0.7,
    alignment: 'center',
    screenOffset: {x: 15, y: -10} // ƫ�Ʊ����ص�
});

// �Ƴ����б�ǩ
this.viewer.removeAllLabels();
```

### ��������API

```javascript
// �����������
this.viewer.clear();

// ������ͼ��Ĭ��λ��
this.viewer.zoomTo();

// �ֶ���Ⱦ��״̬�ı����Ҫ���ã�
this.viewer.render();
```

---

## 3. ˼Դ�ʼ� API

###  ��ȡ�Ҽ���ID

```javascript
getBlockId() {
    try {
        // ��ȡ��ǰ�Ҽ����ڵĿ�ID
        const id = window.frameElement?.parentElement?.parentElement?.getAttribute("data-node-id");
        return id || 'default';
    } catch (error) {
        console.warn('�޷���ȡ��ID��ʹ��Ĭ��ID');
        return 'default';
    }
}
```

### �ļ�����API

#### �����ļ�

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
            throw new Error("�����ļ�ʧ��");
        }
    } catch (error) {
        console.error("д���ļ�����:", error);
        throw error;
    }
}
```

#### ��ȡ�ļ�

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
            return null; // �ļ�������
        } else {
            throw new Error("��ȡ�ļ�ʧ��");
        }
    } catch (error) {
        console.error("��ȡ�ļ�����:", error);
        return null;
    }
}
```

### �ļ�·���淶

```javascript
// �Ƽ������ݴ洢·��
this.dataPath = "/data/widgets/molecule-viewer/";

// �����ļ�·��ʾ��
const filePath = `${this.dataPath}${this.blockId}.json`;
// ���: /data/widgets/molecule-viewer/20241210123456-abc123.json
```

---

## 4. ���ݳ־û��ṹ

### JSON���ݸ�ʽ

```javascript
const data = {
    input: inputData,              // ԭʼ��������
    format: this.currentFormat,    // �ļ���ʽ (XYZ/ORCA/GAUSSIAN)
    style: this.currentStyle,      // ��ʾ��ʽ (stick/sphere/line)
    background: this.currentBackground, // ������ɫ
    viewState: viewState,          // ��������ͼ״̬
    isViewLocked: this.isViewLocked, // �Ƿ�������ͼ
    showingAtomLabels: this.showingAtomLabels, // ԭ�ӱ�ǩ״̬
    timestamp: Date.now()          // ����ʱ���
};
```

---

## 5. �ؼ������ѵ�������

### �7�4 ���󷽷���ʹ��getView/setView

```javascript
// ��ЩAPI���ɿ�������ʧЧ
const view = this.viewer.getView(); // ���ص����ݲ�����
this.viewer.setView(view);          // �޷���ȷ�ָ�
```

### �7�3 ��ȷ������ֱ�Ӳ����ڲ�����

```javascript
// ֱ�ӷ��ʺ�����3Dmol.js���ڲ��任����
this.viewer.rotationGroup.position.set(x, y, z);
this.viewer.rotationGroup.quaternion.set(x, y, z, w);
this.viewer.modelGroup.position.set(x, y, z);
this.viewer.CAMERA_Z = z;
```

###  CSS�����ݵĺ���ԭ��

- 3Dmol.jsʹ��WebGL canvas��Ⱦ��Ĭ�Ͽ����������
- ͨ��`position: relative`��`overflow: hidden`���Ʒ�Χ
- ǿ������canvas��`position: absolute`ȷ����λ��ȷ
- `z-index: 1`��ֹ�㼶����

---

## 6. ���ʵ��

1. **��ʼ��˳��**���ȴ���viewer �� ����ģ�� �� ������ʽ �� ��Ⱦ
2. **״̬����**��ÿ��״̬�ı��Ҫ����`render()`
3. **������**���ļ�������3D��Ⱦ����Ҫtry-catch��װ
4. **�����Ż�**������Ƶ����`render()`���ã������÷���
5. **�û�����**���ṩ����״̬�ʹ�����ʾ