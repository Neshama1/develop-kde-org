---
title: Creating a Python package
weight: 2
group: python
description: Understand the requirements to create your own Python package.
---

# Creating a Python package

### Packaging the application

To distribute the application to users we have to package it. We are going to use the [setuptools](https://pypi.org/project/setuptools/) library.

If you'd like to learn more about Python packaging, you'll be interested in the [Python Packaging User Guide](https://packaging.python.org/en/latest/guides/distributing-packages-using-setuptools/).

Let's recapitulate what the file structure of the project should be:

```
simplemdviewer
├── README.md
├── LICENSE.txt
├── MANIFEST.in                        # To add our QML file
├── pyproject.toml                     # To declare the tools used to build
├── setup.py                           # To import setuptools
├── setup.cfg                          # The setuptools metadata
├── org.kde.simplemdviewer.desktop
├── org.kde.simplemdviewer.json
├── org.kde.simplemdviewer.svg
├── org.kde.simplemdviewer.metainfo.xml
└── src/
    ├── __init__.py                    # To import the src/ directory as a package
    ├── __main__.py                    # To signal simplemdviewer_app as the entrypoint
    ├── simplemdviewer_app.py
    ├── md_converter.py
    └── qml/
        └── main.qml
```

Create a `simplemdviewer/pyproject.toml` to tell the Python build tools what is needed to build our project:

```toml
[build-system]
requires = ["setuptools", "wheel"]
build-backend = "setuptools.build_meta"
```

Create a `simplemdviewer/setup.py` to call `setuptools`:

```python
from setuptools import setup
setup()
```

Add a new `simplemdviewer/setup.cfg` to describe the application:

```ini
[metadata]
name = org.kde.simplemdviewer
version = 0.1
url = https://mydomain.org/simplemdviewer
author= Example Author
author_email = example@author.org
maintainer = Example Author
maintainer_email = example@author.org
description = A simple markdown viewer
long_description = file: README.md
long_description_content_type = text/markdown
classifiers =
    Development Status :: 5 - Production/Stable
    License :: OSI Approved :: GNU General Public License v3 or later (GPLv3+)
    Intended Audience :: End Users/Desktop
    Topic :: Utilities
    Programming Language :: Python
    Operating System :: POSIX :: Linux
keywords= viewer converter markdown

[options]
packages = simplemdviewer
package_dir =
    simplemdviewer = src
include_package_data = True
install_requires =
    markdown
```

In the `metadata` section we have provided information about the application.

The `options` section contains the project dependencies and the [import package](https://packaging.python.org/en/latest/glossary/#term-Import-Package) that our [distribution package](https://packaging.python.org/en/latest/glossary/#term-Distribution-Package) is going to provide. There is a lot of [options](https://setuptools.readthedocs.io/en/latest/userguide/declarative\_config.html) and [classifiers](https://pypi.org/classifiers/) available.

For more details on dependency management in setuptools, check [here](https://setuptools.pypa.io/en/latest/userguide/dependency\_management.html).

Create an empty `simplemdviewer/src/__init__.py` file. This file just needs to be present in order to import a directory as a package.

```bash
touch __init__.py
```

Since we are using a custom package directory, namely `src/`, we pass it as the correct `package_dir` to find our `simplemdviewer` application.

It is good to have a `README.md` file as well.

Create a `simplemdviewer/README.md`:

```markdown
# Simple Markdown Viewer

A simple Markdown viewer created with Kirigami, QML and Python.
```

Another important piece is the license of our project. Create a `simplemdviewer/LICENSE.txt` and add the text of the [license](https://www.gnu.org/licenses/gpl-3.0.txt) of our project.

Apart from the Python stuff, we have to add the QML code to the distribution package as well.

Create a `simplemdviewer/MANIFEST.in` file with the following contents:

```qml
include src/qml/*.qml
```

### App metadata

Some last pieces and we are ready to build. We are going to add:

1. The [Appstream](https://www.freedesktop.org/wiki/Distributions/AppStream/) metadata used to show the app in software stores.
2. A [Desktop Entry](https://specifications.freedesktop.org/desktop-entry-spec/desktop-entry-spec-latest.html) file to add the application to the application launcher.
3. An application icon.

Create a new `simplemdviewer/org.kde.simplemdviewer.desktop`. This file is used to show our Markdown Viewer in application menus/launchers.

```bash
[Desktop Entry]
Name=Simple Markdown Viewer
Name[ca]=Visualitzador senzill de Markdown
Name[cs]=Jednoduché prohlížení souborů Markdown
Name[eo]=Simpla Markdown-Vidilo
Name[es]=Sencillo visor de Markdown
Name[fr]=Afficheur simple pour langage « Markdown »
Name[it]=Visore Markdown semplice
Name[ja]=シンプルな Markdown ビューアー
Name[nl]=Eenvoudige Markdown-viewer
Name[sl]=Preprosti ogledovalnik Markdown
Name[sv]=Enkel Markdown-visning
Name[tr]=Basit Markdown Görüntüleyicisi
Name[uk]=Простий переглядач Markdown
Name[x-test]=xxSimple Markdown Viewerxx
Name[zh_TW]=簡單 Markdown 檢視器
GenericName=Markdown Viewer
GenericName[ca]=Visualitzador de Markdown
GenericName[cs]=Prohlížeč souborů Markdown
GenericName[eo]=Markdown-Vidilo
GenericName[es]=Visor de Markdown
GenericName[fr]=Afficheur pour langage « Markdown »
GenericName[it]=Visore Markdown
GenericName[ja]=Markdown ビューアー
GenericName[nl]=Markdown-viewer
GenericName[sl]=Ogledovalnik Markdown
GenericName[sv]=Markdown-visning
GenericName[tr]=Markdown Görüntüleyicisi
GenericName[uk]=Переглядач Markdown
GenericName[x-test]=xxMarkdown Viewerxx
GenericName[zh_TW]=Markdown 檢視器
Comment=A simple Markdown viewer application
Comment[ca]=Una aplicació senzilla de visualització de Markdown
Comment[cs]=Jednoduchá aplikace pro prohlížení souborů Markdown
Comment[eo]=Simpla Markdown-vidila aplikaĵo
Comment[es]=Una sencilla aplicación de visor de Markdown
Comment[fr]=Une application d'afficheur pour langage « Markdown »
Comment[it]=L'applicazione di un visore Markdown semplice
Comment[ja]=シンプルな Markdown ビューアーアプリケーション
Comment[nl]=Een eenvoudige toepassing als Markdown-viewer
Comment[sl]=Aplikacija preprostega ogledovalnika Markdown
Comment[sv]=Ett enkelt Markdown-visningsprogram
Comment[tr]=Basit bir Markdown görüntüleyici uygulama
Comment[uk]=Проста програма для перегляду Markdown
Comment[x-test]=xxA simple Markdown viewer applicationxx
Comment[zh_TW]=一個簡單的 Markdown 檢視器應用程式
Version=1.0
Exec=simplemdviewer
Icon=org.kde.simplemdviewer
Type=Application
Terminal=false
Categories=Office;
X-KDE-FormFactor=desktop;tablet;handset;
```

Add a new `simplemdviewer/org.kde.simplemdviewer.metainfo.xml`. This file is used to show the application in app stores.

```xml
<?xml version="1.0" encoding="utf-8"?>
<component type="desktop">
  <id>org.kde.simplemdviewer</id>
  <metadata_license>CC0-1.0</metadata_license>
  <project_license>GPL-3.0+</project_license>
  <name>Simple Markdown Viewer</name>
  <name xml:lang="ca">Visualitzador senzill de Markdown</name>
  <name xml:lang="cs">Jednoduché prohlížení souborů Markdown</name>
  <name xml:lang="eo">Simpla Markdown-Vidilo</name>
  <name xml:lang="es">Sencillo visor de Markdown</name>
  <name xml:lang="fr">Afficheur simple pour langage « Markdown »</name>
  <name xml:lang="it">Visore Markdown semplice</name>
  <name xml:lang="ja">シンプルな Markdown ビューアー</name>
  <name xml:lang="nl">Eenvoudige Markdown-viewer</name>
  <name xml:lang="sl">Enostavni ogledovalnik Markdown</name>
  <name xml:lang="sv">Enkel Markdown-visning</name>
  <name xml:lang="tr">Basit Markdown Görüntüleyicisi</name>
  <name xml:lang="uk">Простий переглядач Markdown</name>
  <name xml:lang="x-test">xxSimple Markdown Viewerxx</name>
  <name xml:lang="zh-TW">簡單 Markdown 檢視器</name>
  <summary>A simple markdown viewer application</summary>
  <summary xml:lang="ca">Una aplicació senzilla de visualització de Markdown</summary>
  <summary xml:lang="eo">Simpla markdown-spektila aplikaĵo</summary>
  <summary xml:lang="es">Una sencilla aplicación de visor markdown</summary>
  <summary xml:lang="fr">Une application d'afficheur simple pour langage « Markdown »</summary>
  <summary xml:lang="it">L'applicazione di un visore Markdown semplice</summary>
  <summary xml:lang="ja">シンプルな markdown ビューアーアプリケーション</summary>
  <summary xml:lang="nl">Een eenvoudige toepassing als Markdown-viewer</summary>
  <summary xml:lang="sl">Aplikacija enostavnega ogledovalnika Markdown</summary>
  <summary xml:lang="sv">Ett enkelt Markdown-visningsprogram</summary>
  <summary xml:lang="tr">Basit bir Markdown görüntüleyici uygulama</summary>
  <summary xml:lang="uk">Проста програма для перегляду Markdown</summary>
  <summary xml:lang="x-test">xxA simple markdown viewer applicationxx</summary>
  <summary xml:lang="zh-TW">一個簡單的 Markdown 檢視器應用程式</summary>
  <description>
    <p>Simple Markdown Viewer is a showcase application for QML with Python development</p>
    <p xml:lang="ca">El visualitzador senzill de Markdown és una aplicació per a presentar el desenvolupament del QML amb el Python</p>
    <p xml:lang="eo">Simple Markdown Viewer estas montra aplikaĵo por QML kun Python-evoluo</p>
    <p xml:lang="es">Sencillo visor de Markdown es una aplicación de demostración para QML con desarrollo en Python</p>
    <p xml:lang="fr">Un afficheur simple pour langage « Markdown » est une application majeure pour QML pour les développement sous Python</p>
    <p xml:lang="it">Visore Markdown semplice è un'applicazione dimostrativa per QML nello sviluppo in Python</p>
    <p xml:lang="ja">シンプルな Markdown ビューアーは Python 開発による QML のサンプルアプリケーションです。</p>
    <p xml:lang="nl">Eenvoudige Markdown-viewer is een uitstelkast voor QML met Python ontwikkeling</p>
    <p xml:lang="sl">Enostavni ogledovalnik Markdown je predstavitvena aplikacija za razvoj QML s Pythonom</p>
    <p xml:lang="sv">Enkel Markdown-visning är ett förevisningsprogram för QML med Python-utveckling</p>
    <p xml:lang="tr">Basit Markdown Görüntüleyicisi, Python ile QML geliştirmek için bir vitrin uygulamadır</p>
    <p xml:lang="uk">Проста програма для перегляду Markdown є прикладом програми для розробки з QML за допомогою Python</p>
    <p xml:lang="x-test">xxSimple Markdown Viewer is a showcase application for QML with Python developmentxx</p>
    <p xml:lang="zh-TW">《簡單 Markdown 檢視器》是用來示範 QML 與 Python 併用開發的應用程式</p>
  </description>
  <url type="homepage">https://mydomain.org/simplemdviewer</url>
  <releases>
    <release version="0.1" date="2022-02-25">
      <description>
        <p>First release</p>
      </description>
    </release>
  </releases>
  <provides>
    <binary>simplemdviewer</binary>
  </provides>
</component>
```

For this tutorial the well known Markdown icon is okay.

Get the [Markdown](https://en.wikipedia.org/wiki/Markdown#/media/File:Markdown-mark.svg) icon and save it as `simplemdviewer/org.kde.simplemdviewer.svg`.

We need the icon to be perfectly squared, which can be accomplished with [Inkscape](https://inkscape.org):

1. Open `Markdown-mark.svg` in Inkscape.
2. Type Ctrl+a to select everything.
3. On the top `W:` text field, type 128 and press Enter.
4. Go to File -> Document Properties...
5. Change the `Width:` text field to 128 and press Enter.
6. Save the file.

Now we have to let `setup.cfg` know about the new files. Let’s also provide an easy way to open the application from the console by just typing `simplemdviewer`.

Update `simplemdviewer/setup.cfg` to:

```ini
[metadata]
name = org.kde.simplemdviewer
version = 0.1
url = https://mydomain.org/simplemdviewer
author= Example Author
author_email = example@author.org
maintainer = Example Author
maintainer_email = example@author.org
description = A simple markdown viewer
long_description = file: README.md
long_description_content_type = text/markdown
classifiers =
    Development Status :: 5 - Production/Stable
    License :: OSI Approved :: GNU General Public License v3 or later (GPLv3+)
    Intended Audience :: End Users/Desktop
    Topic :: Utilities
    Programming Language :: Python
    Operating System :: POSIX :: Linux
keywords= viewer converter markdown

[options]
packages = simplemdviewer
package_dir =
    simplemdviewer = src
include_package_data = True
install_requires =
    markdown

[options.data_files]
share/applications =
    org.kde.simplemdviewer.desktop
share/icons/hicolor/scalable/apps =
    org.kde.simplemdviewer.svg
share/metainfo =
     org.kde.simplemdviewer.metainfo.xml

[options.entry_points]
console_scripts =
    simplemdviewer = simplemdviewer.simplemdviewer_app:main
```

The last step is to tinker with the way we import modules.

Update `simplemdviewer/src/simplemdviewer_app.py` to

{% tabs %}
{% tab title="PySide6" %}
```python
#!/usr/bin/env python3

import os
import sys
import signal
from PySide6.QtGui import QGuiApplication
from PySide6.QtCore import QUrl
from PySide6.QtQml import QQmlApplicationEngine

# from md_converter import MdConverter
from simplemdviewer.md_converter import MdConverter  # noqa: F401


def main():
    """Initializes and manages the application execution"""
    app = QGuiApplication(sys.argv)
    engine = QQmlApplicationEngine()

    """Needed to close the app with Ctrl+C"""
    signal.signal(signal.SIGINT, signal.SIG_DFL)

    """Needed to get proper KDE style outside of Plasma"""
    if not os.environ.get("QT_QUICK_CONTROLS_STYLE"):
        os.environ["QT_QUICK_CONTROLS_STYLE"] = "org.kde.desktop"

    base_path = os.path.abspath(os.path.dirname(__file__))
    url = QUrl(f"file://{base_path}/qml/main.qml")
    engine.load(url)

    if len(engine.rootObjects()) == 0:
        quit()

    app.exec()


if __name__ == "__main__":
    main()
```
{% endtab %}

{% tab title="PyQt6" %}

{% endtab %}
{% endtabs %}

Create a `__main__.py` file into the `src/` directory. Now that there's a module, this tells the build tools what's the main function, the entrypoint for running the application.

```python
from . import simplemdviewer_app

simplemdviewer_app.main()
```

### Calling the app

This last step will facilitate the execution of the package during development and let us call the application by its name.

From inside the `simplemdviewer/` directory, install the app locally as a module and try running it:

```bash
python3 -m pip install -e .
python3 -m simplemdviewer
```

If you have put the required files in the right places, running the application as a module should work.

It’s time to generate the package for our program.

Make sure that the latest version of `build` is installed:

```bash
python3 -m pip install --upgrade build
```

Execute from inside the `simplemdviewer/` directory:

```bash
python3 -m build
```

As soon as the build completes, two archives will be created in the `dist/` directory:

1. the `org.kde.simplemdviewer-0.1.tar.gz` source archive
2. the `org.kde.simplemdviewer-0.1-py3-none-any.whl` package ready for distribution in places such as PyPI

Install the newly-created package into the virtual environment:

```bash
python3 -m pip install dist/org.kde.simplemdviewer-0.1.tar.gz
```

Run:

```bash
simplemdviewer
```

At this point we can tag and release the source code. Linux distributions will package it and the application will be added to their software repositories.

Well done.
