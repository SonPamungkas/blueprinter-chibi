### 1. Targeting Stacked Components in Blueprinter
When you have multiple components of the exact same type on a single GameObject, Unity stores them in an array. Blueprinter's `LocationRef` system has a specific field called **`componentIndex`** just for this scenario. 

If you have three `UnitPart` components on the same part of your helicopter, you can target each one individually by changing the index (it is 0-indexed):

```json
{
  "id": "TargetFirstUnitPart",
  "asset": { "locator": "MyCustomIbis", "type": "UnityEngine.GameObject" },
  "hierarchyPath": "Fuselage/SomePart",
  "componentType": "UnitPart",
  "componentIndex": 0,
  "memberPath": "someFieldToPatch"
},
{
  "id": "TargetSecondUnitPart",
  "asset": { "locator": "MyCustomIbis", "type": "UnityEngine.GameObject" },
  "hierarchyPath": "Fuselage/SomePart",
  "componentType": "UnitPart",
  "componentIndex": 1,
  "memberPath": "someFieldToPatch"
}
```
If you don't define `componentIndex`, Blueprinter will always default to `0` (the first component of that type it finds).

### 2. Injecting your Animation/Logic
If you built the unfolding logic using Unity's Animation system (e.g., an `Animator` component with a custom `RuntimeAnimatorController` or `AnimationClip`), you can use Blueprinter to overwrite the base game's animation controller with yours.

To do this, you would create an `AssetPatch` that takes your custom animation controller from your AssetBundle and assigns it to the helicopter's Animator:

```json
{
  "GameAsset": {
    "id": "CustomUnfoldAnim",
    "asset": { "locator": "MyUnfoldAnimatorController", "type": "UnityEngine.RuntimeAnimatorController" }
  },
  "PatchLocations": [
    {
      "id": "PatchIbisAnimator",
      "asset": { "locator": "UH-90 Ibis", "type": "UnityEngine.GameObject" },
      "hierarchyPath": "Model/AnimationRig", 
      "componentType": "UnityEngine.Animator",
      "memberPath": "runtimeAnimatorController"
    }
  ]
}
```
*(Note: You'll need to double-check the exact `hierarchyPath` where the Ibis's Animator is located).*

### 3. Modifying `UnitPart` values dynamically
If your unfolding logic involves changing the physical `UnitPart` properties (like hitboxes, drag, or health) based on whether it is folded or unfolded, you might need a custom script if it's a dynamic change during flight. 

However, if you are just replacing the base stats of the `UnitPart` to accommodate the new model shapes, you can patch those fields directly using `memberPath` (e.g., `"memberPath": "mass"` or `"memberPath": "dragMultiplier"`).

**Summary of the workflow:**
1. Put your custom components/animations on your prefab in your Unity project.
2. Build it into an AssetBundle.
3. Write your `PatchManifest.json` using `componentIndex` to precisely target the correct stacked `UnitPart`.
4. Run it through Blueprinter!