# KTX: VisUI type-safe builders

Utilities for creating VisUI widgets using Kotlin type-safe builders.

### Why?

While LibGDX layout managers are simple enough to use directly in Kotlin or Java, their usage usually leads to overly
verbose code. GUI layouts presented in HTML, XML and other readable markup languages are easier to reason about than
cluttered Java code. Fortunately, Kotlin [type-safe builders](https://Kotlinlang.org/docs/reference/type-safe-builders.html)
allow to write DSL that is both as readable as markup languages and as powerful as Java.

### Guide

To start creating UI layout, call one of top level functions from `builder.kt` such as `table { }`, `verticalGroup { }`
or `gridGroup { }`. Every VisUI and LibGDX `WidgetGroup` has an equivalent method. By passing lambdas to these methods,
you can fully customize their content with extension functions for each `Actor`, as well as invoke any internal methods:

```Kotlin
val root = table {
  setFillParent(true)
  label("Hello, World!")
}
stage.addActor(root)
```

The closures get full access to all public methods that chosen `WidgetGroup` has. You can also create and immediately add 
new widgets to group simply by invoking methods from `WidgetFactory` interface, which is available from all type-safe 
builders.

In LibGDX `WidgetGroup` classes can be divided into two types: `Table`-like, where all actors are stored in fully
customizable `Cell` instances and regular groups, which handle children rendering internally. If the root actor extends
`Table` class, its actor-adding methods return `Cell` instance that you can use to set up individual actor properties:

```Kotlin
table {
  val cell = label("") // Assigning cell for further use.
  val actor = label("").actor // Extracting VisLabel from Cell. 
  label("").pad(1f).grow() // Adding VisLabel, changing Cell properties.
}

verticalGroup { 
  val label = label("") // No Cell wrapping: not a Table.
}
```

#### Additional extensions

Consider using [ktx-actors](../actors) module to improve event handling with lambda-friendly extension methods like
`onChange` and `onClick`.

#### Note about `KWidgets`
In `ktx-vis` there are many utility widget classes starting with `K` followed by their original names. Those widgets
purpose is to provide syntax sugar for type-safe builders, and there is usually no need use them directly. In fact, all
factory methods for root actors already return the extended widgets where necessary to help you with GUI building.

#### Tooltips
`ktx-vis` provides extension methods for creating VisUI tooltips:
```Kotlin
label("Label with tooltip") {
  addTextTooltip("Tooltip text")
}
```

These methods include:

- `addTextTooltip` - adds simple text tooltip.
- `addTooltip` - adds tooltip with fully customized content.

#### Menus

`Menu` and `PopupMenu` instances are created in very similar way to UI layouts.
```Kotlin
val menu = popupMenu {
  menuItem("First Item")
  menuItem("Second Item")
  menuItem("Third Item") {
    subMenu {
      menuItem("SubMenu Item")
    }
  }
}
// ...
menu.showMenu(stage, 0f, 0f)
```

See examples section for `MenuBar` usage.

### Usage examples

Creating a `VisWindow`, immediately added to a `Stage`:

```Kotlin
stage.addActor(window("Window") {
  isModal = true
  label("Hello from Window")
})
```

Creating a `MenuBar`:

```Kotlin
val menuBar = menuBar {
  menu("File") {
    menuItem("New") {
      subMenu {
        menuItem("Project")
        menuItem("Module")
        menuItem("File")
      }
    }
    menuItem("Open") { /**/ }
  }
  menu ("Edit") {
    menuItem("Undo") {
      setShortcut(Keys.CONTROL_LEFT, Keys.Z)
    }
    menuItem("Undo") { /**/ }
  }
}
rootTable.add(menuBar.table).top().growX().row()
```

![](http://dl.kotcrab.com/github/ktx/menu.png)

Creating `ButtonGroup`:
```Kotlin
buttonTable {
    checkBox("First")
    checkBox("Second")
    checkBox("Third")
    buttonGroup.setMinCheckCount(1)
    buttonGroup.setMaxCheckCount(1)
}
```

`ButtonTable` is a specialized `Table` that adds all `Button` instances to internal `ButtonGroup`, allowing to keep
minimum and maximum counts of buttons checked at once.

Creating a `ButtonBar`:

```Kotlin
buttonBar {
  setButton(APPLY, textButton("Accept"))
  setButton(CANCEL, textButton("Cancel"))
}
```

Creating a `TabbedPane` with multiple tabs:

```Kotlin
table {
  tabbedPane("vertical") {
    tab("Tab1") {
      label("Inside tab 1")
    }
    tab("Tab2") {
      label("Inside tab 2")
    }
    tab("Tab3") {
      label("Inside tab 3")
    }

    addTabContentsTo(table().grow())
    //OR addTabContentsTo(container<Table>().grow())

    switchTab(0)
  }.cell.growY()
}
```

![](http://dl.kotcrab.com/github/ktx/tabs.png)

Creating a form using `FormValidator`:

```Kotlin
table(true) {
  validator {
    defaults().left()

    label("Name: ")
    notEmpty(validatableTextField().grow().actor, "Name can't be empty")
    row()

    label("Age: ")
    val ageField = validatableTextField().grow().actor
    notEmpty(ageField, "Age can't be empty")
    integerNumber(ageField, "Age must be number")
    valueGreaterThan(ageField, "You must be at least 18 years old", 18f, true)
    row()

    checked(checkBox("Accept terms").colspan(2).actor, "You must accept terms")
    row()

    table(true) {
      setMessageLabel(label("").width(200f).actor)
      addDisableTarget(textButton("Accept").right().actor)
    }.colspan(2)
  }
}
```

![](http://dl.kotcrab.com/github/ktx/form.png)

Creating a `ListView`:

```Kotlin
import com.badlogic.gdx.utils.Array as GdxArray
// ...

table(true) {
  val osList = GdxArray<String>()
  osList.add("Windows")
  osList.add("Linux")
  osList.add("Mac")

  val adapter = SimpleListAdapter(osList)
  adapter.selectionMode = SINGLE
  listView(adapter) {
    header = label("ListView header")
    footer = label("ListView footer")
  }
}
```

![](http://dl.kotcrab.com/github/ktx/list.png)


### Alternatives

- Creating layouts with [VisUI](https://github.com/kotcrab/vis-editor/wiki/VisUI) directly in Kotlin or Java.
- [LibGDX Markup Language](https://github.com/czyzby/gdx-lml/tree/master/lml) allows to build `Scene2D` views using
HTML-like syntax. It also features a [VisUI extension](https://github.com/czyzby/gdx-lml/tree/master/lml-vis). However,
it lacks first-class Kotlin support and the flexibility of a powerful programming language.
