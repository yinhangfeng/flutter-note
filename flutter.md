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
  findRenderObject: RenderObject
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

      < SingleChildRenderObjectElement
        _child: Element

      < MultiChildRenderObjectElement
        visitChildren(ElementVisitor visitor)

      < RootRenderObjectElement
        assignOwner(BuildOwner owner)

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
