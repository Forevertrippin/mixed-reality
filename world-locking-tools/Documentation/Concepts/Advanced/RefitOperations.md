---
title: Refit operations in World Locking Tools
description: Infrequent (optional) operations to improve the locking of the virtual coordinate space to physical features.
author: AMollis
ms.author: amollis
ms.date: 10/06/2021
ms.service: mixed-reality
ms.localizationpriority: high
ms.topic: concept-article
keywords: Unity, HoloLens, HoloLens 2, Augmented Reality, Mixed Reality, ARCore, ARKit, development, MRTK
---

# Refit operations

Refit operations in World Locking Tools are when the system determines that a repositioning of some of the objects in the scene would better register those objects with their physical world anchors.

This section will attempt to give further insight into the situations leading up to refit operations, and the mechanics of the operations themselves.

It's important to stress here that refit operations occur infrequently. The default error tolerances triggering a refreeze operation are customizable by the application, but normally, mixed reality environments trigger refreeze operations only in extraordinary circumstances. Conditions that might contribute to the need for a refreeze include:

* Loss of tracking due to poor environment.
* Fast movement of the head influencing environment scanning.
* Dynamic environments.
* Loop closure (that is, accumulating tracking errors on a roundabout path leading back to a previously visited spot).

The root cause of these issues is poor tracking--in other words, tracking errors. With reasonable environments producing reasonably good tracking, and especially after an initial scan of the space, refit operations will become exceedingly rare.

## Fragment merge

Many conditions may lead to the existence of multiple fragments, the most common cause being temporary loss of tracking. Fragments are defined as collections of objects sharing a common coordinate space, but where the coordinate space of one fragment is indeterminately located relative to another fragment.

When enough new sensor data is received and processed that the contents of two previously unrelated fragments can be positioned correctly relative to each other in the same space, a fragment merge can be performed.

The new coordinate space into which the contents of the two (or more) fragments will be merged is arbitrary. We mention here that the final coordinate space will be that of one of the spaces, which is only relevant because it means that all of the fragments being merged except one, the destination fragment, will need their coordinates adjusted. The contents of the fragment chosen as the final target of the merge will be unaffected.

An *adjustment transform* is computed by the system for each of the source fragments being merged. The AdjustLocationDelegate for each attachment point in those fragments will be called with the adjustment transform. Again, attachment points in the destination fragment will not be affected, nor have their AdjustLocationDelegates called.

The two rooms connected by a dark hallway scenario in the description of [fragments](Fragments.md) is an example of one such situation. During the initial phase, both fragments (rooms) have been scanned, but there is no information available regarding the relative positions of the two fragments. So the coordinate system that the contents of each fragment is placed in is arbitrary, as long as it is constant across all objects in that room. For example, the contents of each room might be in a coordinate system with its origin at the southwest corner of that room. The coordinates of two objects in the same fragment indicate the positions of the two objects relative to each other, but the coordinates of two objects in two different fragments tell nothing about their relative positions.

When more information is acquired, then there is an opportunity to adjust the coordinates of the contents of the second fragment so that the coordinates of its contents are meaningful relative to objects in the first fragment. For example, the hallway light might be turned on and the hallway traversed, bridging the gap between the two fragments. If the coordinates of all objects in both rooms are adjusted to be in the same consistent coordinate space, there is no longer any real distinction between the two fragments, and so their contents may be considered to all belong to a single common fragment.

This operation of collapsing multiple fragments into a single fragment is a *merge operation*.

It's important to note here that for normal Unity objects placed in the scene in world locked space, the merge operation will have no effect. Movement of objects from refit operations only happens through attachment points.

## Refreeze operations

Another situation that arises is when the positions of the anchors are refined over time. It becomes apparent that a rotation/offset transform is no longer adequate to compensate for the difference between the initial rough anchor positions and the more recent improved positions in the physical world. Keep in mind that the anchors themselves are constantly moving relative to each other in spongy space. But the attachment points derived from these anchors are fixed in world locked space.

When the system recognizes that the attachment points that it manages could be better registered with the physical world because of updates to anchor positions, then it has another opportunity for a correction event. This adjustment of attachment point positions to reflect new sensor data is known as a *refreeze operation*. Whereas in a merge operation the contents of a fragment are all adjusted by a single transform to merge coordinate spaces of two fragments into a single unified space, a refreeze adjusts each attachment point individually based on the updated positions of the anchors influencing it.

As in the merge operation, each attachment point is informed of its computed adjustment transform via its AdjustLocationDelegate.  

If conditions are right, the refreeze might also perform a merge operation. That merge will be considered an implicit part of the refreeze: There will be no separate events generated for the merge, and the adjustment transform delivered as part of the refreeze will include both the individual adjustment due to anchor movement and the fragment adjust due to the merge.

## Reacting to refit events

In either merge or refreeze, the reaction to refit events is up to the application. More precisely, it's up to each of the attachment point handlers, since different object types might react differently. Typically, the objects influenced by the attachment point will be moved by the adjustment transform via their GameObject.transform. Motion might instead be implemented by some other mechanism, such as manually moving vertices. It might even be advantageous for some applications to discard objects affected by a merge and begin a new cycle of creation.

The point is that World Locking Tools has no dependence on how or whether the application reacts to refit operations. It's entirely up to the application developer's needs.

The preferred mechanism for notification and reaction to refit operations is the [Attachment point](AttachmentPoints.md). More details and options are discussed in that article.

## See also

* [Attachment points](AttachmentPoints.md)
* [Fragments](Fragments.md)
