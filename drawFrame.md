## darwFrame

```dart
// native 调用 _beginFrame
// https://github.com/flutter/engine/blob/master/lib/ui/hooks.dart#L113
@pragma('vm:entry-point')
// ignore: unused_element
void _beginFrame(int microseconds) {
  PlatformDispatcher.instance._beginFrame(microseconds);
}

@pragma('vm:entry-point')
// ignore: unused_element
void _drawFrame() {
  PlatformDispatcher.instance._drawFrame();
}


// PlatformDispatcher._beginFrame()
// https://github.com/flutter/engine/blob/master/lib/ui/platform_dispatcher.dart#L236
void _beginFrame(int microseconds) {
  _invoke1<Duration>(
    onBeginFrame,
    _onBeginFrameZone,
    Duration(microseconds: microseconds),
  );
}

void _drawFrame() {
  _invoke(onDrawFrame, _onDrawFrameZone);
}


// window.onBeginFrame
// https://github.com/flutter/engine/blob/master/lib/ui/window.dart#L488
FrameCallback? get onBeginFrame => platformDispatcher.onBeginFrame;
set onBeginFrame(FrameCallback? callback) {
  platformDispatcher.onBeginFrame = callback;
}

VoidCallback? get onDrawFrame => platformDispatcher.onDrawFrame;
set onDrawFrame(VoidCallback? callback) {
  platformDispatcher.onDrawFrame = callback;
}


// SchedulerBinding.ensureFrameCallbacksRegistered()
// https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/scheduler/binding.dart#L726
void ensureFrameCallbacksRegistered() {
  window.onBeginFrame ??= _handleBeginFrame;
  window.onDrawFrame ??= _handleDrawFrame;
}


// SchedulerBinding._handleBeginFrame()
void _handleBeginFrame(Duration rawTimeStamp) {
  if (_warmUpFrame) {
    _rescheduleAfterWarmUpFrame = true;
    return;
  }
  handleBeginFrame(rawTimeStamp);
}

void _handleDrawFrame() {
  if (_rescheduleAfterWarmUpFrame) {
    _rescheduleAfterWarmUpFrame = false;
    addPostFrameCallback((Duration timeStamp) {
      _hasScheduledFrame = false;
      scheduleFrame();
    });
    return;
  }
  handleDrawFrame();
}


// SchedulerBinding.handleBeginFrame()
// 执行 _transientCallbacks，主要用于驱动动画， 在 SchedulerBinding.scheduleFrameCallback() 里注册的
// https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/scheduler/binding.dart#L1075
void handleBeginFrame(Duration? rawTimeStamp) {
  _firstRawTimeStampInEpoch ??= rawTimeStamp;
  _currentFrameTimeStamp = _adjustForEpoch(rawTimeStamp ?? _lastRawTimeStamp);
  if (rawTimeStamp != null)
    _lastRawTimeStamp = rawTimeStamp;

  _hasScheduledFrame = false;
  try {
    // TRANSIENT FRAME CALLBACKS
    _schedulerPhase = SchedulerPhase.transientCallbacks;
    final Map<int, _FrameCallbackEntry> callbacks = _transientCallbacks;
    _transientCallbacks = <int, _FrameCallbackEntry>{};
    callbacks.forEach((int id, _FrameCallbackEntry callbackEntry) {
      if (!_removedIds.contains(id))
        _invokeFrameCallback(callbackEntry.callback, _currentFrameTimeStamp!, callbackEntry.debugStack);
    });
    _removedIds.clear();
  } finally {
    _schedulerPhase = SchedulerPhase.midFrameMicrotasks;
  }
}

// SchedulerBinding.handleDrawFrame()
// 主要是执行 persistentCallbacks 和 postFrameCallbacks
// 最主要的是 persistentCallbacks，在 SchedulerBinding.addPersistentFrameCallback() 注册，内部完成了 build layout paint 等的调用
void handleDrawFrame() {
  try {
    // PERSISTENT FRAME CALLBACKS
    _schedulerPhase = SchedulerPhase.persistentCallbacks;
    for (final FrameCallback callback in _persistentCallbacks)
      _invokeFrameCallback(callback, _currentFrameTimeStamp!);

    // POST-FRAME CALLBACKS
    _schedulerPhase = SchedulerPhase.postFrameCallbacks;
    final List<FrameCallback> localPostFrameCallbacks =
        List<FrameCallback>.from(_postFrameCallbacks);
    _postFrameCallbacks.clear();
    for (final FrameCallback callback in localPostFrameCallbacks)
      _invokeFrameCallback(callback, _currentFrameTimeStamp!);
  } finally {
    _schedulerPhase = SchedulerPhase.idle;
    _currentFrameTimeStamp = null;
  }
}


// RendererBinding._handlePersistentFrameCallback()
// RendererBinding.initInstances() 里面注册了 addPersistentFrameCallback(_handlePersistentFrameCallback);
// https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/rendering/binding.dart#L327
void _handlePersistentFrameCallback(Duration timeStamp) {
  drawFrame();
  _scheduleMouseTrackerUpdate();
}


// WidgetsBinding.drawFrame()
// https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/widgets/binding.dart#L846
// 执行 build 流程，并调用 RendererBinding.drawFrame()
void drawFrame() {
  TimingsCallback? firstFrameCallback;
  if (_needToReportFirstFrame) {
    firstFrameCallback = (List<FrameTiming> timings) {
      SchedulerBinding.instance!.removeTimingsCallback(firstFrameCallback!);
      firstFrameCallback = null;
      _firstFrameCompleter.complete();
    };
    // Callback is only invoked when FlutterView.render is called. When
    // sendFramesToEngine is set to false during the frame, it will not be
    // called and we need to remove the callback (see below).
    SchedulerBinding.instance!.addTimingsCallback(firstFrameCallback!);
  }

  try {
    if (renderViewElement != null)
      // 执行 build 流程
      buildOwner!.buildScope(renderViewElement!);
    // super.drawFrame() 就是 RendererBinding.drawFrame()
    super.drawFrame();
    buildOwner!.finalizeTree();
  }
  _needToReportFirstFrame = false;
  if (firstFrameCallback != null && !sendFramesToEngine) {
    // This frame is deferred and not the first frame sent to the engine that
    // should be reported.
    _needToReportFirstFrame = true;
    SchedulerBinding.instance!.removeTimingsCallback(firstFrameCallback!);
  }
}


// RendererBinding.drawFrame()
// https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/rendering/binding.dart#L460
// 执行 layout paint semantics 流程
void drawFrame() {
  pipelineOwner.flushLayout();
  pipelineOwner.flushCompositingBits();
  pipelineOwner.flushPaint();
  if (sendFramesToEngine) {
    // 将 paint 内容提交给 engine 最终通过 skia 渲染
    renderView.compositeFrame(); // this sends the bits to the GPU
    pipelineOwner.flushSemantics(); // this also sends the semantics to the OS.
    _firstFrameSent = true;
  }
}
```


## markNeedsBuild
```
State.setState -> Element.markNeedsBuild -> BuildOwner.scheduleBuildFor()
-> BuildOwner.onBuildScheduled() -> WidgetsBinding._handleBuildScheduled() -> SchedulerBinding.ensureVisualUpdate()
```

## markNeedsLayout
```
RenderObject.markNeedsLayout() -> PipelineOwner.requestVisualUpdate() -> SchedulerBinding.ensureVisualUpdate()
```

## markNeedsPaint
```
RenderObject.markNeedsLayout() -> PipelineOwner.requestVisualUpdate() -> SchedulerBinding.ensureVisualUpdate()
```

## ensureVisualUpdate
```
SchedulerBinding.ensureVisualUpdate() -> SchedulerBinding.scheduleFrame() -> SingletonFlutterWindow.scheduleFrame() -> PlatformDispatcher.scheduleFrame()
```

