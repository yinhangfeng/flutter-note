flutter 1.10.1

## Widget

```
Widget
  key: Key
  createElement: Element

  < StatelessWidget
    createElement: StatelessElement
    build(BuildContext): Widget

  < StatefulWidget
    createElement: StatefulElement
    createState: State

  < RenderObjectWidget
    createElement: RenderObjectElement
    createRenderObject(BuildContext context): RenderObject
    updateRenderObject(BuildContext context, covariant RenderObject renderObject)
    didUnmountRenderObject(covariant RenderObject renderObject)

    < LeafRenderObjectWidget

    < SingleChildRenderObjectWidget
      child: Widget

    < MultiChildRenderObjectWidget
      children: List<Widget>

  < ProxyWidget
    child: Widget

    // InheritedWidget 可以向下级组件传递数据
    // BuildContext.inheritFromWidgetOfExactType(XXX extends InheritedWidget)
    < InheritedWidget
      createElement: InheritedElement
      updateShouldNotify(InheritedWidget): bool

    < ParentDataWidget<T extends RenderObjectWidget>
```

## BuildOwner

```
BuildOwner
  scheduleBuildFor(Element)
  lockState(callback())
  buildScope(context: Element, VoidCallback)
  finalizeTree()
  reassemble(root: Element)
```

## Element

```
BuildContext
  widget: Widget
  // The [BuildOwner] for this context. The [BuildOwner] is in charge of
  // managing the rendering pipeline for this context.
  owner: BuildOwner
  findRenderObject(): RenderObject
  size: Size

  < Element
    _parent: Element
    slot: dynamic
    // An integer that is guaranteed to be greater than the parent's, if any
    depth: int
    _active: bool
    reassemble()
    get renderObject()
    visitChildren(ElementVisitor)
    updateChild(Element child, Widget newWidget, dynamic newSlot): Element
    mount(Element parent, dynamic newSlot)
    update(covariant Widget newWidget)
    updateSlotForChild(Element child, dynamic newSlot)
    _updateSlot(dynamic newSlot)
    _updateDepth(int parentDepth)
    detachRenderObject()
    attachRenderObject(dynamic newSlot)
    inflateWidget(Widget newWidget, dynamic newSlot): Element
    deactivateChild(Element child)
    forgetChild(Element child)
    _activateWithParent(Element parent, dynamic newSlot)
    activate()
    deactivate()
    unmount()
    _inheritedWidgets: Map<Type, InheritedElement>
    _dependencies: Set<InheritedElement>
    _updateInheritance()
    didChangeDependencies()
    dirty: bool
    _inDirtyList: bool
    markNeedsBuild()
    rebuild()
    performRebuild()

    < RenderObjectElement
      _ancestorRenderObjectElement: RenderObjectElement
      _findAncestorRenderObjectElement(): RenderObjectElement
      _findAncestorParentDataElement(): ParentDataElement<RenderObjectWidget>
      updateChildren(List<Element> oldChildren, List<Widget> newWidgets, { Set<Element> forgottenChildren }): List<Element>
      insertChildRenderObject(covariant RenderObject child, covariant dynamic slot)
      moveChildRenderObject(covariant RenderObject child, covariant dynamic slot)
      removeChildRenderObject(covariant RenderObject child)

      < LeafRenderObjectElement
        // 没有 child 的 RenderObjectElement

      < SingleChildRenderObjectElement
        _child: Element

      < MultiChildRenderObjectElement
        visitChildren(ElementVisitor visitor)

      < RootRenderObjectElement
        assignOwner(BuildOwner owner)

        < RenderObjectToWidgetElement

    < ComponentElement
      _child: Element
      _firstBuild()
      build(): Widget

      < StatelessElement

      < StatefulElement
        state: State<StatefulWidget>

      < ProxyElement
        updated(covariant ProxyWidget oldWidget)
        notifyClients(covariant ProxyWidget oldWidget)

        < ParentDataElement<T extends RenderObjectWidget>

        < InheritedElement
          _dependents: Map<Element, Object>
          getDependencies(Element dependent): Object
          setDependencies(Element dependent, Object value): Object
          updateDependencies(Element dependent, Object aspect)
          notifyDependent(covariant InheritedWidget oldWidget, Element dependent)
```

## RenderObject

```
AbstractNode
  depth: int
  redepthChild(AbstractNode child)
  redepthChildren()
  owner: Object
  attached: bool
  attach(covariant Object owner)
  detach()
  parent: AbstractNode
  adoptChild(covariant AbstractNode child)
  dropChild(covariant AbstractNode child)

  < RenderObject
    reassemble()
    parentData: ParentData
    setupParentData(covariant RenderObject child)
    visitChildren(RenderObjectVisitor visitor)
    _needsLayout: bool
    _relayoutBoundary: RenderObject
    _doingThisLayoutWithCallback: bool
    constraints: Constraints
    markNeedsLayout()
    markParentNeedsLayout()
    markNeedsLayoutForSizedByParentChange()
    _cleanRelayoutBoundary()
    scheduleInitialLayout()
    _layoutWithoutResize()
    layout(Constraints constraints, { bool parentUsesSize = false })
    get sizedByParent: bool
    performResize()
    performLayout()
    invokeLayoutCallback(LayoutCallback<T> callback)
    rotate()
    get isRepaintBoundary: bool
    get alwaysNeedsCompositing: bool
    get layer: ContainerLayer
    set layer(ContainerLayer newLayer)
    _needsCompositingBitsUpdate: bool
    markNeedsCompositingBitsUpdate()
    get needsCompositing: bool
    _updateCompositingBits()
    _needsPaint: bool
    markNeedsPaint()
    _paintWithContext(PaintingContext context, Offset offset)
    get paintBounds: Rect
    paint(PaintingContext context, Offset offset)
    applyPaintTransform(covariant RenderObject child, Matrix4 transform)
    getTransformTo(RenderObject ancestor): Matrix4
    describeApproximatePaintClip(covariant RenderObject child): Rect
    describeSemanticsClip(covariant RenderObject child): Rect
    scheduleInitialSemantics()
    handleEvent(PointerEvent event, covariant HitTestEntry entry)
    showOnScreen({
      RenderObject descendant,
      Rect rect,
      Duration duration,
      Curve curve,
    })

    // The root of the render tree
    < RenderView

    < RenderBox
      _cachedIntrinsicDimensions: Map<_IntrinsicDimensionsCacheEntry, double>
      _computeIntrinsicDimension(_IntrinsicDimension dimension, double argument, double computer(double argument)): double
      getMinIntrinsicWidth(double height): double
      computeMinIntrinsicWidth(double height): double
      getMaxIntrinsicWidth(double height): double
      computeMaxIntrinsicWidth(double height): double
      // ... height
      hasSize: bool
      size: Size
      // ...
      hitTest(BoxHitTestResult result, { @required Offset position }): bool
      hitTestSelf(Offset position): bool
      hitTestChildren(BoxHitTestResult result, { Offset position }): bool
      globalToLocal(Offset point, { RenderObject ancestor }): Offset
      localToGlobal(Offset point, { RenderObject ancestor }): Offset

      < RenderProxyBox
        // 与 RenderBox 功能相同 所有方法调用转移到 child
        // 来自 RenderObjectWithChildMixin
        child: RenderBox

        < RenderConstrainedBox
          additionalConstraints: BoxConstraints
          // 主要重写了 computeMinIntrinsicWidth computeMaxIntrinsicWidth computeMinIntrinsicHeight computeMaxIntrinsicHeight performLayout
          // 实现约束之后在调用 child

```

## Window

```
Window
  // ... MediaQueryData 相关数据 如 devicePixelRatio viewInsets physicalDepth

  // 由 native 调用 MediaQueryData 相关
  _onMetricsChanged()

  // 请求 native 在接下来的适当时候调用 onBeginFrame onDrawFrame
  scheduleFrame()

  get set onBeginFrame()
  get set onDrawFrame()
```

## RendererBinding

```
RendererBinding
  // https://github.com/flutter/flutter/blob/v1.10.1/packages/flutter/lib/src/rendering/binding.dart#L28
  initInstances()

  < WidgetsBinding
    // https://github.com/flutter/flutter/blob/v1.10.1/packages/flutter/lib/src/widgets/binding.dart#L251
    initInstances()
```
