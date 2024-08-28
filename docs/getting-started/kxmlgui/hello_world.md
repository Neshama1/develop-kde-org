---
title: Hello World!
linkTitle: Hello World!
weight: 1
aliases:
  - /docs/getting-started/hello_world/
description: Your first window using KDE Frameworks
---

# Hello World!

### Abstract

Your first program shall greet the world with a friendly "Hello World". For that, we will use a [KMessageBox](docs:kwidgetsaddons;KMessageBox) and customize one of its buttons.

{% hint style="info" %}
Note To get more information about any class you come across, you can use \[KDE's API Reference site]\(https://api.kde.org/index.html). It can be quickly accessed via KRunner with the \`kde:\` search keyword (e.g. \`kde: KMessageBox\`). You may also find it useful to consult Qt's documentation with \`qt:\`, since much of KDE's Frameworks builds upon it.
{% endhint %}

### Preparation

We are going to discuss some basic code, and in the final section we will build it. You will need to set up your development environment (so that you can use the KDE Frameworks) first. You can do that in two ways.

Create a folder `~/kde/src/kxmlgui-tutorial`. In that folder you will place the source code files from this tutorial.

#### Option 1: Using kdesrc-build

Go through the \[setting up your development environment]\(\{{< ref "building" >\}}) part of the _Get Involved_ documentation. That will give you the necessary development tools and underlying libraries, and build the KDE Frameworks from scratch.

In your `~/.config/kdesrc-buildrc` add the following:

```
module kxmlgui-tutorial
  no-src
end module
```

#### Option 2: Manually

| [Manjaro](https://software.manjaro.org/package/kcoreaddons), [Arch](https://archlinux.org/packages/?q=kcoreaddons) | <pre><code>sudo pacman -S kcoreaddons ki18n kxmlgui ktextwidgets kconfigwidgets kwidgetsaddons kio kiconthemes
</code></pre>                                                                                      |
| ------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [OpenSUSE](https://software.opensuse.org/package/kf6-kcoreaddons-devel)                                            | <pre><code>sudo zypper install kf6-kcoreaddons-devel kf6-ki18n-devel kf6-kxmlgui-devel kf6-ktextwidgets-devel kf6-kconfigwidgets-devel kf6-kwidgetsaddons-devel kf6-kio-devel kf6-kiconthemes-devel
</code></pre> |
| [Fedora](https://packages.fedoraproject.org/pkgs/kf6-kcoreaddons-devel/kf6-kcoreaddons-devel/)                     | <pre><code>sudo dnf install kf6-kcoreaddons-devel kf6-ki18n-devel kf6-kxmlgui-devel kf6-ktextwidgets-devel kf6-kconfigwidgets-devel kf6-kwidgetsaddons-devel kf6-kio-devel kf6-kiconthemes-devel
</code></pre>    |

### The Code

#### Hello World

All the code we need will be in one file, `main.cpp`. We'll start simple and increment our file as we go further. Create it with the code below:

```cpp
#include <QApplication>
#include <KMessageBox>

int main (int argc, char *argv[])
{
    QApplication app(argc, argv);

    KGuiItem primaryAction(
        // Content text, Icon
        QStringLiteral("Hello"), QString(),
        // Tooltip text
        QStringLiteral("This is a tooltip"),
        // WhatsThis text
        QStringLiteral("This is a WhatsThis help text."));

    auto messageBox = KMessageBox::questionTwoActions(
        // Parent
        nullptr,
        // MessageBox contents
        QStringLiteral("Hello World"),
        // MessageBox title
        QStringLiteral("Hello Title"),
        // Primary action, Secondary action
        primaryAction, KStandardGuiItem::cancel());

    if (messageBox == KMessageBox::PrimaryAction)
    {
        return EXIT_SUCCESS;
    }
    else
    {
        return EXIT_FAILURE;
    }
}
```

![](../../../content/docs/getting-started/kxmlgui/hello\_world/hello\_world.webp)

Our popup box is a [KMessageBox](docs:kwidgetsaddons;KMessageBox), which has primarily two buttons: a [PrimaryAction](docs:kwidgetsaddons;KMessageBox::PrimaryAction), which usually serves as a confirmation button, and a [SecondaryAction](docs:kwidgetsaddons;KMessageBox::SecondaryAction), which usually portrays a different action, like a cancel or discard button. The popup box uses the KMessageBox class, the primary action uses a custom [KGuiItem](docs:kwidgetsaddons;KGuiItem) with the text "Hello", and the secondary action uses [KStandardGuiItem::cancel()](docs:kwidgetsaddons;KStandardGuiItem::cancel).

First we need to create a [QApplication](docs:qtwidgets;QApplication) object. It needs to be created exactly once and before any other KDE Frameworks or Qt object, as it is the starting point for creating your application and thus required for other components, like [Ki18n](docs:ki18n) for translations.

The first argument of the [KGuiItem](docs:kwidgetsaddons;KGuiItem) constructor is the text that will appear on the item (in our case, a button object to be used soon). Then we have the option to set an icon for the button, but for now we don't want one so we can pass an empty [QString](docs:qtcore;QString) using `QString()`. We then set the tooltip (the text that appears when you hover over an item), and finally the "What's This?" text (accessed through right-clicking or Shift-F1).

Now that we have the item needed for our primary action button, we can create our popup with [KMessageBox::questionTwoActions()](docs:kwidgetsaddons;KMessageBox::questionTwoActions). The first argument is the parent widget of the [KMessageBox](docs:kwidgetsaddons;KMessageBox), which is not needed for us here, so we pass `nullptr`. The second argument is the text that will appear inside the message box and above the buttons, in our case, "Hello World". The third is the caption shown in the window's titlebar, "Hello Title". Then, we set our custom [KGuiItem](docs:kwidgetsaddons;KGuiItem), `primaryAction`. Lastly, we add a convenience object with [KStandardGuiItem::cancel()](docs:kwidgetsaddons;KStandardGuiItem::cancel), which returns a ready-made [KGuiItem](docs:kwidgetsaddons;KGuiItem) with localized text and cancel functionality, satisfying the function signature.

{% hint style="warning" %}
Important

Using a QStringLiteral for strings like `QStringLiteral("Hello World!")` instead of literals like `"Hello World!"` is both a best practice in Qt programming and an expected coding practice in KDE software.
{% endhint %}

#### About and Internationalization

```cpp
#include <QApplication>
#include <KMessageBox>
#include <KAboutData>
#include <KLocalizedString>

int main (int argc, char *argv[])
{
    using namespace Qt::Literals::StringLiterals;

    QApplication app(argc, argv);
    KLocalizedString::setApplicationDomain("tutorial1");

    KAboutData aboutData(
        // The program name used internally. (componentName)
        u"helloworld"_s,
        // A displayable program name string. (displayName)
        i18n("Hello World tutorial"),
        // The program version string. (version)
        u"1.0"_s,
        // Short description of what the app does. (shortDescription)
        i18n("Displays a KMessageBox popup"),
        // The license this code is released under
        KAboutLicense::GPL,
        // Copyright Statement (copyrightStatement = QString())
        i18n("(c) 2024"),
        // Optional text shown in the About box.
        // Can contain any information desired. (otherText)
        i18n("Educational application..."),
        // The program homepage string. (homePageAddress = QString())
        u"https://apps.kde.org/someappname/"_s,
        // The bug report email address
        // (bugsEmailAddress = QLatin1String("submit@bugs.kde.org")
        u"submit@bugs.kde.org"_s);

    aboutData.addAuthor(
        // Author full name
        i18n("John Doe"),
        // Author role
        i18n("Tutorial learner"),
        // Author email
        u"john.doe@example.com"_s,
        // Author website
        u"https://john-doe.example.com"_s,
        // Username in store.kde.org (for avatar image)
        u"johndoe"_s);

    KAboutData::setApplicationData(aboutData);

    KGuiItem primaryAction(
        i18n("Hello"), QString(),
        i18n("This is a tooltip"),
        i18n("This is a WhatsThis help text."));

    auto messageBox = KMessageBox::questionTwoActions(
        nullptr,
        i18n("Hello World"),
        i18n("Hello Title"),
        primaryAction, KStandardGuiItem::cancel());

    if (messageBox == KMessageBox::PrimaryAction)
    {
        return EXIT_SUCCESS;
    }
    else
    {
        return EXIT_FAILURE;
    }
}
```

![](../../../content/docs/getting-started/kxmlgui/hello\_world/hello\_world\_complete.webp)

For your application to be localized, we must first prepare our code so that it can be adapted to various languages and regions without engineering changes: this process is called [internationalization](https://doc.qt.io/qt-6/internationalization.html). KDE uses [Ki18n](docs:ki18n) for that, which provides [KLocalizedString](docs:ki18n;KLocalizedString).

We start with a call to [KLocalizedString::setApplicationDomain()](docs:ki18n;KLocalizedString::setApplicationDomain), which is required to properly set the translation catalog and must be done before everything else (except the creation of the [QApplication](docs:qtwidgets;QApplication) instance). After that, we can just start enveloping the relevant user-visible, translatable strings with `i18n()`. The non-user visible strings that do not need to be translated should use a [QStringLiteral](docs:qtcore;QString::QStringLiteral) instead. We'll use those next with [KAboutData](docs:kcoreaddons;KAboutData).

More information on internalization can be found in the [programmer's guide for internationalization](https://api.kde.org/frameworks/ki18n/html/prg\_guide.html).

{% hint style="warning" %}
Important

Since we are about to use many string arguments, instead of writing `QStringLiteral()` 7 times, we can use QString's [operator""\_s](https://doc.qt.io/qt-6/qstring.html#operator-22-22\_s), a shorter notation for [string literals](https://en.cppreference.com/w/cpp/language/string\_literal) special to Qt that does the same thing. This is also where the `using namespace Qt::Literals::StringLiterals;` comes from.

So instead of `QStringLiteral("Hello World!")`, just typing `u"Hello World!"_s` is enough.
{% endhint %}

[KAboutData](docs:kcoreaddons;KAboutData) is a core KDE Frameworks component that stores information about an application, which can then be reused by many other KDE Frameworks components. We instantiate a new [KAboutData](docs:kcoreaddons;KAboutData) object with its fairly complete default constructor and add author information. After all the required information has been set, we call [KAboutData::setApplicationData()](docs:kcoreaddons;KAboutData::setApplicationData) to initialize the properties of the [QApplication ](docs:qtwidgets;QApplication)object.

Note how the message box adapts to its own contents, and how the window title now includes "Tutorial 1", like we set using [KAboutData](docs:kcoreaddons;KAboutData). This property can then be accessed with [KAboutData::displayName()](docs:kcoreaddons;KAboutData::displayName) when needed.

One more thing of note is that, if you are using a different system language, the [KStandardGuiItem::cancel()](docs:kwidgetsaddons;KStandardGuiItem::cancel) button we created will likely already show up in your language instead of saying "Cancel".

#### Command line

```cpp
#include <QApplication>
#include <QCommandLineParser>
#include <KMessageBox>
#include <KAboutData>
#include <KLocalizedString>

int main (int argc, char *argv[])
{
    using namespace Qt::Literals::StringLiterals;

    QApplication app(argc, argv);
    KLocalizedString::setApplicationDomain("tutorial1");

    KAboutData aboutData(
        u"helloworld"_s,
        i18n("Hello World tutorial"),
        u"1.0"_s,
        i18n("Displays a KMessageBox popup"),
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

    KGuiItem primaryAction(
        i18n("Hello"), QString(),
        i18n("This is a tooltip"),
        i18n("This is a WhatsThis help text."));

    auto messageBox = KMessageBox::questionTwoActions(
        nullptr,
        i18n("Hello World"),
        i18n("Hello Title"),
        primaryAction, KStandardGuiItem::cancel());

    if (messageBox == KMessageBox::PrimaryAction)
    {
        return EXIT_SUCCESS;
    }
    else
    {
        return EXIT_FAILURE;
    }
}
```

Then we come to [QCommandLineParser](docs:qtcore;QCommandLineParser). This is the class one would use to specify command line flags to open your program with a specific file, for instance. However, in this tutorial, we simply initialize it with the [KAboutData](docs:kcoreaddons;KAboutData) object we created before so we can use the `--version` or `--author` flags that are provided by default by Qt.

We're all done as far as the code is concerned. Now to build it and try it out.

### Compiling and running the project <a href="#kxmlgui-running" id="kxmlgui-running"></a>

In order to run our project, we need a build system in place to compile and link the required libraries; for that, we use the industry standard CMake, together with files in our project folder called `CMakeLists.txt`.

#### Writing a CMakeLists.txt

Create a file named `CMakeLists.txt` in the same directory as `main.cpp` with this content:

```cmake
cmake_minimum_required(VERSION 3.20)

project(helloworld)

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
    WidgetsAddons   # KMessageBox
)

add_executable(helloworld)

target_sources(helloworld
    PRIVATE
    main.cpp
)

target_link_libraries(helloworld
    Qt6::Widgets
    KF6::CoreAddons
    KF6::I18n
    KF6::WidgetsAddons
)

install(TARGETS helloworld ${KDE_INSTALL_TARGETS_DEFAULT_ARGS})

feature_summary(WHAT ALL INCLUDE_QUIET_PACKAGES FATAL_ON_MISSING_REQUIRED_PACKAGES)
```

The [`find_package()`](https://cmake.org/cmake/help/latest/command/find\_package.html) function locates the package that you ask it for (in this case ECM, Qt6, or KF6) and sets some variables describing the location of the package's headers and libraries. [ECM](https://api.kde.org/ecm/), or Extra CMake Modules, is required to import special CMake files and functions for building KDE applications.

Here we try to find the modules for Qt 6 and KDE Frameworks 6 required to build our tutorial. The necessary files are included by CMake so that the compiler can see them at build time. Minimum version numbers are set at the very top of `CMakeLists.txt` file for easier reference.

Then we use [`add_executable()`](https://cmake.org/cmake/help/latest/command/add\_executable.html) to create an executable called `helloworld`. Afterwards, we link our executable to the necessary libraries using the [`target_link_libraries()`](https://cmake.org/cmake/help/latest/command/target\_link\_libraries.html) function. The [`install()`](https://cmake.org/cmake/help/latest/command/install.html) function call creates a default "install" target, putting executables and libraries in the default path using a convenience macro `KDE_INSTALL_TARGETS_DEFAULT_ARGS` provided by ECM. Additionally, just by including ECM, an "uninstall" target automatically gets created based on this "install" target.

#### Compiling and running with kdesrc-build

Compile your project by running the following command in a terminal:

```bash
kdesrc-build kxmlgui-tutorial
```

You can then run the application with:

```bash
kdesrc-build --run --exec helloworld kxmlgui-tutorial
```

#### Compiling and running manually

Change directories to the project's root folder, then run the following command in a terminal:

```bash
cmake -B build/
cmake --build build/ --parallel
cmake --install build/ --prefix "~/.local"
```

Each line above matches a step of the compilation process: the configuration, build, and install steps.

The `--parallel` flag lets CMake compile multiple files at the same time, and the `--prefix` flag tells CMake where it will be installed. In this case, the executable `helloworld` will be installed to `~/.local/bin/helloworld`.

You can then run the application with:

```bash
helloworld
```

You can also run the binary with flags. The flag `--help` is a standard flag added by Qt via [QCommandLineParser](docs:qtcore;QCommandLineParser), and the content of the `--version`, `--author` and `--license` flags should match the information we added with [KAboutData](docs:kcoreaddons;KAboutData).

```bash
kdesrc-build --run --exec helloworld kxmlgui-tutorial --help
kdesrc-build --run --exec helloworld kxmlgui-tutorial --version
kdesrc-build --run --exec helloworld kxmlgui-tutorial --author
kdesrc-build --run --exec helloworld kxmlgui-tutorial --license
```

or

```bash
helloworld --help
helloworld --version
helloworld --author
helloworld --license
```
