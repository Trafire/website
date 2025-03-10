---
title: Drag and drop
sidebar_label: Drag and drop
---

The mechanics of drag-and-drop in Flet is pretty simple - a user starts dragging [`Draggable`](/docs/controls/draggable) control and "drops" it on [`DragTarget`](/docs/controls/dragtarget). If both draggable and drag target has the same `group` a drag target will call `on_accept` event handler and pass draggable control ID as event data. In this case draggable serves as a source "data" for drag-and-drop operation.

Let's take a look at the following example. In the program below you can drag left control displaying "1" on top of the right control dislaying "0" and when drag operation completes left control is replaced with "0" and the right control becomes "1":

```python
import flet
from flet import Container, Draggable, DragTarget, Page, Row, Text, alignment, colors

def main(page: Page):
    page.title = "Drag and Drop example"

    def drag_accept(e):
        # get draggable (source) control by its ID
        src = page.get_control(e.data)
        # update text inside draggable control
        src.content.content.value = "0"
        # update text inside drag target control
        e.control.content.content.value = "1"
        page.update()

    page.add(
        Row(
            [
                Draggable(
                    group="number",
                    content=Container(
                        width=50,
                        height=50,
                        bgcolor=colors.CYAN_200,
                        border_radius=5,
                        content=Text("1", size=20),
                        alignment=alignment.center,
                    ),
                ),
                Container(width=100),
                DragTarget(
                    group="number",
                    content=Container(
                        width=50,
                        height=50,
                        bgcolor=colors.PINK_200,
                        border_radius=5,
                        content=Text("0", size=20),
                        alignment=alignment.center,
                    ),
                    on_accept=drag_accept,
                ),
            ]
        )
    )

flet.app(target=main)
```

<img src="/img/docs/getting-started/drag-and-drop-number.gif"className="screenshot-50" />

So, it's developer's responsibility to determine what happens with "source" (draggable) and "destination" (drag target) controls when `on_accept` event occurs.

:::noteTry something
Change DragTarget's group property to `number1` and note `on_accept` is not called any more when you drop "1" on the target.
:::

There are additional properties and event handlers to make drag-and-drop operation even more interactive. For example, draggable has `content_when_dragging` property to display a different control instead of `content` when drag operation is under way. There is also `content_feedback` property to show a different control under the pointer. By default, the same `content` control, but with 50% opacity is displayed under cursor when dragging.

Let's modify Draggable in our example to display a "hole" in place of dragged control and just "1" under cursor while dragging:

```python
...
                Draggable(
                    group="number",
                    content=Container(
                        width=50,
                        height=50,
                        bgcolor=colors.CYAN_200,
                        border_radius=5,
                        content=Text("1", size=20),
                        alignment=alignment.center,
                    ),
                    content_when_dragging=Container(
                        width=50,
                        height=50,
                        bgcolor=colors.BLUE_GREY_200,
                        border_radius=5,
                    ),
                    content_feedback=Text("1"),
                ),
...
```

<img src="/img/docs/getting-started/drag-and-drop-number-2.gif"className="screenshot-50" />

Drag target control additionally has `on_will_accept` and `on_leave` event handlers which help better visualize when it's a good time to "drop" something on the target. Let's modify DragTarget in our example to draw a border around target control when it's ready to accept incoming drag:

```python
import flet
from flet import (Container, Draggable, DragTarget, Page, Row, Text, alignment, border, colors)

def main(page: Page):
    page.title = "Drag and Drop example 2"

    def drag_accept(e):
        # get draggable (source) control by its ID
        src = page.get_control(e.data)
        # update text inside draggable control
        src.content.content.value = "0"
        # reset source group, so it cannot be dropped to a target anymore
        src.group = ""
        # update text inside drag target control
        e.control.content.content.value = "1"
        # reset border
        e.control.content.border = None
        page.update()

    def drag_will_accept(e):
        # black border when it's allowed to drop and red when it's not
        e.control.content.border = border.all(
            2, colors.BLACK45 if e.data == "true" else colors.RED
        )
        e.control.update()

    def drag_leave(e):
        e.control.content.border = None
        e.control.update()

    page.add(
        Row(
            [
                Draggable(
                    group="number",
                    content=Container(
                        width=50,
                        height=50,
                        bgcolor=colors.CYAN_200,
                        border_radius=5,
                        content=Text("1", size=20),
                        alignment=alignment.center,
                    ),
                    content_when_dragging=Container(
                        width=50,
                        height=50,
                        bgcolor=colors.BLUE_GREY_200,
                        border_radius=5,
                    ),
                    content_feedback=Text("1"),
                ),
                Container(width=100),
                DragTarget(
                    group="number",
                    content=Container(
                        width=50,
                        height=50,
                        bgcolor=colors.PINK_200,
                        border_radius=5,
                        content=Text("0", size=20),
                        alignment=alignment.center,
                    ),
                    on_accept=drag_accept,
                    on_will_accept=drag_will_accept,
                    on_leave=drag_leave,
                ),
            ]
        )
    )

flet.app(target=main)
```

<img src="/img/docs/getting-started/drag-and-drop-number-3.gif"className="screenshot-50" />