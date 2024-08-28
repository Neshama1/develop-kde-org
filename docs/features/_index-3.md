---
title: Archives
weight: 5
group: features
SPDX-License-Identifier: missing
SPDX-FileCopyrightText: 2014 Rohan Garg <rohan16garg@gmail.com>
aliases:
  - /docs/features/karchive/
description: KArchive, the convenient way to read and write to archives.
---

# Archives

When you are storing large amounts of data, how do you archive it in a easy way from within your code? The KArchive framework provides a quick and easy way to do this from within Qt apps.

While Qt5 provides the QZipWriter and QZipReader classes, these are limited only to Zips. KArchive on the other hand supports a wide array of formats such as p7zip, tar and ar archives, giving you the flexibility of choosing the formats which fit your project.

### Show me the code

Here's a simple 'Hello World' example of KArchive.

```cpp
    // Create a zip archive
    KZip archive(QStringLiteral("hello.zip"));

    // Open our archive for writing
    if (archive.open(QIODevice::WriteOnly)) {
        // The archive is open, we can now write data
        archive.writeFile(QStringLiteral("world"), // File name
                          QByteArray("The whole world inside a hello."), // Data
                          0100644, // Permissions
                          QStringLiteral("owner"), // Owner
                          QStringLiteral("users")); // Group

        // Don't forget to close!
        archive.close();
    }

    if (archive.open(QIODevice::ReadOnly)) {
        const KArchiveDirectory *dir = archive.directory();

        const KArchiveEntry *e = dir->entry("world");
        if (!e) {
            qDebug() << "File not found!";
            return -1;
        }
        const KArchiveFile *f = static_cast<const KArchiveFile *>(e);
        QByteArray arr(f->data());
        qDebug() << arr; // the file contents

        // To avoid reading everything into memory in one go, we can use createDevice() instead
        QIODevice *dev = f->createDevice();
        while (!dev->atEnd()) {
            qDebug() << dev->readLine();
        }
        delete dev;
    }
```

More files can be added by subsequent calls to writeFile(). You also add folders to your zip by using the writeDir call as follows :

```cpp
archive.writeDir(QStringLiteral("world dir"));
```

Full API docs can be found [here](https://api.kde.org/frameworks/karchive/html/index.html)

### Advanced usecases

#### Sending compressed data over networks

KArchive also supports reading and writing compressed data to devices such as buffers or sockets via the KCompressionDevice class allowing developers to save bandwidth while transmitting data over networks.

A quick example of the KCompressionDevice class can be summed up as:

```cpp
    // Open the input archive
    KCompressionDevice input(&file, false, KCompressionDevice::BZip2);
    input.open(QIODevice::ReadOnly);

    QString outputFile = (info.completeBaseName() + QLatin1String(".gz"));

    // Open the new output file
    KCompressionDevice output(outputFile, KCompressionDevice::GZip);
    output.open(QIODevice::WriteOnly);

    while (!input.atEnd()) {
        // Read and uncompress the data
        QByteArray data = input.read(512);

        // Write data like you would to any other QIODevice
        output.write(data);
    }

    input.close();
    output.close();
```
