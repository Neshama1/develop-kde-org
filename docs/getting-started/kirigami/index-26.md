---
title: About page
weight: 304
group: advanced
aliases:
  - /docs/getting-started/kirigami/advanced-add_about_page/
description: Informations about your application
---

# About page

[Kirigami.AboutPage](docs:kirigami2;AboutPage) allows you to have a page that shows the copyright notice of the application together with the list of contributors and some information of which platform it's running on.

First, we are going to edit our `main.cpp` file from previous tutorials.

#### main.cpp

```qml
#include <QApplication>
#include <QQmlApplicationEngine>
#include <QtQml>
#include <QUrl>
#include <KAboutData>
#include <KLocalizedContext>
#include <KLocalizedString>

int main(int argc, char *argv[])
{
    QCoreApplication::setAttribute(Qt::AA_EnableHighDpiScaling);

    QApplication app(argc, argv);

    KLocalizedString::setApplicationDomain("helloworld");
    QCoreApplication::setOrganizationName(QStringLiteral("KDE"));
    QCoreApplication::setOrganizationDomain(QStringLiteral("kde.org"));
    QCoreApplication::setApplicationName(QStringLiteral("Hello World"));

    if (qEnvironmentVariableIsEmpty("QT_QUICK_CONTROLS_STYLE")) {
        QQuickStyle::setStyle(QStringLiteral("org.kde.desktop"));
    }

    KAboutData aboutData(
        QStringLiteral("helloworld"),
        i18nc("@title", "Hello World"),
        QStringLiteral("1.0"),
        i18n("Hello world application"),
        KAboutLicense::GPL,
        i18n("(c) 2021"));

    aboutData.addAuthor(
        i18nc("@info:credit", "Your name"),
        i18nc("@info:credit", "Author Role"),
        QStringLiteral("your@email.com"),
        QStringLiteral("https://yourwebsite.com"));

    // Set aboutData as information about the app
    KAboutData::setApplicationData(aboutData);

    // Register a singleton that will be accessible from QML.
    qmlRegisterSingletonType(
        "org.kde.example", // How the import statement should look like
        1, 0, // Major and minor versions of the import
        "About", // The name of the QML object
        [](QQmlEngine* engine, QJSEngine *) -> QJSValue {
            // Here we retrieve our aboutData and give it to the QML engine
            // to turn it into a QML type
            return engine->toScriptValue(KAboutData::applicationData());
        }
    );

    // Load an application from a QML file
    QQmlApplicationEngine engine;

    engine.rootContext()->setContextObject(new KLocalizedContext(&engine));
    engine.load(QUrl(QStringLiteral("qrc:/main.qml")));

    if (engine.rootObjects().isEmpty()) {
        return -1;
    }

    return app.exec();
}
```

In the `main.cpp` file we include [KAboutData](docs:kcoreaddons;KAboutData), a core KDE frameworks component that lets us store information about our application. This information can then be reused by many other KDE Frameworks components. We instantiate a new `aboutData` object with its fairly complete default constructor and add author information.

After all the required information has been set, we call [KAboutData::setApplicationData](docs:kcoreaddons;KAboutData::setApplicationData) to initialize the properties of the [QApplication ](docs:qtwidgets;QApplication)object.

We then create a [qmlRegisterSingletonType()](docs:qtqml;QQmlEngine::qmlRegisterSingletonType). This is used to allow us to import the C++ code as a module in our `main.qml` with `import org.kde.example 1.0`.

Its first argument is the URI that will be used for the import, the second and third arguments are major and minor versions respectively, the fourth is the type name, the name that we will call when accessing our `About` type, and the last is a reference to the C++ object that is exposed to QML. In the latter's case, we use a [lambda](https://en.cppreference.com/w/cpp/language/lambda) to instantiate the `aboutData` of our application in place.

#### main.qml

```qml
import QtQuick
import QtQuick.Layouts
import QtQuick.Controls as Controls
import org.kde.kirigami as Kirigami

import org.kde.example 1.0

Kirigami.ApplicationWindow {
    id: root

    title: i18nc("@title:window", "Day Kountdown")

    globalDrawer: Kirigami.GlobalDrawer {
        isMenu: true
        actions: [
            Kirigami.Action {
                text: i18n("Quit")
                icon.name: "gtk-quit"
                shortcut: StandardKey.Quit
                onTriggered: Qt.quit()
            },

            Kirigami.Action { // <==== Action to open About page
                text: i18n("About")
                icon.name: "help-about"
                onTriggered: pageStack.layers.push(aboutPage)
            }
        ]
    }

    Component { // <==== Component that instantiates the Kirigami.AboutPage
        id: aboutPage

        Kirigami.AboutPage {
            aboutData: About
        }
    }

    ListModel {
        id: kountdownModel
    }

    AddEditSheet {
        id: addEditSheet
        onAdded: kountdownModel.append({
            "name": name,
            "description": description,
            "date": Date.parse(kdate)
        });
        onEdited: kountdownModel.set(index, {
            "name": name,
            "description": description,
            "date": Date.parse(kdate)
        });
        onRemoved: kountdownModel.remove(index, 1)
    }

    function openPopulatedSheet(mode, index = -1, listName = "", listDesc = "", listDate = "") {
        addEditSheet.mode = mode
        addEditSheet.index = index;
        addEditSheet.name = listName
        addEditSheet.description = listDesc
        addEditSheet.kdate = listDate

        addEditSheet.open()
    }


    pageStack.initialPage: Kirigami.ScrollablePage {
        title: i18nc("@title", "Kountdown")

        actions: [
            Kirigami.Action {
                id: addAction
                icon.name: "list-add"
                text: i18nc("@action:button", "Add kountdown")
                onTriggered: openPopulatedSheet("add")
            }
        ]

        Kirigami.CardsListView {
            id: layout
            model: kountdownModel
            delegate: KountdownDelegate {}
        }
    }
}
```

First, we use the import we defined in the `main.cpp` file, namely `org.kde.example`. We then add a [Kirigami.Action](docs:kirigami2;Action) to our [global drawer](docs:kirigami2;GlobalDrawer) that will send us to the About page, and create a component with a [Kirigami.AboutPage](docs:kirigami2;AboutPage) in it, which expects a [KAboutData::applicationData()](docs:kcoreaddons;KAboutData::applicationData) object. We exposed precisely that in our `main.cpp` and called it `About`, so we can pass it here.

#### CMakeLists

```qml
cmake_minimum_required(VERSION 3.16)
project(helloworld)

find_package(ECM REQUIRED NO_MODULE)

set(CMAKE_MODULE_PATH ${ECM_MODULE_PATH})

include(KDEInstallDirs)
include(KDECMakeSettings)
include(KDECompilerSettings NO_POLICY_SCOPE)

find_package(Qt6 REQUIRED NO_MODULE COMPONENTS
    Core
    Quick
    Test
    Gui
    QuickControls2
    Widgets
)

find_package(KF6 REQUIRED COMPONENTS
    Kirigami2
    I18n
    CoreAddons
)

add_subdirectory(src)

feature_summary(WHAT ALL INCLUDE_QUIET_PACKAGES FATAL_ON_MISSING_REQUIRED_PACKAGES)
```

In the `CMakeLists.txt` file in our top-level folder, be sure to have `CoreAddons` in your [find\_package()](https://cmake.org/cmake/help/latest/command/find\_package.html) call. It is needed for [KAboutData](docs:kcoreaddons;KAboutData).

```qml
add_executable(helloworld)

target_sources(helloworld PRIVATE
    main.cpp
    resources.qrc
)

target_link_libraries(helloworld
    Qt6::Quick
    Qt6::Qml
    Qt6::Gui
    Qt6::QuickControls2
    Qt6::Widgets
    KF6::Kirigami2
    KF6::I18n
    KF6::CoreAddons
)
```

In the `CMakeLists.txt` file in the `src/` directory, nothing is needed since we instantiated out `aboutData` in place.

### Running the application

Now if you run your application and trigger the "About" action in the global drawer you should see our about page.

![Screenshot of the Kirigami About Page](../../../content/docs/getting-started/kirigami/advanced-add\_about\_page/about-page.png)
