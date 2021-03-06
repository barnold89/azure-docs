---
title: Azure Media Services LiveEvent and a cloud DVR | Microsoft Docs
description: This article explains what LiveOutput is and how to use a cloud DVR.  
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''

ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: article
ms.date: 01/14/2019
ms.author: juliako

---

# Using a cloud DVR

A [LiveOutput](https://docs.microsoft.com/rest/api/media/liveoutputs) enables you to control the properties of the outgoing live stream, such as how much of the stream is recorded (for example, the capacity of the cloud DVR), and whether or not viewers can start watching the live stream. The relationship between a **LiveEvent** and its **LiveOutput**s is similar to traditional television broadcast, whereby a channel (**LiveEvent**) represents a constant stream of video and a recording (**LiveOutput**) is scoped to a specific time segment (for example, evening news from 6:30PM to 7:00PM). You can record television using a Digital Video Recorder (DVR) – the equivalent feature in LiveEvents is managed via the ArchiveWindowLength property. It is an ISO-8601 timespan duration (for example, PTHH:MM:SS), which specifies the capacity of the DVR, and can be set from a minimum of 3 minutes to a maximum of 25 hours.

## LiveOutput

The **ArchiveWindowLength** value determines how far back in time a viewer can seek from the current live position.  **ArchiveWindowLength** also determines how long the client manifests can grow.

Suppose you are streaming a football game, and it has an **ArchiveWindowLength** of only 30 minutes. A viewer who starts watching your event 45 minutes after the game started can seek back to at most the 15-minute mark. Your **LiveOutput**s for the game will continue until the **LiveEvent** is stopped, but content that falls outside of **ArchiveWindowLength** is continuously discarded from storage and is non-recoverable. In this example, the video between the start of the event and the 15-minute mark would have been purged from your DVR and from the container in blob storage for the [Asset](https://docs.microsoft.com/rest/api/media/assets). The archive is not recoverable and is removed from the container in Azure blob storage.

Each **LiveOutput** is associated with an **Asset**, which it uses to record the video into the associated Azure blob storage container. To publish the LiveOutput, you must create a **StreamingLocator** for that **Asset**. After creating a [StreamingLocator](https://docs.microsoft.com/rest/api/media/streaminglocators), you can build a streaming URL that you can provide to your viewers.

A **LiveEvent** supports up to three concurrently running **LiveOutput**s so you can create at most 3 recordings/archives from one live stream. This allows you to publish and archive different parts of an event as needed. Suppose you need to broadcast a 24x7 live linear feed, and create "recordings" of the different programs throughout the day to offer to customers as on-demand content for catch-up viewing. For this scenario, you first create a primary LiveOutput, with a short archive window of 1 hour or less – this is the primary live stream that your viewers would tune into. You would create a **StreamingLocator** for this **LiveOutput** and publish it to your application or web site as the "Live" feed. While the **LiveEvent** is running, you can programmatically create a second concurrent **LiveOutput** at the beginning of a program (or 5 minutes early to provide some handles to trim later). This second **LiveOutput** can be deleted 5 minutes after the program ends. With this second **Asset**, you can create a new **StreamingLocator** to publish this program as an on-demand asset in your application's catalog. You can repeat this process multiple times for other program boundaries or highlights that you wish to share as on-demand videos, all while the "Live" feed from the first **LiveOutput** continues to broadcast the linear feed. 

> [!NOTE]
> **LiveOutput**s start on creation and stop when deleted. When you delete the **LiveOutput**, you are not deleting the underlying **Asset** and content in the asset. 
>
> If you have published the **LiveOutput** asset using a **StreamingLocator**, the **LiveEvent** (up to the DVR window length) will continue to be viewable until the **StreamingLocator**’s expiry or deletion, whichever comes first.

## Next steps

- [Live streaming overview](live-streaming-overview.md)
- [Live streaming tutorial](stream-live-tutorial-with-api.md)

