---
title: Material Animation
description: Learn about the Graphics Tools material animation.
author: lolambean
ms.author: lolab
ms.date: 05/05/2022
ms.localizationpriority: high
keywords: Unity, HoloLens, HoloLens 2, Mixed Reality, development, MRTK, Graphics Tools, MRGT, MR Graphics Tools, Standard Shader, Animation
---

# Animation &#8212; MRTK3

Most properties on the Graphics Tools Standard shader can be animated using Unity's built-in [animation system](https://docs.unity3d.com/Manual/AnimationOverview.html). Materials that are used on [Unity UI](https://docs.unity3d.com/ScriptReference/CanvasRenderer.html) components don't expose their material properties to Unity's animation system by default (nor do they support [material property blocks](https://docs.unity3d.com/ScriptReference/MaterialPropertyBlock.html)). Graphics Tools contains a system to support the animation of Unity UI material properties.

The _CanvasMaterialAnimatorGraphicsToolsStandardCanvas.cs_ script exposes all material properties available in the *Graphics Tools/Standard Canvas* shader. Adding this component to a UnityUI game object with a [CanvasRenderer](https://docs.unity3d.com/ScriptReference/CanvasRenderer.html) will expose material properties to Unity's animation system and will automatically update the correct material when animated.

> [!NOTE]
> _CanvasMaterialAnimatorGraphicsToolsStandardCanvas.cs_ only works with the *Graphics Tools/Standard Canvas* shader. For other shaders, use their corresponding animation script. For example, _CanvasMaterialAnimatorCanvasBackplate.cs_ for the `Graphics Tools/Canvas/Backplate` shader.

## Programmatic usage

Normally a canvas material animator is driven by Unity's animation system, however, it's possible to use this class programmatically. After changing any of the class's members, make sure to call the `ApplyToMaterial` method. An example of pulsing the vertex extrusion amount is below:

```C#
using UnityEngine;

public class ScriptedMaterialAnimation : MonoBehaviour
{
    public CanvasMaterialAnimatorGraphicsToolsStandardCanvas Animator;

    private void Update()
    {
        Animator._VertexExtrusionValue = Mathf.Lerp(0, 0.002f, (Mathf.Sin(Mathf.Repeat(Time.time, Mathf.PI * 2.0f)) + 1.0f) * 0.5f);
        Animator.ApplyToMaterial();
    }
}
```

## Advanced usage

If you inspect the content of _CanvasMaterialAnimatorGraphicsToolsStandardCanvas.cs_, there's boilerplate code that could fall out of sync with the *Graphics Tools/Standard Canvas* shader. Fortunately, this code is auto generated by right clicking on a shader in the [project window](https://docs.unity3d.com/Manual/ProjectView.html) and selecting **Graphics Tools > Generate Canvas Material Animator**.

You can generate a canvas material animator for any shader your project needs to animate. Material properties will be updated at edit and run time.

> [!NOTE]
> By default, canvas material animators operate on the renderer's shared material. If you would like the animation to only affect a single material, you can select the **Instance Materials** property on the canvas material animator's inspector. This will allocate a new material for every instance.

It's also worth noting that when animating shared materials at run time in the editor material updates may get serialized to disk. To avoid this, Graphics Tools uses the _MaterialRestorer.cs_ pattern.

## See also

* [Standard Shader](standard-shader.md)
