---
title: Material instance
description: Documentation on Material Instance and its uses in MRTK
author: lolambean
ms.author: lolab
ms.date: 01/12/2021
keywords: Unity,HoloLens, HoloLens 2, Mixed Reality, development, MRTK, MaterialInstance,
---

# Material instance &#8212; MRTK2

The [`MaterialInstance`](xref:Microsoft.MixedReality.Toolkit.Rendering.MaterialInstance?view=mixed-reality-toolkit-unity-2020-dotnet-2.8.0&preserve-view=true) behavior aides in tracking instance material lifetime and automatically destroys instanced materials for the user. This utility component can be used as a replacement to [Renderer.material](https://docs.unity3d.com/ScriptReference/Renderer-material.html) or
[Renderer.materials](https://docs.unity3d.com/ScriptReference/Renderer-materials.html).

> [!NOTE]
> [MaterialPropertyBlocks](https://docs.unity3d.com/ScriptReference/MaterialPropertyBlock.html) are preferred over material instancing but are not always available  in all scenarios.

Why can using [Renderer.material](https://docs.unity3d.com/ScriptReference/Renderer-material.html) be an issue? If you add the below code to a Unity scene and hit play memory usage will continue to climb and climb:

```c#
public class Leak : MonoBehaviour
{
    private void Update()
    {
        var cube = GameObject.CreatePrimitive(PrimitiveType.Cube);
        // Memory leak, the material allocated is not tracked & destroyed.
        cube.GetComponent<Renderer>().material.color = Color.red;
        ...
        Destroy(cube);
    }
}
```

> [!NOTE]
> The above Leak behavior **will crash Unity** if ran for too long!

As an alternative try using the [`MaterialInstance`](xref:Microsoft.MixedReality.Toolkit.Rendering.MaterialInstance?view=mixed-reality-toolkit-unity-2020-dotnet-2.8.0&preserve-view=true) behavior:

```c#
public class NoLeak : MonoBehaviour
{
    private void Update()
    {
        var cube = GameObject.CreatePrimitive(PrimitiveType.Cube);
        // No memory leak, the material allocated is tracked & destroyed by MaterialInstance.
        cube.EnsureComponent<MaterialInstance>().Material.color = Color.red;
        ...
        Destroy(cube);
    }
}
```

## Usage

When invoking Unity's [Renderer.material](https://docs.unity3d.com/ScriptReference/Renderer-material.html)(s), Unity automatically instantiates new materials. It is the caller's responsibility to destroy the materials when a material is no longer needed or the game object is destroyed. The [`MaterialInstance`](xref:Microsoft.MixedReality.Toolkit.Rendering.MaterialInstance?view=mixed-reality-toolkit-unity-2020-dotnet-2.8.0&preserve-view=true) behavior helps avoid material leaks and keeps material allocation paths consistent during edit and run time.

When a [MaterialPropertyBlock](https://docs.unity3d.com/ScriptReference/MaterialPropertyBlock.html) can not be used and a material must be instanced, [`MaterialInstance`](xref:Microsoft.MixedReality.Toolkit.Rendering.MaterialInstance?view=mixed-reality-toolkit-unity-2020-dotnet-2.8.0&preserve-view=true) can be used as follows:

```c#
public class MyBehaviour : MonoBehaviour
{
    // Assigned via the inspector.
    public Renderer targetRenderer;

    private void OnEnable()
    {
        Material material = targetRenderer.EnsureComponent<MaterialInstance>().Material;
        material.color = Color.red;
        ...
    }
}
```

If multiple objects need ownership of the material instance it's best to take explicit ownership for reference tracking. (An optional interface called [`IMaterialInstanceOwner`](xref:Microsoft.MixedReality.Toolkit.Rendering.IMaterialInstanceOwner?view=mixed-reality-toolkit-unity-2020-dotnet-2.8.0&preserve-view=true) exists to aide with ownership.) Below is example usage:

```c#
public class MyBehaviour : MonoBehaviour,  IMaterialInstanceOwner
{
    // Assigned via the inspector.
    public Renderer targetRenderer;

    private void OnEnable()
    {
        Material material = targetRenderer.EnsureComponent<MaterialInstance>().AcquireMaterial(this);
        material.color = Color.red;
        ...
    }

    private void OnDisable()
    {
        targetRenderer.GetComponent<MaterialInstance>()?.ReleaseMaterial(this)
    }

    public void OnMaterialChanged(MaterialInstance materialInstance)
    {
        // Optional method for when materials change outside of the MaterialInstance.
        ...
    }
}
```

For more information please see the example usage demonstrated within the [`ClippingPrimitive`](xref:Microsoft.MixedReality.Toolkit.Utilities.ClippingPrimitive?view=mixed-reality-toolkit-unity-2020-dotnet-2.8.0&preserve-view=true) behavior.

## See also

* [MRTK Standard Shader](mrtk-standard-shader.md)
