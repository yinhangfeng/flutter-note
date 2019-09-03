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

  < ProxyWidget
    child: Widget

    // InheritedWidget 可以向下级组件传递数据
    // BuildContext.inheritFromWidgetOfExactType(XXX extends InheritedWidget)
    < InheritedWidget
      createElement: InheritedElement
      updateShouldNotify(InheritedWidget): bool
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
      TODO

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

## BuildOwner

```
BuildOwner
  scheduleBuildFor(Element)
  lockState(callback())
  buildScope(context: Element, VoidCallback)
  finalizeTree()
  reassemble(root: Element)
```

## 更新过程

```
State.setState
  Element.markNeedsBuild
    BuildOwner.scheduleBuildFor(element)
      _dirtyElements.add(element)


WidgetsBinding.drawFrame
  BuildOwner.buildScope
    Element.rebuild
      Element.performRebuild


ComponentElement:
  ComponentElement.performRebuild
  built = ComponentElement.build

  StatelessElement:
    StatelessElement.build
    StatelessWidget.build
  StatefulElement:
    StatefulElement.build
    State.build

  child = ComponentElement.updateChild(child, built, slot)
  deactivateChild || inflateWidget || updateSlotForChild || child.update

  Element.update
    widget = newWidget
  StatelessElement:
    Element.rebuild
  StatefulElement:
    state.widget = newWidget
    Element.rebuild

  Element.inflateWidget
    newChild = newWidget.createElement()
    newChild.mount

    Element:
    Element.mount
      parent
      slot
      owner
    ComponentElement:
    ComponentElement.mount
      ...
      _firstBuild
        rebuild
    RenderObjectElement:
    RenderObjectElement.mount
      widget.createRenderObject
      attachRenderObject

RenderObjectElement:
  RenderObjectElement.performRebuild
    widget.updateRenderObject
```
