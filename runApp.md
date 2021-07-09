```dart
void runApp(Widget app) {
  WidgetsFlutterBinding.ensureInitialized()
    ..scheduleAttachRootWidget(app)
    ..scheduleWarmUpFrame();
}
```

## WidgetsFlutterBinding.ensureInitialized()
```dart
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
  // 注册每一帧的 darwFrame 回调
  addPersistentFrameCallback(_handlePersistentFrameCallback);
  // ...
}


// PaintingBinding.initInstances()
void initInstances() {
  // ...
  _instance = this;
  _imageCache = createImageCache();
  shaderWarmUp?.execute();
}


// RendererBinding.initRenderView()
// https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/rendering/binding.dart#L167
void initRenderView() {
  // ...
  renderView = RenderView(configuration: createViewConfiguration(), window: window);
  renderView.prepareInitialFrame();
}

// set RendererBinding.renderView
// RenderView 作为 root RenderObject 赋值给了 pipelineOwner.rootNode
set renderView(RenderView value) {
  _pipelineOwner.rootNode = value;
}

// set PipelineOwner.rootNode
set rootNode(AbstractNode? value) {
  if (_rootNode == value)
    return;
  _rootNode?.detach();
  _rootNode = value;
  // rootNode 的 owner 是 PipelineOwner
  _rootNode?.attach(this);
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
// 触发首次 layout 与 paint
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
  // RenderView 肯定是 relayoutBoundary
  _relayoutBoundary = this;
  // ...
  owner!._nodesNeedingLayout.add(this);
}

// RenderView._updateMatricesAndCreateNewRootLayer()
// 创建 RenderView 的 layer
TransformLayer _updateMatricesAndCreateNewRootLayer() {
  _rootTransform = configuration.toMatrix();
  final TransformLayer rootLayer = TransformLayer(transform: _rootTransform);
  // layer 的 owner 是 RenderView
  rootLayer.attach(this);
  return rootLayer;
}


// ViewConfiguration.toMatrix()
// https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/rendering/view.dart#L37
// rootLayer 根据 devicePixelRatio 进行了一次缩放
// 如果需要改为等比缩放模式可以考虑从这里入手处理，或者在 native 修改传入的 devicePixelRatio
Matrix4 toMatrix() {
  return Matrix4.diagonal3Values(devicePixelRatio, devicePixelRatio, 1.0);
}


// RenderObject.scheduleInitialPaint()
void scheduleInitialPaint(ContainerLayer rootLayer) {
  // ...
  // 设置 RenderView 的 layer
  _layer = rootLayer;
  // ...
  owner!._nodesNeedingPaint.add(this);
}

```

## WidgetsBinding.scheduleAttachRootWidget()
```dart
// WidgetsBinding.scheduleAttachRootWidget()
void scheduleAttachRootWidget(Widget rootWidget) {
  Timer.run(() {
    attachRootWidget(rootWidget);
  });
}


// WidgetsBinding.attachRootWidget()
// 创建 RenderView 对应的 Widget，将 runApp 传入的 Widget 作为 child，并创建 RenderView 对应的 Element (renderViewElement)
// RenderObjectToWidgetAdapter 是一个 RenderObjectWidget，创建时传入的 container 为 RenderView，其 createRenderObject 直接返回了 container
// https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/widgets/binding.dart#L930
void attachRootWidget(Widget rootWidget) {
  _readyToProduceFrames = true;
  _renderViewElement = RenderObjectToWidgetAdapter<RenderBox>(
    container: renderView,
    debugShortDescription: '[root]',
    child: rootWidget,
  ).attachToRenderTree(buildOwner!, renderViewElement as RenderObjectToWidgetElement<RenderBox>?);
}


// RenderObjectToWidgetAdapter.attachToRenderTree()
// https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/widgets/binding.dart#L1096
RenderObjectToWidgetElement<T> attachToRenderTree(BuildOwner owner, [ RenderObjectToWidgetElement<T>? element ]) {
  if (element == null) {
    owner.lockState(() {
      element = createElement();
      assert(element != null);
      element!.assignOwner(owner);
    });
    owner.buildScope(element!, () {
      element!.mount(null, null);
    });
    // This is most likely the first time the framework is ready to produce
    // a frame. Ensure that we are asked for one.
    SchedulerBinding.instance!.ensureVisualUpdate();
  } else {
    element._newWidget = this;
    element.markNeedsBuild();
  }
  return element!;
}
```

## SchedulerBinding.scheduleWarmUpFrame()
```dart
// SchedulerBinding.scheduleWarmUpFrame()
void scheduleWarmUpFrame() {
  if (_warmUpFrame || schedulerPhase != SchedulerPhase.idle)
    return;

  _warmUpFrame = true;
  Timeline.startSync('Warm-up frame');
  final bool hadScheduledFrame = _hasScheduledFrame;
  // We use timers here to ensure that microtasks flush in between.
  Timer.run(() {
    assert(_warmUpFrame);
    handleBeginFrame(null);
  });
  Timer.run(() {
    assert(_warmUpFrame);
    handleDrawFrame();
    // We call resetEpoch after this frame so that, in the hot reload case,
    // the very next frame pretends to have occurred immediately after this
    // warm-up frame. The warm-up frame's timestamp will typically be far in
    // the past (the time of the last real frame), so if we didn't reset the
    // epoch we would see a sudden jump from the old time in the warm-up frame
    // to the new time in the "real" frame. The biggest problem with this is
    // that implicit animations end up being triggered at the old time and
    // then skipping every frame and finishing in the new time.
    resetEpoch();
    _warmUpFrame = false;
    if (hadScheduledFrame)
      scheduleFrame();
  });

  // Lock events so touch events etc don't insert themselves until the
  // scheduled frame has finished.
  lockEvents(() async {
    await endOfFrame;
    Timeline.finishSync();
  });
}
```
