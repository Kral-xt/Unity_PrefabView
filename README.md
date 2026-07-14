# Unity_PrefabView
An exclusive plugin dedicated to prefab preview in Unity

Unity Prefab 实时预览插件原理
Unity 对材质、模型和纹理提供了 Inspector 预览，但普通 Prefab，尤其是 UI Prefab，通常无法直接查看完整效果。这个插件通过在编辑器中临时实例化 Prefab，并将渲染结果显示在独立窗口中，实现实时预览。
1. Prefab 选择
插件监听 Unity 的 Selection.selectionChanged 事件。
当用户在 Project 窗口中选中 Prefab 时，插件会：
获取 Prefab 资源路径。
判断它是普通 3D Prefab 还是 UI Prefab。
创建一个仅用于预览的临时实例。
在选择变化时销毁旧实例并加载新实例。
所有临时对象都会使用 HideFlags.HideAndDontSave，因此不会出现在场景层级中，也不会被保存到场景。
2. 3D Prefab 预览
3D Prefab 使用 Unity Editor 提供的 PreviewRenderUtility 渲染。
插件会收集 Prefab 中的：
MeshFilter
MeshRenderer
SkinnedMeshRenderer
Material
然后计算所有 Renderer 的整体包围盒，根据包围盒自动设置摄像机距离和取景范围。
渲染流程如下：
Prefab Mesh
    ↓
计算 Bounds
    ↓
设置预览摄像机和灯光
    ↓
PreviewRenderUtility 渲染
    ↓
绘制到 EditorWindow
鼠标拖动会修改模型旋转角度，滚轮则控制摄像机缩放。
3. UI Prefab 预览
UI Prefab 不能直接使用普通 Mesh 预览，因为它依赖：
Canvas
RectTransform
CanvasRenderer
CanvasScaler
UGUI 布局系统
插件会创建一个临时 World Space Canvas，将 UI Prefab 挂载到该 Canvas 下，再使用正交摄像机渲染到 RenderTexture。
UI Prefab
    ↓
临时 Canvas
    ↓
重建 RectTransform 和 Layout
    ↓
正交摄像机渲染
    ↓
RenderTexture
    ↓
显示在预览窗口
渲染前会调用：
Canvas.ForceUpdateCanvases
LayoutRebuilder.ForceRebuildLayoutImmediate
TMP_Text.ForceMeshUpdate
这样可以确保布局组件和 TextMeshPro 文本在编辑模式下及时刷新。
4. 自动取景
插件遍历所有可见的 Graphic，读取对应 RectTransform 的四个世界坐标角点，再转换到预览 Canvas 的局部空间中。
合并这些坐标后即可得到整个 UI 的实际显示边界。
摄像机根据边界宽高和预览窗口宽高比计算正交尺寸，从而让 UI 自动居中并完整显示。
5. 横竖屏适配
插件提供两种参考画布：
竖屏：1080 × 1920
横屏：1920 × 1080
切换画布后，插件会重新创建 UI 预览环境，并重新计算锚点、布局和可见区域，以便检查不同屏幕方向下的 UI 表现。
6. Unity 6 的注意事项
在 Unity 6 中，将 UGUI Canvas 放入自建 PreviewScene 后，Camera.Render() 可能无法获得 Canvas 的渲染批次，最终得到的 RenderTexture 只有背景色。
解决方法是将 UI 预览对象放在普通已加载场景中，同时设置：
HideFlags.HideAndDontSave
这样既能让 Unity 正常提交 UGUI 渲染数据，又不会污染场景层级或保存场景。预览窗口关闭或切换 Prefab 时，再立即销毁这些临时对象。
总结
这个插件的核心是为不同类型的 Prefab 使用不同的渲染方式：
3D Prefab：PreviewRenderUtility
UI Prefab：临时 Canvas、正交摄像机和 RenderTexture
自动取景：Renderer Bounds 或 RectTransform 世界角点
编辑器隔离：HideFlags.HideAndDontSave
它不会修改原始 Prefab，也不会向场景写入持久内容，只在编辑器中创建临时预览环境。




调用方法为Tools/PrefabPreview
