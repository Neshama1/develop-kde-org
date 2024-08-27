---
title: Introduction to Kirigami Addons
weight: 401
group: addons
description: Get acquainted with Kirigami Addons components
---

# Introduction to Kirigami Addons

Kirigami Addons is an additional set of visual components that work well on mobile and desktop and are guaranteed to be cross-platform. It uses Kirigami under the hood to create its components.

Here you will be setting up your new Kirigami Addons project and be introduced to a few useful components.

These components make use of KDE's localization facilities, so before we start using these, we will need to set a little project that makes use of [KLocalizedContext](docs:ki18n;KLocalizedContext).

### Setting up your project

The initial project structure will look like so:

```bash
addonsexample/
├── CMakeLists.txt
├── main.cpp
├── resources.qrc
└── contents/
    └── ui/
        └── main.qml
```

{% hint style="success" %}Tip

You can quickly create this file structure with:

```bash
mkdir -p addonsexample/contents/ui
touch addonsexample/{CMakeLists.txt,main.cpp,resources.qrc}
touch addonsexample/contents/ui/main.qml
```

{% endhint %}

We start by using a very standard `CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.20)

project(FormCardTutorial)

find_package(ECM REQUIRED NO_MODULE)
set(CMAKE_MODULE_PATH ${ECM_MODULE_PATH})

include(KDEInstallDirs)
include(KDECompilerSettings)
include(KDECMakeSettings)
include(ECMQmlModule)

find_package(Qt6 REQUIRED COMPONENTS
    Widgets        # For QApplication
    Quick          # For QML
    QuickControls2 # For QQuickStyle
)
find_package(KF6 REQUIRED COMPONENTS
    CoreAddons # For KAboutData
    I18n       # For KLocalizedContext
)
qt_policy(SET QTP0001 NEW)

add_executable(addonsexample)

target_sources(addonsexample PRIVATE main.cpp)

ecm_add_qml_module(addonsexample
    GENERATE_PLUGIN_SOURCE
    URI org.kde.addonsexample
)

ecm_target_qml_sources(addonsexample SOURCES
    Main.qml
    SettingsPage.qml
    JsonAboutPage.qml
)

target_link_libraries(addonsexample PRIVATE
    Qt::Widgets
    Qt::Quick
    Qt::QuickControls2
    KF6::CoreAddons
    KF6::I18n
)

install(TARGETS addonsexample DESTINATION ${KDE_INSTALL_BINDIR})
```

A standard `resources.qrc`:

```
<!DOCTYPE RCC>
<RCC version="1.0">
<qresource prefix="/">
    <file alias="main.qml">contents/ui/main.qml</file>
</qresource>
</RCC>
```

The interesting part will be the `main.cpp`:

```cpp
#include <QtQml>
#include <QApplication>
#include <QQmlApplicationEngine>
#include <QIcon>
#include <QQuickStyle>
#include <KAboutData>
#include <KLocalizedContext>
#include <KLocalizedString>

int main(int argCount, char* argVector[])
{
    QApplication app(argCount, argVector);
    KLocalizedString::setApplicationDomain("org.kde.addonsexample");

    KAboutData aboutData(
        QStringLiteral("addonsexample"),
        i18nc("@title:window", "Addons Example"),
        QStringLiteral("1.0"),
        i18nc("@info", "This program shows how to use Kirigami Addons"),
        KAboutLicense::GPL_V3,
        QStringLiteral("(C) 2023"),
        i18nc("@info", "Optional text shown in the About"),
        QStringLiteral("https://kde.org"));

    aboutData.addAuthor(i18nc("@info:credit", "John Doe"),
                        i18nc("@info:credit", "Maintainer"));

    KAboutData::setApplicationData(aboutData);

    if (qEnvironmentVariableIsEmpty("QT_QUICK_CONTROLS_STYLE")) {
        QQuickStyle::setStyle(QStringLiteral("org.kde.desktop"));
    }
    QApplication::setWindowIcon(QIcon::fromTheme(QStringLiteral("kde")));

    QQmlApplicationEngine engine;
    engine.rootContext()->setContextObject(new KLocalizedContext(&engine));
    engine.loadFromModule("org.kde.addonsexample", "Main");
    app.exec();
}
```

If you have read our [KXmlGui tutorial](../kxmlgui/) or the last Kirigami tutorial on the [Kirigami About page](advanced-add\_about\_page/), much of this will seem familiar to you.

We create our application and use [KAboutData's](docs:kcoreaddons;KAboutData) default constructor to add the metadata of our application, add ourselves as an author, and then use [setApplicationData()](docs:kcoreaddons;KAboutData::setApplicationData) to finish the process. For later, we also set an application icon that comes from the system theme.

We then use a lambda in [qmlRegisterSingletonType](docs:qtqml;QQmlEngine::qmlRegisterSingletonType) to directly send the metadata of our application to the QML side, exposing its properties.

We then instantiate our QML engine, and set its [context](docs:qtqml;QQmlContext) to use KDE's [KLocalizedContext](docs:ki18n;KLocalizedContext), used to integrate translated strings, passing the just created engine as a parameter.

We simply load our QML file from the resource file, and now we just need to take care of our initial QML file.

### FormCard and FormButtonDelegate

The idea for our app is to design our own Kirigami Addons gallery, showcasing multiple components, one per page. The main page will contain a simple list of buttons in a [ColumnLayout](https://doc.qt.io/qt-6/qml-qtquick-layouts-columnlayout.html), each opening a separate page.

Initially, our `contents/ui/main.qml` should look like this:

```qml
import QtQuick
import QtQuick.Layouts

import org.kde.kirigami as Kirigami
import org.kde.kirigamiaddons.formcard as FormCard

import org.kde.about 1.0

Kirigami.ApplicationWindow {
    id: root
    width: 600
    height: 700

    pageStack.initialPage: Kirigami.ScrollablePage {
        ColumnLayout {
            // Our code will go here
        }
    }
}
```

We use our handy [pageStack](components-pagerow\_pagestack/) to set the initial page to a [Kirigami.ScrollablePage](docs:kirigami2;ScrollablePage).

While we could use a [FormLayout](components-formlayouts/) together with [QtQuick Controls components](../../../content/docs/getting-started/kirigami/addons-introduction/components-controls/) to achieve our goal, here you will be introduced to [FormCard](https://api.kde.org/frameworks/kirigami-addons/html/classFormCard.html).

The main purpose of a FormCard is to serve as a container for other components while following a color different from the background, in a similar manner to a [Kirigami.Card](docs:kirigami2;Card), but for settings windows. You can have multiple FormCards in your application to indicate different sections. Your FormCard is also expected to be a direct child of a ColumnLayout.

Importing `org.kde.kirigamiaddons.formcard` makes all FormCard components available to your QML file.

We will have only a single section in our main page, so we add a single FormCard:

```qml
import QtQuick
import QtQuick.Layouts

import org.kde.kirigami as Kirigami
import org.kde.kirigamiaddons.formcard as FormCard

import org.kde.about 1.0

Kirigami.ApplicationWindow {
    id: root
    width: 600
    height: 700

    pageStack.initialPage: Kirigami.ScrollablePage {
        ColumnLayout {
            FormCard.FormCard {
                // Our buttons will go here
            }
        }
    }
}
```

The great thing about FormCard is that it does automatic layouting for you. In other words, just the order of its components is enough to indicate their position inside the FormCard, no [Layout attached properties](https://doc.qt.io/qt-6/qml-qtquick-layouts-layout.html) are necessary and you are expected not to use [anchors](https://doc.qt.io/qt-6/qtquick-positioning-anchors.html) or [positioners](https://doc.qt.io/qt-6/qtquick-positioning-layouts.html).

We can simply add a few buttons inside our FormCard:

```qml
import QtQuick
import QtQuick.Layouts

import org.kde.kirigami as Kirigami
import org.kde.kirigamiaddons.formcard as FormCard

import org.kde.about 1.0

Kirigami.ApplicationWindow {
    id: root
    width: 600
    height: 700

    pageStack.initialPage: Kirigami.ScrollablePage {
        ColumnLayout {
            FormCard.FormCard {
                FormCard.FormButtonDelegate {
                    id: aboutKDEButton
                    icon.name: "kde"
                    text: i18n("About KDE Page")
                }
                FormCard.FormButtonDelegate {
                    id: aboutPageButton
                    icon.name: "applications-utilities"
                    text: i18n("About Addons Example")
                }
                FormCard.FormButtonDelegate {
                    id: settingsButton
                    icon.name: "settings-configure"
                    text: i18n("Single Settings Page")
                }
            }
        }
    }
}
```

That's it! The buttons are not usable just yet, but we are now set up to play with our About pages!

We then build and run it like so:

```bash
cmake -B build/ -DCMAKE_INSTALL_PREFIX=~/kde5/usr
cmake --build build/
cmake --install build/
aboutexample
```

To see other ways to build your application (for example, on Windows), see the [Getting Started with Kirigami](introduction-getting\_started/) page.

![](../../../content/docs/getting-started/kirigami/addons-introduction/setting-up.webp)
