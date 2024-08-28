---
title: KIdleTime
weight: 6
group: features
SPDX-License-Identifier: CC-BY-SA-4.0
SPDX-FileCopyrightText: 2014 Andreas Cord-Landwehr <cordlandwehr@kde.org>
aliases:
  - /docs/kidletime/
description: Detect and handle system idling
---

# KIdleTime

KIdleTime is a helper framework to get reporting information on idle time of the system. It is useful not only for finding out about the current idle time of the system, but also for getting notified upon idle time events, such as custom timeouts or user activity. It features:

* current idling time
* timeout notifications, to be emitted if the system idled for a specified time
* activity notifications, if the user resumes acting after an idling period

### Using It

To understand how to use KIdleTime, we will create a small testing application, called "KIdleTest". This application initially waits for the first user action and afterwards registers some timeout intervals, acting whenever the system idles for such a time. The framework includes the singleton KIdleTime, which provides all necessary signals and information about the idling status of the system. For our example, we start by connecting to the signals for user resuming from idling and for reaching timeouts that we will set ourselves:

```cpp
KIdleTest::KIdleTest()
{
    // connect to idle events
    connect(KIdleTime::instance(), &KIdleTime::resumingFromIdle, this, &KIdleTest::resumeEvent);
    connect(KIdleTime::instance(), qOverload<int, int>(&KIdleTime::timeoutReached), this, &KIdleTest::timeoutReached);

    // register to get informed for the very next user event
    KIdleTime::instance()->catchNextResumeEvent();

    printf("Your idle time is %d\n", KIdleTime::instance()->idleTime());
    printf("Welcome!! Move your mouse or do something to start...\n");
}
```

We also tell KIdleTime to notify us the very next time when the user acts. Note that this is actually only for the next time. If we were interested in further events, we would need to invoke `catchNextResumeEvent()` again.

Next, in our event listener for the user resume event, we register a couple of idle intervals:

```cpp
void KIdleTest::resumeEvent()
{
    KIdleTime::instance()->removeAllIdleTimeouts();

    printf("Great! Now stay idle for 5 seconds to get a nice message. From 10 seconds on, you can move your mouse to get back here.\n");
    printf("If you will stay idle for too long, I will simulate your activity after 25 seconds, and make everything start back\n");

    KIdleTime::instance()->addIdleTimeout(5000);
    KIdleTime::instance()->addIdleTimeout(10000);
    KIdleTime::instance()->addIdleTimeout(25000);
}
```

If any of these idle intervals is reached, our initially registered `timeoutReached(...)` slot is invoked and we print out an appropriate message.

```cpp
void KIdleTest::timeoutReached(int id, int timeout)
{
    Q_UNUSED(id)

    if (timeout == 5000) {
        printf("5 seconds passed, stay still some more...\n");
    } else if (timeout == 10000) {
        KIdleTime::instance()->catchNextResumeEvent();
        printf("Cool. You can move your mouse to start over\n");
    } else if (timeout == 25000) {
        printf("Uff, you're annoying me. Let's start again. I'm simulating your activity now\n");
        KIdleTime::instance()->simulateUserActivity();
    } else {
        qDebug() << "OUCH";
    }
}
```

From then on, depending on the reached idle interval, we can go back to one of the former steps.
