```dart
void runApp(Widget app) {
  WidgetsFlutterBinding.ensureInitialized()
    ..scheduleAttachRootWidget(app)
    ..scheduleWarmUpFrame();
}


// WidgetsFlutterBinding.ensureInitialized()
// 创建 WidgetsFlutterBinding 实例
// https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/widgets/binding.dart#L1241
static WidgetsBinding ensureInitialized() {
  if (WidgetsBinding.instance == null)
    WidgetsFlutterBinding();
  return WidgetsBinding.instance!;
}


// https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/foundation/binding.dart#L52
BindingBase() {
  // ...
  initInstances();
  // ...
}


// WidgetsBinding.initInstances()
// https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/widgets/binding.dart#L276
void initInstances() {
  // ...
  _instance = this; // 设置 WidgetsBinding.instance 为 WidgetsFlutterBinding()
  // ...
  _buildOwner = BuildOwner();
  buildOwner!.onBuildScheduled = _handleBuildScheduled;
  // ...
}


// RendererBinding.initInstances()
// https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/rendering/binding.dart#L27
void initInstances() {
  _instance = this;
  _pipelineOwner = PipelineOwner(
    onNeedVisualUpdate: ensureVisualUpdate,
    onSemanticsOwnerCreated: _handleSemanticsOwnerCreated,
    onSemanticsOwnerDisposed: _handleSemanticsOwnerDisposed,
  );
  window
    ..onMetricsChanged = handleMetricsChanged
    ..onTextScaleFactorChanged = handleTextScaleFactorChanged
    ..onPlatformBrightnessChanged = handlePlatformBrightnessChanged
    ..onSemanticsEnabledChanged = _handleSemanticsEnabledChanged
    ..onSemanticsAction = _handleSemanticsAction;
  initRenderView();
  // ...
}


// PaintingBinding.initInstances()
void initInstances() {
  // ...
  _instance = this;
  _imageCache = createImageCache();
  shaderWarmUp?.execute();
}
```


// RendererBinding.initRenderView()
// https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/rendering/binding.dart#L167
void initRenderView() {
  // ...
  renderView = RenderView(configuration: createViewConfiguration(), window: window);
  renderView.prepareInitialFrame();
}


// RendererBinding.createViewConfiguration()
ViewConfiguration createViewConfiguration() {
  final double devicePixelRatio = window.devicePixelRatio;
  return ViewConfiguration(
    size: window.physicalSize / devicePixelRatio,
    devicePixelRatio: devicePixelRatio,
  );
}


// RenderView.prepareInitialFrame()
// https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/rendering/view.dart#L118
void prepareInitialFrame() {
  // ...
  scheduleInitialLayout();
  scheduleInitialPaint(_updateMatricesAndCreateNewRootLayer());
  // ...
}


// RenderObject.scheduleInitialLayout()
void scheduleInitialLayout() {
  // ...
  _relayoutBoundary = this;
  // ...
  owner!._nodesNeedingLayout.add(this);
}

TransformLayer _updateMatricesAndCreateNewRootLayer() {
  _rootTransform = configuration.toMatrix();
  final TransformLayer rootLayer = TransformLayer(transform: _rootTransform);
  rootLayer.attach(this);
  return rootLayer;
}


// ViewConfiguration.toMatrix()
// https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/rendering/view.dart#L37
// rootLayer 根据 devicePixelRatio 进行了一次缩放
Matrix4 toMatrix() {
  return Matrix4.diagonal3Values(devicePixelRatio, devicePixelRatio, 1.0);
}


// RenderObject.scheduleInitialPaint()
void scheduleInitialPaint(ContainerLayer rootLayer) {
  // ...
  _layer = rootLayer;
  // ...
  owner!._nodesNeedingPaint.add(this);
}

