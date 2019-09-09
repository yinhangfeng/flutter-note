## setState

```
:State.setState
  _element:Element.markNeedsBuild
    _dirty = true
    owner:BuildOwner.scheduleBuildFor(this)
    // (Element element)
      _dirtyElements.add(element)
      element._inDirtyList = true
```

## build

WidgetsBinding.drawFrame

```
:WidgetsBinding.drawFrame()
  buildOwner:BuildOwner.buildScope(renderViewElement)
    // foreach _dirtyElements
    :Element.rebuild()
```

Element.rebuild

```
:Element.rebuild()
  performRebuild()
```

ComponentElement.performRebuild

```
// https://github.com/flutter/flutter/blob/v1.10.0/packages/flutter/lib/src/widgets/framework.dart#L3927
:ComponentElement.performRebuild
  let built: Widget = build()
  >StatelessElement.build
    widget:StatelessWidget.build
  >StatefulElement.build
    state:State.build
  _dirty = false

  // built 为 _child 新的 _widget
  _child: Element = updateChild(_child, built, slot)
  // (Element child, Widget newWidget, dynamic newSlot)
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

    /// deactivateChild
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

    /// updateSlotForChild
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

    /// inflateWidget
      // key 为 GlobalKey 的话 执行复用的相关逻辑

      let newChild: Element = newWidget.createElement()
        // Element constructor
        // newChild._widget 为 newWidget
      newChild.mount(this, newSlot)
      // (Element parent, dynamic newSlot)
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

        // >RenderObjectElement.mount
          super.mount()
          _renderObject = widget.createRenderObject(this)
          attachRenderObject(newSlot)
            _slot = newSlot
            _ancestorRenderObjectElement = _findAncestorRenderObjectElement()
            _ancestorRenderObjectElement?.insertChildRenderObject(renderObject, newSlot)
            final ParentDataElement<RenderObjectWidget> parentDataElement = _findAncestorParentDataElement()
            if (parentDataElement != null)
              _updateParentData(parentDataElement.widget)
          _dirty = false
      newChild

    /// update
      _widget = newWidget
      // >StatelessElement.update
        super.update()
        _dirty = true
        rebuild()
      // >StatefulElement.update
        super.update()
        _dirty = true
        _state._widget = widget
        rebuild()
      // >RenderObjectElement.update
        super.update()
        widget.updateRenderObject(this, renderObject)
        _dirty = false
```

RenderObjectElement.performRebuild

```
:RenderObjectElement.performRebuild
  widget.updateRenderObject(this, renderObject)
  _dirty = false
```