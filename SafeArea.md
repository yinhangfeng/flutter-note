# SafeArea (flutter 1.9.x)

https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/widgets/safe_area.dart

通过 context MediaQuery 获取 window insets 数据

https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/widgets/media_query.dart

MediaQueryData.fromWindow 构造 MediaQueryData context

https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/widgets/app.dart#L1198

## MediaQueryData 更新过程

Android

https://github.com/flutter/engine/blob/master/shell/platform/android/io/flutter/view/FlutterView.java#L663

cpp

https://github.com/flutter/engine/blob/master/runtime/runtime_controller.cc#L164

native 通过 _updateWindowMetrics 通知 dart vm

https://github.com/flutter/engine/blob/master/lib/ui/hooks.dart#L63

WidgetsBinding.instance.addObserver

https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/widgets/app.dart#L715

didChangeMetrics 更新 MediaQueryData

https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/widgets/app.dart#L1009
