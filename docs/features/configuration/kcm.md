---
title: Settings module (KCM) development
weight: 3
aliases:
  - /docs/features/configuration/kcm/
description: This tutorial will help you create a Plasma configuration module.
---

# Settings module (KCM) development

### Introduction

Settings in Plasma are provided by KDE Configuration Modules (KCM). These can be loaded by multiple wrapper applications such as `systemsettings` on the desktop, `plasma-settings` on mobile or `kcmshell6` for standalone config windows. The source code for the different modules is split across different locations, such as [plasma-workspace](https://invent.kde.org/plasma/plasma-workspace/-/tree/master/kcms) or [plasma-desktop](https://invent.kde.org/plasma/plasma-desktop/-/tree/master/kcms).

You can query the available KCMs using `kcmshell6 --list`. To load an individual module in a standalone window pass its name to the wrapper application, e.g. `systemsettings kcm_accounts`, `plasma-settings -m kcm_kaccounts`, or `kcmshell6 kcm_kaccounts`.

### Basic KCM

KCMs consist of a KPackage holding the QML UI and a C++ library holding the logic. Some legacy KCMs are based on QtWidgets, however this is not recommended for new KCMs and it's not possible to load these in `plasma-settings`. In Plasma, new KCMs should be built using QML and [Kirigami](docs:kirigami2).

As an example, we are going to create a time settings module that allows us to configure the time in our system. The basic structure of this hypothetical time settings module is the following:

```
...
├── CMakeLists.txt
└── src
    ├── CMakeLists.txt
    ├── timesettings.cpp
    ├── timesettings.h
    ├──  kcm_time.json
    └── ui
        └── main.qml
```

### CMakeLists.txt

There are two CMake files here, one inside `src` folder and one in the root folder.

#### ./CMakeLists.txt

```cmake
# SPDX-FileCopyrightText: Year Author <email@company.com>
#
# SPDX-License-Identifier: BSD-2-Clause

cmake_minimum_required(VERSION 3.16)

set(QT_MIN_VERSION "6.5.0")
set(KF6_MIN_VERSION "6.0.0")
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED TRUE)

project(kcm_time)

find_package(ECM ${KF6_MIN_VERSION} REQUIRED NO_MODULE)

set(CMAKE_MODULE_PATH ${ECM_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake )

include(KDEInstallDirs)
include(KDECompilerSettings NO_POLICY_SCOPE)
include(KDECMakeSettings)

find_package(Qt6 ${QT_MIN_VERSION} CONFIG REQUIRED Core Quick Gui Qml)

find_package(KF6 ${KF6_MIN_VERSION} REQUIRED COMPONENTS
    Config
    CoreAddons
    KCMUtils
    I18n
)

add_subdirectory(src)
```

The `CMakeLists.txt` file in the root folder declares the project and prepares the KCM for building. This is only needed if you build the KCM alone, but usually KCM is included with the project it's for. Therefore it's not necessarily needed, but you need to make sure the project includes the libraries its KCM needs as well.

#### ./src/CMakeLists.txt

```cmake
# SPDX-FileCopyrightText: Year Author <email@company.com>
#
# SPDX-License-Identifier: BSD-2-Clause

kcmutils_add_qml_kcm(kcm_time)
target_sources(kcm_time PRIVATE
   timesettings.cpp
   timesettings.h
)
target_link_libraries(kcm_time PRIVATE
    Qt::Quick
    KF6::CoreAddons
    KF6::KCMUtils
    KF6::I18n
)
```

These CMake files contain a few packages of note: [KCMUtils](docs:kcmutils) provides various classes that allow us to work with [KCModules](docs:kconfigwidgets;KCModule), and `Config` includes the [KConfig](docs:kconfig) classes. You are likely to have seen most of the other packages elsewhere in this documentation; if not, [you can read this page](../../getting-started/kirigami/advanced-understanding\_cmakelists/) which goes through a similar CMakeLists file line by line.

What's different here is that we are using C++ code as a plugin for our QML code. This is why we don't have a `main.cpp`: we only need the class that will provide the backend functionality for our KCM. `kcoreaddons_add_plugin` creates such a plugin and installs it to the right location.

### timesettings.h

```cpp
/**
 * SPDX-FileCopyrightText: Year Author <author@domain.com>
 * SPDX-License-Identifier: GPL-2.0-or-later
 */

#pragma once

#include <KQuickConfigModule>

class TimeSettings : public KQuickConfigModule
{
    Q_OBJECT
    public:
        TimeSettings(QObject *parent, const KPluginMetaData &data);
};
```

Here we are defining the class we will be using for our KCM. [KQuickConfigModule](docs:kcmutils;classKQuickConfigModule.html) serves as the base class for all QML-based KCMs. You can read the linked API documentation to get a full description, and the KConfigXT page goes into more detail about how KConfigXT works.

### timesettings.cpp

```cpp
/**
 * SPDX-FileCopyrightText: Year Author <author@domain.com>
 * SPDX-License-Identifier: GPL-2.0-or-later
 */

#include "timesettings.h"

#include <KPluginFactory>

K_PLUGIN_CLASS_WITH_JSON(TimeSettings, "kcm_time.json")

TimeSettings::TimeSettings(QObject *parent, const KPluginMetaData &data)
    : KQuickConfigModule(parent, data)
{
    setButtons(Help | Apply | Default);
}

#include "timesettings.moc"
```

Here is where we would put any backend code that changes things behind the scenes. We can expose functions in here to our QML so that elements of our UI can be used to trigger backend functionality. This C++ code is then installed in our system on compilation in a KCM plugin directory, where upon execution the KCM can access and use it.

In this C++ file we define the constructor for the class in the previous file. We include some basic metadata about this KCM and we provide the buttons that we will want included in our window.

### package/kcm\_time.json

```json
{
    "KPlugin": {
        "Description": "Configure Time",
        "Description[ca]": "Configura la data i l'hora",
        "Description[cs]": "Nastavit čas",
        "Description[de]": "Zeit einrichten",
        "Description[eo]": "Agordu Tempon",
        "Description[es]": "Configurar la hora",
        "Description[fi]": "Ajan asetukset",
        "Description[fr]": "Configurer l'heure",
        "Description[gl]": "Configurar a hora.",
        "Description[it]": "Configura orario",
        "Description[ja]": "設定時間",
        "Description[ko]": "시간 설정",
        "Description[nl]": "Tijd configureren",
        "Description[pl]": "Nastaw czas",
        "Description[pt]": "Configurar a Hora",
        "Description[sk]": "Nastaviť čas",
        "Description[sl]": "Nastavi čas",
        "Description[sv]": "Anpassa tid",
        "Description[tr]": "Zamanı Yapılandır",
        "Description[uk]": "Налаштування часу",
        "Description[x-test]": "xxConfigure Timexx",
        "Description[zh_CN]": "配置时间",
        "Description[zh_TW]": "設定時間",
        "FormFactors": [
            "desktop",
            "handset",
            "tablet",
            "mediacenter"
        ],
        "Icon": "preferences-system-time",
        "Name": "Time",
        "Name[ca]": "Data i hora",
        "Name[cs]": "Čas",
        "Name[de]": "Zeit",
        "Name[eo]": "Tempo",
        "Name[es]": "Hora",
        "Name[fi]": "Aika",
        "Name[fr]": "Heure",
        "Name[gl]": "Hora",
        "Name[it]": "Orario",
        "Name[ja]": "時間",
        "Name[ko]": "시간",
        "Name[nl]": "Tijd",
        "Name[pl]": "Czas",
        "Name[pt]": "Hora",
        "Name[sk]": "Čas",
        "Name[sl]": "Čas",
        "Name[sv]": "Tid",
        "Name[tr]": "Zaman",
        "Name[uk]": "Час",
        "Name[x-test]": "xxTimexx",
        "Name[zh_CN]": "时间",
        "Name[zh_TW]": "時間"
    },
    "X-KDE-Keywords": "Time,Date,Clock",
    "X-KDE-Keywords[ca]": "hora,data,rellotge",
    "X-KDE-Keywords[cs]": "Čas,Datum,Hodiny",
    "X-KDE-Keywords[de]": "Zeit,Datum,Uhr",
    "X-KDE-Keywords[es]": "Hora,Fecha,Reloj",
    "X-KDE-Keywords[eu]": "Ordua,Data,Erlojua",
    "X-KDE-Keywords[fi]": "Aika,Kellonaika,Päiväys,Päivämäärä,Kello",
    "X-KDE-Keywords[fr]": "Heure, Date, Horloge",
    "X-KDE-Keywords[it]": "Ora,data,orologio",
    "X-KDE-Keywords[nl]": "Tijd,datum,klok",
    "X-KDE-Keywords[pl]": "Czas,Data,Zegar",
    "X-KDE-Keywords[pt]": "Hora,Data,Relógio",
    "X-KDE-Keywords[pt_BR]": "Hora,Data,Relógio",
    "X-KDE-Keywords[sk]": "Čas,dátum,hodiny",
    "X-KDE-Keywords[sl]": "Čas,Datum,Ura",
    "X-KDE-Keywords[sv]": "Tid,Datum,Klocka",
    "X-KDE-Keywords[uk]": "час,дата,годинник",
    "X-KDE-Keywords[x-test]": "xxTimexx,xxDatexx,xxClockxx",
    "X-KDE-Keywords[zh_CN]": "Time,Date,Clock,时间,日期,时钟",
    "X-KDE-System-Settings-Parent-Category": "personalization"
}
```

This `.json` file provides metadata about our KCM. These entries specify the following:

* `Name` defines the name of the KCM which is shown in the settings app.
* `Description` is a short, one sentence description of the module.
* `FormFactors` defines on which kinds of devices this KCM should be shown.
* `X-KDE-System-Settings-Parent-Category` defines the category systemsettings5 is showing the module in.
* `X-KDE-Keywords` defines Keywords used for searching modules.

### package/contents/ui/main.qml

```json
/**
 * SPDX-FileCopyrightText: Year Author <author@domain.com>
 * SPDX-License-Identifier: GPL-2.0-or-later
 */


import QtQuick
import QtQuick.Controls as Controls

import org.kde.kirigami as Kirigami
import org.kde.kcmutils as KCMUtils

KCMUtils.SimpleKCM {
    Controls.Label {
        text: i18n("Time settings example")
    }
}
```

As you can see, this is a very basic KCM QML file. We have used a [SimpleKCM](docs:kdeclarative;org::kde::kcm::SimpleKCM) component as the root component, and we have just included a label inside here.

More complex layouts will require using a different root component. Each has its own use:

* Use [ScrollViewKCM](docs:kdeclarative;org::kde::kcm::ScrollViewKCM) for content that is vertically scrollable, such as ListView.
* Use [GridViewKCM](docs:kdeclarative;org::kde::kcm::GridViewKCM) for arranging selectable items in a grid.
* Use [SimpleKCM](docs:kdeclarative;org::kde::kcm::SimpleKCM) otherwise.

{% hint style="info" %}
Note KCMs can consist of multiple pages that are dynamically opened and closed. To push another page to the page-stack, we can use:

```js
kcm.push("AnotherPage.qml")
```

AnotherPage.qml should have one of the aforementioned KCM component types as the root element.

To pop a page (remove the last page on the page-stack) you can just use:

```js
kcm.pop()
```
{% endhint %}

### Run it!

All we need to do now is compile and run our KCM. In this case, we are installing our KCM to `~/kde/usr`, a non-standard location, so that we don't risk messing up anything on our local environment.

```bash
// Configure our project in an out-of-tree build/ folder
cmake -B build/ -DCMAKE_INSTALL_PREFIX=~/kde/usr
// Compile the project inside the build/ folder
cmake --build build/
// Install the files compiled in build/ into the ~/kde/usr prefix
cmake --install build/
```

Now that our KCM is installed, we can run it (that is, so long as we have executed `source prefix.sh`, which includes our non-standard `~/kde/usr/` location in our current environment).

```bash
source build/prefix.sh
kcmshell6 kcm_time
```

\{{< figure src="../screenshot-kcm.png" caption="The Time KCM running." >\}}
