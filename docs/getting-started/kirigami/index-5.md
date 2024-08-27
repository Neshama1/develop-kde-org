---
title: Using separate files
group: introduction
weight: 6
aliases:
  - /docs/getting-started/kirigami/introduction-separatefiles/
description: >-
  Separating unwieldy code into different files, and attach signals to your
  components.
---

# Using separate files

### Why and how

For the first time, we will be separating some of our components into their own QML files. If we keep adding things to `Main.qml`, it's going to quickly become hard to tell what does what, and we risk muddying our code.

In this tutorial, we will be splitting the code in `Main.qml` into `Main.qml`, `AddDialog.qml` and `KountdownDelegate.qml`.

Additionally, even when spreading code between multiple QML files, the amount of files in real-life projects can get out of hand. A common solution to this problem is to logically separate files into different folders. We will take a brief look at three common approaches seen in real projects, and implement one of them:

* storing QML files together with C++ files
* storing QML files in a different directory under the same module
* storing QML files in a different directory under a different module

After the split, we will have [separation of concerns](https://en.wikipedia.org/wiki/Separation\_of\_concerns) between each file, and [implementation details will be abstracted](https://en.wikipedia.org/wiki/Abstraction\_\(computer\_science\)), making the code more readable.

#### Storing QML files together with C++ files

This consists of keeping the project's QML files together with C++ files in `src/`. This sort of structure would look like this:

```
kirigami-tutorial/
├── CMakeLists.txt
├── org.kde.tutorial.desktop
└── src/
    ├── CMakeLists.txt
    ├── main.cpp
    ├── Main.qml
    ├── AddDialog.qml
    └── KountdownDelegate.qml
```

This is what we did previously. In the above case, you would just need to keep adding QML files to the existing `kirigami-tutorial/src/CMakeLists.txt`. There's no logical separation at all, and once the project gets more than a couple of QML files (and C++ files that create types to be used in QML), the folder can quickly become crowded.

#### Storing QML files in a different directory under the same module

This consists of keeping all QML files in a separate folder, usually `src/qml/`. This sort of structure would look like this:

```
kirigami-tutorial/
├── CMakeLists.txt
├── org.kde.tutorial.desktop
└── src/
    ├── CMakeLists.txt
    ├── main.cpp
    └── qml/
        ├── Main.qml
        ├── AddDialog.qml
        └── KountdownDelegate.qml
```

This structure is very common in KDE projects, mostly to avoid having an extra CMakeLists.txt file for the `src/qml/` directory and creating a separate module. This method keeps the files themselves in a separate folder, but you would also need to add them in `kirigami-tutorial/src/CMakeLists.txt`. All created QML files would then belong to the same QML module as `Main.qml`.

In practice, once the project gets more than a dozen QML files, while it won't crowd the `src/` directory, it will crowd the `src/CMakeLists.txt` file. It will become difficult to differentiate between traditional C++ files and C++ files that have types exposed to QML.

It will also break the concept of locality (localisation of dependency details), where you would keep the description of your dependencies in the same place as the dependencies themselves.

#### Storing QML files in a different directory under a different module

This consists of keeping all QML files in a separate folder with its own CMakeLists.txt and own separate QML module. This sort of structure would look like this:

```
kirigami-tutorial/
├── CMakeLists.txt
├── org.kde.tutorial.desktop
└── src/
    ├── CMakeLists.txt
    ├── main.cpp
    ├── Main.qml
    └── components/
        ├── CMakeLists.txt
        ├── AddDialog.qml
        └── KountdownDelegate.qml
```

This structure is not as common in KDE projects and requires writing an additional CMakeLists.txt, but it is the most flexible. In our case, we name our folder "components" since we are creating two new QML components out of our previous `Main.qml` file, and keep information about them in `kirigami-tutorial/src/components/CMakeLists.txt`. The `Main.qml` file itself stays in `src/` so it's automatically used when running the executable, as before.

Later on, it would be possible to create more folders with multiple QML files, all grouped together by function, such as "models" and "settings", and C++ files that have types exposed to QML (like models) could be kept together with other QML files where it makes sense.

We will be using this structure in this tutorial.

### Preparing CMake for the new files

First, create the file `kirigami-tutorial/src/components/CMakeLists.txt` with the following contents:

```qml
add_library(kirigami-hello-components)

ecm_add_qml_module(kirigami-hello-components
    URI "org.kde.tutorial.components"
    GENERATE_PLUGIN_SOURCE
)

ecm_target_qml_sources(kirigami-hello-components
    SOURCES
    AddDialog.qml
    KountdownDelegate.qml
)

ecm_finalize_qml_module(kirigami-hello-components)

install(TARGETS kirigami-hello-components ${KDE_INSTALL_TARGETS_DEFAULT_ARGS})

```

We create a new target called `kirigami-hello-components` and then turn it into a QML module using [ecm\_add\_qml\_module()](https://api.kde.org/ecm/module/ECMQmlModule.html) under the import name `org.kde.tutorial.components` and add the relevant QML files.

Because the target is different from the executable, it will function as a different QML module, in which case we will need to do two things: make it _generate_ code for it to work as a Qt plugin with [GENERATE\_PLUGIN\_SOURCE](https://api.kde.org/ecm/module/ECMQmlModule.html), and _finalize_ it with [ecm\_finalize\_qml\_module()](https://api.kde.org/ecm/module/ECMQmlModule.html). We then install it exactly like in previous lessons.

We needed to use [add\_library()](https://cmake.org/cmake/help/latest/command/add\_library.html) so that we can link `kirigami-hello-components` to the executable in the [target\_link\_libraries()](https://cmake.org/cmake/help/latest/command/target\_link\_libraries.html) call in `kirigami-tutorial/src/CMakeLists.txt`:

```qml
add_executable(kirigami-hello)

ecm_add_qml_module(kirigami-hello
    URI
    org.kde.tutorial
)

target_sources(kirigami-hello
    PRIVATE
    main.cpp
)

ecm_target_qml_sources(kirigami-hello
    SOURCES
    Main.qml
)

target_link_libraries(kirigami-hello
    PRIVATE
    Qt6::Quick
    Qt6::Qml
    Qt6::Gui
    Qt6::QuickControls2
    Qt6::Widgets
    KF6::I18n
    KF6::CoreAddons
    kirigami-hello-components
)

install(TARGETS kirigami-hello ${KDE_INSTALL_TARGETS_DEFAULT_ARGS})

add_subdirectory(components)
```

We also need to use [add\_subdirectory()](https://cmake.org/cmake/help/latest/command/add\_subdirectory.html) so CMake will find the `kirigami-tutorial/src/components/` directory.

In the previous lessons, we did not need to add the `org.kde.tutorial` import to our `Main.qml` because it was not needed: being the entrypoint for the application, the executable would run the file immediately anyway. Since our components are in a separate QML module, the a new import in `kirigami-tutorial/src/Main.qml` is necessary, the same one defined earlier, `org.kde.tutorial.components`:

```qml
import QtQuick
import QtQuick.Layouts
import QtQuick.Controls as Controls
import org.kde.kirigami as Kirigami
import org.kde.tutorial.components

// The rest of the code...
```

And we are ready to go.

### Splitting Main.qml

Let's take a look once again at the original `Main.qml`:

```qml
import QtQuick
import QtQuick.Layouts
import QtQuick.Controls as Controls
import org.kde.kirigami as Kirigami

Kirigami.ApplicationWindow {
    id: root

    width: 600
    height: 400

    title: i18nc("@title:window", "Day Kountdown")

    globalDrawer: Kirigami.GlobalDrawer {
        isMenu: true
        actions: [
            Kirigami.Action {
                text: i18n("Quit")
                icon.name: "application-exit-symbolic"
                shortcut: StandardKey.Quit
                onTriggered: Qt.quit()
            }
        ]
    }

    ListModel {
        id: kountdownModel
    }

    Component {
        id: kountdownDelegate
        Kirigami.AbstractCard {
            contentItem: Item {
                implicitWidth: delegateLayout.implicitWidth
                implicitHeight: delegateLayout.implicitHeight
                GridLayout {
                    id: delegateLayout
                    anchors {
                        left: parent.left
                        top: parent.top
                        right: parent.right
                    }
                    rowSpacing: Kirigami.Units.largeSpacing
                    columnSpacing: Kirigami.Units.largeSpacing
                    columns: root.wideScreen ? 4 : 2

                    Kirigami.Heading {
                        level: 1
                        text: i18n("%1 days", Math.round((date-Date.now())/86400000))
                    }

                    ColumnLayout {
                        Kirigami.Heading {
                            Layout.fillWidth: true
                            level: 2
                            text: name
                        }
                        Kirigami.Separator {
                            Layout.fillWidth: true
                            visible: description.length > 0
                        }
                        Controls.Label {
                            Layout.fillWidth: true
                            wrapMode: Text.WordWrap
                            text: description
                            visible: description.length > 0
                        }
                    }
                    Controls.Button {
                        Layout.alignment: Qt.AlignRight
                        Layout.columnSpan: 2
                        text: i18n("Edit")
                    }
                }
            }
        }
    }

    Kirigami.Dialog {
        id: addDialog
        title: i18nc("@title:window", "Add kountdown")
        standardButtons: Kirigami.Dialog.Ok | Kirigami.Dialog.Cancel
        padding: Kirigami.Units.largeSpacing
        preferredWidth: Kirigami.Units.gridUnit * 20

        // Form layouts help align and structure a layout with several inputs
        Kirigami.FormLayout {
            // Textfields let you input text in a thin textbox
            Controls.TextField {
                id: nameField
                // Provides a label attached to the textfield
                Kirigami.FormData.label: i18nc("@label:textbox", "Name*:")
                // What to do after input is accepted (i.e. pressed Enter)
                // In this case, it moves the focus to the next field
                onAccepted: descriptionField.forceActiveFocus()
            }
            Controls.TextField {
                id: descriptionField
                Kirigami.FormData.label: i18nc("@label:textbox", "Description:")
                placeholderText: i18n("Optional")
                // Again, it moves the focus to the next field
                onAccepted: dateField.forceActiveFocus()
            }
            Controls.TextField {
                id: dateField
                Kirigami.FormData.label: i18nc("@label:textbox", "ISO Date*:")
                // D means a required number between 1-9,
                // 9 means a required number between 0-9
                inputMask: "D999-99-99"
                // Here we confirm the operation just like
                // clicking the OK button
                onAccepted: addDialog.onAccepted()
            }
            Controls.Label {
                text: "* = required fields"
            }
        }
        // Once the Kirigami.Dialog is initialized,
        // we want to create a custom binding to only
        // make the Ok button visible if the required
        // text fields are filled.
        // For this we use Kirigami.Dialog.standardButton(button):
        Component.onCompleted: {
            const button = standardButton(Kirigami.Dialog.Ok);
            // () => is a JavaScript arrow function
            button.enabled = Qt.binding( () => requiredFieldsFilled() );
        }
        onAccepted: {
            // The binding is created, but we still need to make it
            // unclickable unless the fields are filled
            if (!addDialog.requiredFieldsFilled()) return;
            appendDataToModel();
            clearFieldsAndClose();
        }
        // We check that the nameField is not empty and that the
        // dateField (which has an inputMask) is completely filled
        function requiredFieldsFilled() {
            return (nameField.text !== "" && dateField.acceptableInput);
        }
        function appendDataToModel() {
            kountdownModel.append({
                name: nameField.text,
                description: descriptionField.text,
                date: new Date(dateField.text)
            });
        }
        function clearFieldsAndClose() {
            nameField.text = ""
            descriptionField.text = ""
            dateField.text = ""
            addDialog.close();
        }
    }

    pageStack.initialPage: Kirigami.ScrollablePage {
        title: i18nc("@title", "Kountdown")

        // Kirigami.Action encapsulates a UI action. Inherits from Controls.Action
        actions: [
            Kirigami.Action {
                id: addAction
                // Name of icon associated with the action
                icon.name: "list-add-symbolic"
                // Action text, i18n function returns translated string
                text: i18nc("@action:button", "Add kountdown")
                // What to do when triggering the action
                onTriggered: addDialog.open()
            }
        ]

        Kirigami.CardsListView {
            id: cardsView
            model: kountdownModel
            delegate: kountdownDelegate
        }
    }
}
```

The custom delegate with `id: kountdownDelegate` can be split completely because it is already enveloped in a [QML Component type](docs:qtqml;QtQml.Component). We use a Component to be able to define it without needing to instantiate it; separate QML files work the same way.

If we move the code to a separate files, then, there is no point in leaving it enveloped in a Component: we can split just the [Kirigami.AbstractCard](docs:kirigami2;AbstractCard) in the separate file. Here is the resulting `KountdownDelegate.qml`:

```qml
import QtQuick
import QtQuick.Layouts
import QtQuick.Controls as Controls
import org.kde.kirigami as Kirigami

Kirigami.AbstractCard {
    contentItem: Item {
        implicitWidth: delegateLayout.implicitWidth
        implicitHeight: delegateLayout.implicitHeight
        GridLayout {
            id: delegateLayout
            anchors {
                left: parent.left
                top: parent.top
                right: parent.right
            }
            rowSpacing: Kirigami.Units.largeSpacing
            columnSpacing: Kirigami.Units.largeSpacing
            columns: root.wideScreen ? 4 : 2

            Kirigami.Heading {
                level: 1
                text: i18n("%1 days", Math.round((date-Date.now())/86400000))
            }

            ColumnLayout {
                Kirigami.Heading {
                    Layout.fillWidth: true
                    level: 2
                    text: name
                }
                Kirigami.Separator {
                    Layout.fillWidth: true
                    visible: description.length > 0
                }
                Controls.Label {
                    Layout.fillWidth: true
                    wrapMode: Text.WordWrap
                    text: description
                    visible: description.length > 0
                }
            }
            Controls.Button {
                Layout.alignment: Qt.AlignRight
                Layout.columnSpan: 2
                text: i18n("Edit")
            }
        }
    }
}
```

Our dialog with `id: addDialog` is not enveloped in a Component, and it is not a component that is visible by default, so the code can be copied as is into the `AddDialog.qml`:

```qml
import QtQuick
import QtQuick.Layouts
import QtQuick.Controls as Controls
import org.kde.kirigami as Kirigami

Kirigami.Dialog {
    id: addDialog
    title: i18nc("@title:window", "Add kountdown")
    standardButtons: Kirigami.Dialog.Ok | Kirigami.Dialog.Cancel
    padding: Kirigami.Units.largeSpacing
    preferredWidth: Kirigami.Units.gridUnit * 20

    Kirigami.FormLayout {
        Controls.TextField {
            id: nameField
            Kirigami.FormData.label: i18nc("@label:textbox", "Name*:")
            onAccepted: descriptionField.forceActiveFocus()
        }
        Controls.TextField {
            id: descriptionField
            Kirigami.FormData.label: i18nc("@label:textbox", "Description:")
            onAccepted: dateField.forceActiveFocus()
        }
        Controls.TextField {
            id: dateField
            Kirigami.FormData.label: i18nc("@label:textbox", "ISO Date*:")
            inputMask: "D999-99-99"
            onAccepted: addDialog.accepted()
        }
        Controls.Label {
            text: "* = required fields"
        }
    }
    Component.onCompleted: {
        const button = standardButton(Kirigami.Dialog.Ok);
        button.enabled = Qt.binding( () => requiredFieldsFilled() );
    }
    onAccepted: {
        if (!addDialog.requiredFieldsFilled()) return;
        appendDataToModel();
        clearFieldsAndClose();
    }
    function requiredFieldsFilled() {
        return (nameField.text !== "" && dateField.acceptableInput);
    }
    function appendDataToModel() {
        kountdownModel.append({
            name: nameField.text,
            description: descriptionField.text,
            date: new Date(dateField.text)
        });
    }
    function clearFieldsAndClose() {
        nameField.text = ""
        descriptionField.text = ""
        dateField.text = ""
        addDialog.close();
    }
}
```

With the code split, `Main.qml` thus becomes much shorter:

```qml
import QtQuick
import QtQuick.Layouts
import QtQuick.Controls as Controls
import org.kde.kirigami as Kirigami
import org.kde.tutorial.components

Kirigami.ApplicationWindow {
    id: root

    width: 600
    height: 400

    title: i18nc("@title:window", "Day Kountdown")

    globalDrawer: Kirigami.GlobalDrawer {
        isMenu: true
        actions: [
            Kirigami.Action {
                text: i18n("Quit")
                icon.name: "application-exit-symbolic"
                shortcut: StandardKey.Quit
                onTriggered: Qt.quit()
            }
        ]
    }

    ListModel {
        id: kountdownModel
    }

    AddDialog {
        id: addDialog
    }

    pageStack.initialPage: Kirigami.ScrollablePage {
        title: i18nc("@title", "Kountdown")

        actions: [
            Kirigami.Action {
                id: addAction
                icon.name: "list-add-symbolic"
                text: i18nc("@action:button", "Add kountdown")
                onTriggered: addDialog.open()
            }
        ]

        Kirigami.CardsListView {
            id: cardsView
            model: kountdownModel
            delegate: KountdownDelegate {}
        }
    }
}
```

We now have two extra QML files, `AddDialog.qml` and `KountdownDelegate`, and we need to find some way of using them in `Main.qml`. The way to add the contents of the new files to `Main.qml` is by _instantiating_ them.

`AddDialog.qml` becomes `AddDialog {}`:

```qml
    AddDialog {
        id: addDialog
    }
```

`KountdownDelegate.qml` becomes `KountdownDelegate {}`:

```qml
        Kirigami.CardsListView {
            id: cardsView
            model: kountdownModel
            delegate: KountdownDelegate {}
        }
```

Most cases you have seen of a component started with uppercase and followed by brackets were instantiations of a QML component. This is why our new QML files need to start with an uppercase letter.

Compile the project and run it, and you should have a functional window that behaves exactly the same as before, but with the code split into separate parts, making things much more manageable.
