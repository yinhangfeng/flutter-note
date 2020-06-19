flutter 1.10.1

## setState

```
:State.setState
  _element:Element.markNeedsBuild
    _dirty = true

    // https://github.com/flutter/flutter/blob/v1.10.1/packages/flutter/lib/src/widgets/framework.dart#L2183
    owner:BuildOwner.scheduleBuildFor(this)
    // (Element element)
      if (!_scheduledFlushDirtyElements && onBuildScheduled != null) {
        _scheduledFlushDirtyElements = true;
        onBuildScheduled();
          // WidgetsBinding.initInstances 设置了 buildOwner.onBuildScheduled = WidgetsBinding._handleBuildScheduled
          :WidgetsBinding._handleBuildScheduled()
            ensureVisualUpdate()
              // 跳转 ensureVisualUpdate
      }
      _dirtyElements.add(element)
      element._inDirtyList = true
```

## Element.rebuild

```
:Element.rebuild()
  if (!_active || !_dirty)
    return;
  performRebuild()
  // >ComponentElement.performRebuild
  // https://github.com/flutter/flutter/blob/v1.10.0/packages/flutter/lib/src/widgets/framework.dart#L3927
    let built: Widget = build()
    >StatelessElement.build
      widget:StatelessWidget.build(this)
    >StatefulElement.build
      state:State.build(this)
    _dirty = false

    // built 为 _child 新的 _widget
    _child: Element = updateChild(_child, built, slot)

  // >StatefulElement.performRebuild
    if (_didChangeDependencies) {
      _state.didChangeDependencies();
      _didChangeDependencies = false;
    }
    super.performRebuild();

  // >RenderObjectElement.performRebuild
    widget.updateRenderObject(this, renderObject)
    _dirty = false
```

## Element.updateChild

```
// Element updateChild(Element child, Widget newWidget, dynamic newSlot)
// https://github.com/flutter/flutter/blob/v1.10.0/packages/flutter/lib/src/widgets/framework.dart#L2864
  if (newWidget == null) {
    if (child != null)
      deactivateChild(child);
    return null;
  }
  if (child != null) {
    if (child.widget == newWidget) {
      if (child.slot != newSlot)
        updateSlotForChild(child, newSlot);
      return child;
    }
    if (Widget.canUpdate(child.widget, newWidget)) {
      if (child.slot != newSlot)
        updateSlotForChild(child, newSlot);
      child.update(newWidget);
      return child;
    }
    deactivateChild(child);
  }
  return inflateWidget(newWidget, newSlot);

  /// deactivateChild(Element child)
    child._parent = null
      child.detachRenderObject()
        visitChildren((child) {
          child.detachRenderObject()
        })
        _slot = null
        // >RenderObjectElement.detachRenderObject
          if (_ancestorRenderObjectElement != null) {
            _ancestorRenderObjectElement.removeChildRenderObject(renderObject)
            _ancestorRenderObjectElement = null
          }
          _slot = null;
      owner._inactiveElements.add(child)
        // _InactiveElements.add
        if (element._active)
          _deactivateRecursively(element); // 父 element 先于子 element deactivate
            element.deactivate()
            // Element.deactivate
              for (final InheritedElement dependency in _dependencies)
                dependency._dependents.remove(this)
              _inheritedWidgets = null;
              _active = false;
            // >StatefulElement.deactivate
              _state.deactivate();
              super.deactivate();
        _elements.add(element)

  /// updateSlotForChild(Element child, dynamic newSlot)
    void visit(Element element) {
      element._updateSlot(newSlot);
        _slot = newSlot
        
        // >RenderObjectElement._updateSlot
          super._updateSlot(newSlot);
          _ancestorRenderObjectElement.moveChildRenderObject(renderObject, slot);
      if (element is! RenderObjectElement)
        element.visitChildren(visit);
    }
    visit(child);

  /// inflateWidget(Widget newWidget, dynamic newSlot): Element
    // newWidget.key 为 GlobalKey 的话 执行复用的相关逻辑
      newChild: Element = _retakeInactiveElement(key, newWidget)
      newChild._activateWithParent(this, newSlot)
      updatedChild: Element = updateChild(newChild, newWidget, newSlot)
      updatedChild

    let newChild: Element = newWidget.createElement()
      // Element constructor
        newChild._widget 为 newWidget
      // StatefulElement constructor
        _state = widget.createState()
        _state._element = this
        _state._widget = widget
    newChild.mount(this, newSlot)
    // Element.mount(Element parent, dynamic newSlot)
      parent
      slot
      depth
      active = true
      owner
      // key 为 GlobalKey 的话 key._register(this);
      _updateInheritance()
        _inheritedWidgets = _parent?._inheritedWidgets;
        // InheritedElement 覆盖了 Element 的实现
          // ...
      
      // >ComponentElement.mount
        super.mount()
        _firstBuild()
          rebuild()
          // >StatefulElement._firstBuild
            _state.initState()
            _state.didChangeDependencies()
            super._firstBuild()

      // >RenderObjectElement.mount
        super.mount()
        _renderObject = widget.createRenderObject(this)
        attachRenderObject(newSlot)
          _slot = newSlot
          _ancestorRenderObjectElement = _findAncestorRenderObjectElement()
            // 查找最近的 RenderObjectElement 祖先
          _ancestorRenderObjectElement?.insertChildRenderObject(renderObject, newSlot)
          // (RenderObject child, dynamic slot)
            // >SingleChildRenderObjectElement.insertChildRenderObject(RenderObject child, dynamic slot)
              renderObject.child = child
                // 跳转 RenderObjectWithChildMixin.child
            // >MultiChildRenderObjectElement.insertChildRenderObject(RenderObject child, Element slot)
              renderObject.insert(child, after: slot?.renderObject);
              // ContainerRenderObjectMixin.insert(ChildType child, { ChildType after })
                adoptChild(child)
                  // RenderObjectWithChildMixin.child adoptChild
                _insertIntoChildList(child, after: after)
                  final ParentDataType childParentData = child.parentData;
                  _childCount += 1;
                  if (after == null) {
                    // insert at the start (_firstChild)
                    childParentData.nextSibling = _firstChild;
                    if (_firstChild != null) {
                      final ParentDataType _firstChildParentData = _firstChild.parentData;
                      _firstChildParentData.previousSibling = child;
                    }
                    _firstChild = child;
                    _lastChild ??= child;
                  } else {
                    final ParentDataType afterParentData = after.parentData;
                    if (afterParentData.nextSibling == null) {
                      childParentData.previousSibling = after;
                      afterParentData.nextSibling = child;
                      _lastChild = child;
                    } else {
                      childParentData.nextSibling = afterParentData.nextSibling;
                      childParentData.previousSibling = after;
                      final ParentDataType childPreviousSiblingParentData = childParentData.previousSibling.parentData;
                      final ParentDataType childNextSiblingParentData = childParentData.nextSibling.parentData;
                      childPreviousSiblingParentData.nextSibling = child;
                      childNextSiblingParentData.previousSibling = child;
                    }
                  }
            // >RenderObjectToWidgetElement.insertChildRenderObject(RenderObject child, dynamic slot)
              assert(slot == _rootChildSlot)
              renderObject.child = child
                // 跳转 RenderObjectWithChildMixin.child
          final ParentDataElement<RenderObjectWidget> parentDataElement = _findAncestorParentDataElement()
          if (parentDataElement != null)
            _updateParentData(parentDataElement.widget)
        _dirty = false
        // >SingleChildRenderObjectElement.mount
          super.mount
          child = updateChild(_child, widget.child, null);
        // >MultiChildRenderObjectElement.mount
          _children = List<Element>(widget.children.length);
          Element previousChild;
          for (int i = 0; i < _children.length; i += 1) {
            final Element newChild = inflateWidget(widget.children[i], IndexedSlot<Element>(i, previousChild));
            _children[i] = newChild;
            previousChild = newChild;
          }
    newChild

  /// update
  Element.update(Widget newWidget)
    _widget = newWidget
    // >StatelessElement.update
      super.update()
      _dirty = true
      rebuild()
    // >StatefulElement.update
      super.update()
      _dirty = true
      _state._widget = widget
      _state.didUpdateWidget(oldWidget)
      rebuild()
    // >RenderObjectElement.update
      super.update()
      widget.updateRenderObject(this, renderObject)
        // 一般会修改 renderObject 的一些属性
        // https://github.com/flutter/flutter/blob/v1.10.0/packages/flutter/lib/src/material/radio.dart#L240
      _dirty = false
      // >SingleChildRenderObjectElement.update
        super.update()
        _child = updateChild(_child, widget.child, null)

      // >MultiChildRenderObjectElement.update
        super.update()
        _children = updateChildren(_children, widget.children, forgottenChildren: _forgottenChildren);
        _forgottenChildren.clear();
```

## RenderObjectElement.updateChildren
```
// List<Element> updateChildren(List<Element> oldChildren, List<Widget> newWidgets, { Set<Element> forgottenChildren })
1. 从前往后遍历新旧列表，执行 updateChild 直到 Widget.canUpdate 为 false
2. 从后往前遍历新旧列表，更新 newChildrenBottom oldChildrenBottom 直到 Widget.canUpdate 为 false
3. 遍历旧列表中间部分，将有 key 的 child 放入 oldKeyedChildren 没有的执行 deactivateChild
4. 遍历新列表中间部分，从 oldKeyedChildren 获取 oldChild，执行 updateChild
5. 遍历新旧列表步骤2 中的部分，执行 updateChild
6. 对 oldKeyedChildren 中剩余的部分执行 deactivateChild
```

## RenderObjectWithChildMixin.child

```
:RenderObjectWithChildMixin.child // set child(ChildType value)
  if (_child != null)
    dropChild(_child)
      // AbstractNode.dropChild(RenderObject child)
      child._parent = null
      if (attached)
        child.detach()
          _owner = null
      
        // >RenderObject.dropChild(RenderObject child)
          child._cleanRelayoutBoundary()
          child.parentData.detach()
          child.parentData = null
          super.dropChild(child)
          markNeedsLayout()
          markNeedsCompositingBitsUpdate()
          markNeedsSemanticsUpdate()
  _child = value
  if (_child != null)
    adoptChild(_child)
      // AbstractNode.adoptChild(covariant AbstractNode child)
      child._parent = this
      if (attached)
        child.attach(_owner)
      redepthChild(child)

      // >RenderObject.adoptChild(RenderObject child)
        setupParentData(child)
        markNeedsLayout()
        markNeedsCompositingBitsUpdate()
        markNeedsSemanticsUpdate()
        super.adoptChild(child)
```

## RenderObject.attach

```
:AbstractNode.attach(covariant Object owner)
  _owner = owner

  // >RenderObject.attach(PipelineOwner owner)
    super.attach(owner)
    if (_needsLayout && _relayoutBoundary != null) {
      _needsLayout = false;
      markNeedsLayout();
    }
    if (_needsCompositingBitsUpdate) {
      _needsCompositingBitsUpdate = false;
      markNeedsCompositingBitsUpdate();
    }
    if (_needsPaint && _layer != null) {
      _needsPaint = false;
      markNeedsPaint();
    }
    if (_needsSemanticsUpdate && _semanticsConfiguration.isSemanticBoundary) {
      _needsSemanticsUpdate = false;
      markNeedsSemanticsUpdate();
    }

    // >RenderObjectWithChildMixin.attach
      super.attach(owner)
      // if _child != null
      _child.attach(owner)
```

## RenderView

```
:RendererBinding.initInstances()
  initRenderView()
    renderView = RenderView(configuration: createViewConfiguration(), window: window)
    // set renderView(RenderView value)
      _pipelineOwner.rootNode = value
    renderView.prepareInitialFrame()
      _rootNode?.detach()
      _rootNode = value
      _rootNode?.attach(this)
        // AbstractNode.attach
```

## ensureVisualUpdate

```
:SchedulerBinding.ensureVisualUpdate()
  // switch(schedulerPhase) ...
  scheduleFrame()
    // https://github.com/flutter/engine/blob/4243324a035d95f5a880b7f4c00ea18fe58ed5fa/lib/ui/window.dart#L1021
    window.scheduleFrame()
    _hasScheduledFrame = true
```

## markNeedsLayout

```
:RenderObject.markNeedsLayout()
  if (_relayoutBoundary != this) {
    markParentNeedsLayout()
      _needsLayout = true
      if (!_doingThisLayoutWithCallback) {
        parent.markNeedsLayout()
      }
  } else {
    _needsLayout = true
    if (owner != null) {
      owner._nodesNeedingLayout.add(this)
      owner.requestVisualUpdate()
        // 跳转 PipelineOwner.requestVisualUpdate
    }
  }

  // >RenderBox.markNeedsLayout
    if ((_cachedBaselines != null && _cachedBaselines.isNotEmpty) ||
        (_cachedIntrinsicDimensions != null && _cachedIntrinsicDimensions.isNotEmpty)) {
      _cachedBaselines?.clear();
      _cachedIntrinsicDimensions?.clear();
      if (parent is RenderObject) {
        markParentNeedsLayout();
        return;
      }
    }
    super.markNeedsLayout()
```

## markNeedsPaint

```
// https://github.com/flutter/flutter/blob/v1.10.1/packages/flutter/lib/src/rendering/object.dart#L2037
:RenderObject.markNeedsPaint
  _needsPaint = true
  if (isRepaintBoundary) {
    if (owner != null) {
      owner._nodesNeedingPaint.add(this);
      owner.requestVisualUpdate();
    }
  } else if (parent is RenderObject) {
    final RenderObject parent = this.parent;
    parent.markNeedsPaint();
  } else {
    if (owner != null)
      owner.requestVisualUpdate();
  }
  // PipelineOwner.requestVisualUpdate
```

## PipelineOwner.requestVisualUpdate

```
// PipelineOwner.requestVisualUpdate
// https://github.com/flutter/flutter/blob/v1.10.1/packages/flutter/lib/src/rendering/object.dart#L794
  onNeedVisualUpdate()
    // RendererBinding.initInstances create PipelineOwner
    // PipelineOwner.onNeedVisualUpdate = RendererBinding.ensureVisualUpdate
    // https://github.com/flutter/flutter/blob/v1.10.1/packages/flutter/lib/src/rendering/binding.dart#L32
    // https://github.com/flutter/flutter/blob/v1.10.1/packages/flutter/lib/src/scheduler/binding.dart#L676
    >SchedulerBinding.ensureVisualUpdate()
      // 跳转 ensureVisualUpdate
```

## onDrawFrame

由 native 调用

```
:Window.onDrawFrame()
  // 在 SchedulerBinding.ensureFrameCallbacksRegistered 中 Window.onDrawFrame = SchedulerBinding._handleDrawFrame
  // https://github.com/flutter/flutter/blob/v1.10.1/packages/flutter/lib/src/scheduler/binding.dart#L658
  :SchedulerBinding._handleDrawFrame()
    handleDrawFrame()
      // TODO 分析具体过程
      Timeline.finishSync()

      _schedulerPhase = SchedulerPhase.persistentCallbacks;
      for (FrameCallback callback in _persistentCallbacks)
        _invokeFrameCallback(callback, _currentFrameTimeStamp);

      // POST-FRAME CALLBACKS
      _schedulerPhase = SchedulerPhase.postFrameCallbacks;
      final List<FrameCallback> localPostFrameCallbacks =
          List<FrameCallback>.from(_postFrameCallbacks);
      _postFrameCallbacks.clear();
      for (FrameCallback callback in localPostFrameCallbacks)
        _invokeFrameCallback(callback, _currentFrameTimeStamp)

      _schedulerPhase = SchedulerPhase.idle
      Timeline.finishSync()
      _currentFrameTimeStamp = null

      // _persistentCallbacks 的其中一个 RendererBinding._handlePersistentFrameCallback
        :RendererBinding._handlePersistentFrameCallback()
          drawFrame()
            // 跳转 drawFrame
```

## drawFrame

```
:WidgetsBinding.drawFrame()

  // https://github.com/flutter/flutter/blob/v1.10.1/packages/flutter/lib/src/widgets/framework.dart#L2294
  buildOwner:BuildOwner.buildScope(renderViewElement)
    // TODO 分析具体过程
    // foreach _dirtyElements
    :Element.rebuild()
  super:RendererBinding.drawFrame()
    pipelineOwner.flushLayout();
      // 跳转 flushLayout
    pipelineOwner.flushCompositingBits();
    pipelineOwner.flushPaint();
      // 跳转 flushPaint
    renderView.compositeFrame(); // this sends the bits to the GPU
    pipelineOwner.flushSemantics(); // this also sends the semantics to the OS.

    // > WidgetsBinding.drawFrame
  buildOwner.finalizeTree()
    lockState(() {
      _inactiveElements._unmountAll(); // this unregisters the GlobalKeys
        // element.visitChildren children 先 unmount
        element.unmount()
        // Element.unmount()
          if (key is GlobalKey) {
            key._unregister(this);
          }
        // StatefulElement.unmount()
          super.unmount()
          _state.dispose();
          _state._element = null;
          _state = null;
        // RenderObject.unmount()
          super.unmount()
          widget.didUnmountRenderObject(renderObject);
    });
```

## flushLayout

```
:PipelineOwner.flushLayout()
  while (_nodesNeedingLayout.isNotEmpty) {
    final List<RenderObject> dirtyNodes = _nodesNeedingLayout;
    _nodesNeedingLayout = <RenderObject>[];
    for (RenderObject node in dirtyNodes..sort((RenderObject a, RenderObject b) => a.depth - b.depth)) {
      if (node._needsLayout && node.owner == this)
        node._layoutWithoutResize();
        // RenderObject._layoutWithoutResize()
          performLayout()
          markNeedsSemanticsUpdate()
          _needsLayout = false
          markNeedsPaint()
    }
  }
```

## layout

```
:RenderObject.layout(Constraints constraints, { bool parentUsesSize = false })
  RenderObject relayoutBoundary;
  if (!parentUsesSize || sizedByParent || constraints.isTight || parent is! RenderObject) {
    relayoutBoundary = this;
  } else {
    final RenderObject parent = this.parent;
    relayoutBoundary = parent._relayoutBoundary;
  }
  if (!_needsLayout && constraints == _constraints && relayoutBoundary == _relayoutBoundary) {
    return;
  }
  _constraints = constraints;
  _relayoutBoundary = relayoutBoundary;

  if (sizedByParent) {
    performResize();
  }
  performLayout();
  markNeedsSemanticsUpdate();
  _needsLayout = false;
  markNeedsPaint();
```

## flushPaint

```
:PipelineOwner.flushPaint()
  final List<RenderObject> dirtyNodes = _nodesNeedingPaint;
  _nodesNeedingPaint = <RenderObject>[];
  // Sort the dirty nodes in reverse order (deepest first).
  for (RenderObject node in dirtyNodes..sort((RenderObject a, RenderObject b) => b.depth - a.depth)) {
    if (node._needsPaint && node.owner == this) {
      if (node._layer.attached) {
        PaintingContext.repaintCompositedChild(node);
          PaintingContext._repaintCompositedChild(child)
            OffsetLayer childLayer = child._layer;
            if (childLayer == null) {
              child._layer = childLayer = OffsetLayer();
            } else {
              childLayer.removeAllChildren();
            }
            childContext ??= PaintingContext(child._layer, child.paintBounds);
            child._paintWithContext(childContext, Offset.zero);
              if (_needsLayout)
                return;
              _needsPaint = false;
              paint(context, offset);
            childContext.stopRecordingIfNeeded();
      } else {
        node._skippedPaintingOnLayer();
      }
    }
  }
```
