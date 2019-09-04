## setState

```
State.setState
  Element.markNeedsBuild
    BuildOwner.scheduleBuildFor(element)
      _dirtyElements.add(element)
```

## build

```
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