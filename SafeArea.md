# SafeArea (flutter 1.10.1)

https://github.com/flutter/flutter/blob/v1.10.1/packages/flutter/lib/src/widgets/safe_area.dart

通过 context MediaQuery 获取 window insets 数据

https://github.com/flutter/flutter/blob/v1.10.1/packages/flutter/lib/src/widgets/safe_area.dart#L196

MediaQueryData.fromWindow 构造 MediaQueryData context

https://github.com/flutter/flutter/blob/v1.10.1/packages/flutter/lib/src/widgets/app.dart#L1200
https://github.com/flutter/flutter/blob/v1.10.1/packages/flutter/lib/src/widgets/media_query.dart#L113

## MediaQueryData 更新过程

Android

https://github.com/flutter/engine/blob/master/shell/platform/android/io/flutter/view/FlutterView.java#L754
```
:FlutterView.updateViewportMetrics()
  mNativeView.getFlutterJNI().setViewportMetrics()
```

cpp

https://github.com/flutter/engine/blob/master/runtime/runtime_controller.cc#L136
```
:RuntimeController.SetViewportMetrics()
  window->UpdateWindowMetrics(metrics)
```

dart

https://github.com/flutter/engine/blob/master/lib/ui/hooks.dart#L31
```
_updateWindowMetrics()
  _invoke(window.onMetricsChanged, window._onMetricsChangedZone)
```

```
:Window._onMetricsChanged()
  // 在 RendererBinding.initInstances window.onMetricsChanged = RendererBinding.handleMetricsChanged
  :RendererBinding.handleMetricsChanged()
    renderView.configuration = createViewConfiguration()
      scheduleForcedFrame()
        window.scheduleFrame()
        _hasScheduledFrame = true

        // > WidgetsBinding.handleMetricsChanged
          super.handleMetricsChanged()
          // https://github.com/flutter/flutter/blob/v1.10.1/packages/flutter/lib/src/widgets/binding.dart#L425
          for (WidgetsBindingObserver observer in _observers)
            observer.didChangeMetrics()
```

WidgetsApp

https://github.com/flutter/flutter/blob/v1.10.1/packages/flutter/lib/src/widgets/app.dart#L717

```
:_WidgetsAppState.initState()
  // https://github.com/flutter/flutter/blob/v1.10.1/packages/flutter/lib/src/widgets/binding.dart#L410
  WidgetsBinding.instance.addObserver(this)
    _observers.add(observer)
```

_WidgetsAppState.didChangeMetrics 更新 MediaQueryData

https://github.com/flutter/flutter/blob/v1.10.1/packages/flutter/lib/src/widgets/app.dart#L1011

```
:_WidgetsAppState.didChangeMetrics()
  setState()
```
