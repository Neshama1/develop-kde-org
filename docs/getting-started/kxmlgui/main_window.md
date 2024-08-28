---
title: Creating the main window
weight: 2
aliases:
  - /docs/getting-started/main_window/
description: >-
  This tutorial shows you the magic of an application's most important thing:
  the main window.
---

# Creating the main window

### Summary

This tutorial carries on from our \[Hello World project]\(\{{< ref "hello\_world.md" >\}}) and will introduce the [KXmlGuiWindow](docs:kxmlgui;KXmlGuiWindow) class.

In the previous tutorial, the program caused a dialog box to pop up. Now we are going to take steps towards creating a functioning application with a more advanced window structure.

![](../../../content/docs/getting-started/kxmlgui/main_window/main\_window.webp)

### KXmlGuiWindow

[KXmlGuiWindow](docs:kxmlgui;KXmlGuiWindow) provides a full main window view with menubars, toolbars, a statusbar and a main area in the centre for a large widget. For example, the help menu is predefined. Most KDE applications will derive from this class as it provides an easy way to define menu and toolbar layouts through XML files (this technology is called [KXmlGui](docs:kxmlgui)). While we will not be using it in this tutorial, we will use it in the next.

In order to have a useful [KXmlGuiWindow](docs:kxmlgui;KXmlGuiWindow), we must subclass it. So we create two files, `mainwindow.cpp` and `mainwindow.h`, which will contain our code.

#### mainwindow.h

```cpp
#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include <KXmlGuiWindow>

class KTextEdit;

class MainWindow : public KXmlGuiWindow
{
public:
    explicit MainWindow(QWidget *parent = nullptr);

private:
    KTextEdit *textArea;
};

#endif // MAINWINDOW_H
```

First we [subclass](https://en.wikipedia.org/wiki/Inheritance\_\(object-oriented\_programming\)#Subclasses\_and\_superclasses) [KXmlGuiWindow](docs:kxmlgui;KXmlGuiWindow) with `class MainWindow : public KXmlGuiWindow`, then we declare the [constructor](https://en.wikipedia.org/wiki/Constructor\_\(object-oriented\_programming\)) with `MainWindow(QWidget *parent = nullptr);`.

Finally, we declare a pointer to the object that will make up the bulk of our program, turning it into a text editor. [`KTextEdit`](docs:ktextwidgets;KTextEdit) is a generic rich text editing widget with some niceties like cursor auto-hiding and spell checking.

#### mainwindow.cpp

```cpp
#include <KTextEdit>
#include "mainwindow.h"

MainWindow::MainWindow(QWidget *parent) : KXmlGuiWindow(parent)
{
    textArea = new KTextEdit();
    setCentralWidget(textArea);
    setupGUI();
}
```

First, of course, we have to include the header file containing the class declaration.

Inside the constructor of our subclassed [KXmlGuiWindow](docs:kxmlgui;KXmlGuiWindow), we initialize our [KTextEdit](docs:ktextwidgets;KTextEdit) `textArea`. Because `textArea` derives from [QWidget](docs:qtwidgets;QWidget) and our [KXmlGuiWindow](docs:kxmlgui;KXmlGuiWindow) `MainWindow` derives from [QMainWindow](docs:qtwidgets;QMainWindow), we can call [QMainWindow::setCentralWidget()](docs:qtwidgets;QMainWindow::setCentralWidget) to make our `textArea` occupy the empty area in the central section of our window.

Finally, [KXmlGuiWindow::setupGUI()](docs:kxmlgui;KXmlGuiWindow::setupGUI) is called which does a lot of behind-the-scenes stuff and creates the default menus (Settings, Help).

#### Back to main.cpp

In order to actually run this window, we need to add a few lines in main.cpp:

```cpp
#include <QApplication>
#include <QCommandLineParser>
#include <KAboutData>
#include <KLocalizedString>
#include "mainwindow.h"

int main (int argc, char *argv[])
{
    using namespace Qt::Literals::StringLiterals;

    QApplication app(argc, argv);
    KLocalizedString::setApplicationDomain("mainwindow");

    KAboutData aboutData(
        u"mainwindow"_s,
        i18n("Main Window"),
        u"1.0"_s,
        i18n("A simple text area"),
        KAboutLicense::GPL,
        i18n("(c) 2024"),
        i18n("Educational application..."),
        u"https://apps.kde.org/someappname/"_s,
        u"submit@bugs.kde.org"_s);

    aboutData.addAuthor(
        i18n("John Doe"),
        i18n("Tutorial learner"),
        u"john.doe@example.com"_s,
        u"https://john-doe.example.com"_s,
        u"johndoe"_s);

    KAboutData::setApplicationData(aboutData);

    QCommandLineParser parser;
    aboutData.setupCommandLine(&parser);
    parser.process(app);
    aboutData.processCommandLine(&parser);

    MainWindow *window = new MainWindow();
    window->show();

    return app.exec();
}
```

We include our new header file `mainwindow.h`. This lets us create our new `MainWindow` object which we then display near the end of the main function (by default, new window objects are hidden).

### CMake

The best way to build the program is to use CMake. We add `mainwindow.cpp` to the sources list, include the `XmlGui` and `TextWidgets` frameworks, and replace all `helloworld` text with `mainwindow`.

#### CMakeLists.txt

```cpp
cmake_minimum_required(VERSION 3.20)

project(mainwindow)

set(QT_MIN_VERSION "6.6.0")
set(KF_MIN_VERSION "6.0.0")

find_package(ECM ${KF_MIN_VERSION} REQUIRED NO_MODULE)
set(CMAKE_MODULE_PATH ${ECM_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

include(KDEInstallDirs)
include(KDECMakeSettings)
include(KDECompilerSettings NO_POLICY_SCOPE)
include(FeatureSummary)

find_package(Qt6 ${QT_MIN_VERSION} CONFIG REQUIRED COMPONENTS
    Core    # QCommandLineParser, QStringLiteral
    Widgets # QApplication
)

find_package(KF6 ${KF_MIN_VERSION} REQUIRED COMPONENTS
    CoreAddons      # KAboutData
    I18n            # KLocalizedString
    XmlGui          # KXmlGuiWindow
    TextWidgets     # KTextEdit
)

add_executable(mainwindow)

target_sources(mainwindow
    PRIVATE
    main.cpp
    mainwindow.cpp
)

target_link_libraries(mainwindow
    Qt6::Widgets
    KF6::CoreAddons
    KF6::I18n
    KF6::XmlGui
    KF6::TextWidgets
)

install(TARGETS mainwindow ${KDE_INSTALL_TARGETS_DEFAULT_ARGS})

feature_summary(WHAT ALL INCLUDE_QUIET_PACKAGES FATAL_ON_MISSING_REQUIRED_PACKAGES)
```

### Running our application

You can repeat the same steps provided in the \[KXmlGui Hello World]\(\{{< ref "hello\_world#kxmlgui-running" >\}}) to build and install the application. You can then run the project with:

```bash
kdesrc-build --run --exec mainwindow kxmlgui-tutorial
```

or

```bash
mainwindow
```
