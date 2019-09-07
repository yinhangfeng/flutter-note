## setState

```
State.setState
  Element.markNeedsBuild
    BuildOwner.scheduleBuildFor(element)
      _dirtyElements.add(element)
```

## build

```
:WidgetsBinding.drawFrame()
  buildOwner:BuildOwner.buildScope(renderViewElement)
    // foreach _dirtyElements
    :Element.rebuild()
      performRebuild()
```

ComponentElement.performRebuild

```
:ComponentElement.performRebuild
  let built: Widget = build()
  >StatelessElement.build
    widget:StatelessWidget.build
  >StatefulElement.build
    state:State.build

  _child: Element = updateChild(_child, built, slot)
  // (Element child, Widget newWidget, dynamic newSlot)
    case: inflateWidget
      inflateWidget(newWidget, newSlot)
        let newChild: Element = newWidget.createElement()
        newChild.mount(this, newSlot)
        // (Element parent, dynamic newSlot)
          // set parent slot depth owner
          parent
          slot
          depth
          owner
          active = true
        newChild

    case: deactivateChild // newWidget == null || !Widget.canUpdate()
      deactivateChild(child)
        child._parent = null
        child.detachRenderObject()
        owner._inactiveElements.add(child)
    
    case: updateSlotForChild
      updateSlotForChild(child, newSlot)
      // visit children _updateSlot(newSlot)

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

  Element.update
    widget = newWidget
  StatelessElement:
    Element.rebuild
  StatefulElement:
    state.widget = newWidget
    Element.rebuild

RenderObjectElement:
  RenderObjectElement.performRebuild
    widget.updateRenderObject
```