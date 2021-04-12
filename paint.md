## flushPaint

```dart
// PipelineOwner.flushPaint()
// https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/rendering/object.dart#L959
void flushPaint() {
  try {
    final List<RenderObject> dirtyNodes = _nodesNeedingPaint;
    _nodesNeedingPaint = <RenderObject>[];
    // Sort the dirty nodes in reverse order (deepest first).
    for (final RenderObject node in dirtyNodes..sort((RenderObject a, RenderObject b) => b.depth - a.depth)) {
      if (node._needsPaint && node.owner == this) {
        if (node._layer!.attached) {
          PaintingContext.repaintCompositedChild(node);
        } else {
          node._skippedPaintingOnLayer();
        }
      }
    }
  }
}


// PaintingContext.repaintCompositedChild()
// https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/rendering/object.dart#L106
static void _repaintCompositedChild(
  RenderObject child, {
  bool debugAlsoPaintedParent = false,
  PaintingContext? childContext,
}) {
  OffsetLayer? childLayer = child._layer as OffsetLayer?;
  if (childLayer == null) {
    // Not using the `layer` setter because the setter asserts that we not
    // replace the layer for repaint boundaries. That assertion does not
    // apply here because this is exactly the place designed to create a
    // layer for repaint boundaries.
    child._layer = childLayer = OffsetLayer();
  } else {
    childLayer.removeAllChildren();
  }
  childContext ??= PaintingContext(child._layer!, child.paintBounds);
  child._paintWithContext(childContext, Offset.zero);

  // Double-check that the paint method did not replace the layer (the first
  // check is done in the [layer] setter itself).
  assert(identical(childLayer, child._layer));
  childContext.stopRecordingIfNeeded();
}


// RenderObject._paintWithContext()
// https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/rendering/object.dart#L2227
void _paintWithContext(PaintingContext context, Offset offset) {
  // If we still need layout, then that means that we were skipped in the
  // layout phase and therefore don't need painting. We might not know that
  // yet (that is, our layer might not have been detached yet), because the
  // same node that skipped us in layout is above us in the tree (obviously)
  // and therefore may not have had a chance to paint yet (since the tree
  // paints in reverse order). In particular this will happen if they have
  // a different layer, because there's a repaint boundary between us.
  if (_needsLayout)
    return;
  _needsPaint = false;
  try {
    paint(context, offset);
  }
}
```
