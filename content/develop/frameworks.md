---
title: KDE Frameworks
subtitle: The KDE Frameworks are a set of 83 add-on libraries for programming with Qt.
logo: https://kde.org/stuff/clipart/logo/kde-logo-white-blue-rounded-source.svg
scss: scss/framework.scss
description: The KDE Frameworks are a set of 83 add-on libraries for programming with Qt.
---

# KDE Frameworks

### Get Started

[Developer DocumentationCode an application in C++ with Qt and QML.](https://api.kde.org/frameworks/index.html)[Design GuidelinesUse our UI standards to their fullest for a flexible and consistent user experience if you are creating an app for the Linux desktop.](https://hig.kde.org/)

## Features

The KDE Frameworks are add-on libraries for coding applications with Qt.

The individual Frameworks are well documented, tested and their API style will be familiar to Qt developers.

Frameworks are developed under the proven KDE governance model with a predictable release schedule, a clear and vendor neutral contributor process, open governance and flexible LGPL or MIT licensing.

The frameworks are cross-platform and function on Windows, Mac, Android and Linux.

![some operating systems supported by the KDE Frameworks](../products/platform-icons.png)

### Organization

The Frameworks have a clear dependency structure, divided into Categories and Tiers. The Categories refer to runtime dependencies:

Functional elements have no runtime dependencies.\
Integration designates code that may require runtime dependencies for integration depending on what the OS or platform offers.\
Solutions have mandatory runtime dependencies.

The Tiers refer to compile-time dependencies on other Frameworks.

Tier 1 Frameworks have no dependencies within Frameworks and only need Qt and other relevant libraries.\
Tier 2 Frameworks can depend only on Tier 1.\
Tier 3 Frameworks can depend on other Tier 3 Frameworks as well as Tier 2 and Tier 1.

### Highlights:

KArchive offers support for many popular compression codecs in a self-contained, featureful and easy-to-use file archiving and extracting library. Just feed it files; there's no need to reinvent an archiving function in your Qt-based application!

ThreadWeaver offers a high-level API to manage threads using job- and queue-based interfaces. It allows easy scheduling of thread execution by specifying dependencies between the threads and executing them satisfying these dependencies, greatly simplifying the use of multiple threads.

Breeze Icons. KDE Frameworks includes two icon themes for your applications. Breeze icons is a modern, recognisable theme which fits in with all form factors.

KConfig is a Framework to deal with storing and retrieving configuration settings. It features a group-oriented API. It works with INI files and XDG-compliant cascading directories. It generates code based on XML files.

KI18n adds Gettext support to applications, making it easier to integrate the translation workflow of Qt applications in the general translation infrastructure of many projects.

[Kirigami](../../frameworks/kirigami/) is a responsive UI framework for QML.

### Examples

GammaRay is a cross platform (Linux, Windows, Mac) inspection tool for Qt apps from KDAB. It uses [KSyntaxHighlighting](https://api.kde.org/frameworks/syntax-highlighting/html/index.html) to colour text it shows.

![screenshot of app GammaRay](../products/gammaray.png)

LXQt is a simple desktop environment for Linux. It uses [Solid](https://api.kde.org/frameworks/solid/html/) for hardware integration, [KWindowSystem](https://api.kde.org/frameworks/kwindowsystem/html/) for window management and [KIdleTime](https://api.kde.org/frameworks/kidletime/html/) for power management.

Good looks are added with [Oxygen Icons](https://invent.kde.org/frameworks/oxygen-icons), one of our icon theme frameworks.

![screenshot of the desktop environment LXQt](../products/lxqt.png)

## Get The Frameworks

### Latest Version

Download the latest release

[Release Announcements](https://www.kde.org/announcements/)

### Find more Qt Addons

The Qt library archive.

[Inqlude](https://inqlude.org/)

### Build it Yourself

Our tool to compile KDE software

[kdesrc-build](https://kdesrc-build.kde.org)

### Linux Packages

Linux distros already package KDE Frameworks

[Binary Packages](https://community.kde.org/Get\_KDE\_Software\_on\_Your\_Linux\_Distro)

### Windows

Our Craft tool builds KDE software on Windows and other platforms

[Craft](https://community.kde.org/Craft)

### Android

Use KDE Frameworks on mobile

[Android Tutorial](https://community.kde.org/Android)
