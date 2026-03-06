# Tomo Core
## What is it?
A simple plugin that bring various basic functionalities that solves issues faced in majority of the projects (at least for me)
1) solves race conditions for projects using singleton architecture without the need of relying execution order
2) Bring easy number formating (e.g 12,345.67) to game while respecting client's locale and easy rounding to the desired decimal places
3) Minor but useful extensions to commonly used struct and classes like transform and Vector

## Race Condition
At the base of Tomo Core is 2 important scripts
1) Readyable Monobehaviour
2) ControllerBase<T>

### ReadyableMonobehaviour
As the name suggest this brings the concept of "Readyable" to monobehaviour. 
To implement, simply replace monobehaviour with ReadyableMonobehaviour class

It introduces 
1) Bool; IsReady
2) Action; OnReady

This allows a daisy chaining of logic by getting it's dependencies listen in on the ready event while having a ready state itself for potential dependencies downstream. (This example assumes reference to the script is handled; adjust accordingly)
Implementation example:
Upstream script/logic:
```
public class ExampleScript : ReadyableMonobehaviour
{
    private void Awake()
    {
        /*
        Handle initialization logic here
        */
        SetToReady();
    }
}
```
Downstream/Dependency script:
```
public class ExampleDependencyScript : ReadyableMonobehaviour
{
    private void Awake()
    {
        ExampleScript.OnReady += HandleDependenciesReady;
        HandleDependenciesReady;
    }

    void HandleDependenciesReady()
    {
        if(!ExampleScript.OnReady || IsReady)
            return;

        /*
            Handle init here
        */
    }
}
```
In the event multiple dependencies exist it can be easily scaled up like this
```
public class ExampleDependencyScript : ReadyableMonobehaviour
{
    private void Awake()
    {
        ExampleScript.OnReady += HandleDependenciesReady;
        ExampleScript2.OnReady += HandleDependenciesReady;
        ...
        ...

        HandleDependenciesReady;
    }

    void HandleDependenciesReady()
    {
        if(!ExampleScript.IsReady || ExampleScript2.IsReady || ... || ... || IsReady)
            return;

        /*
            Handle init here
        */
    }
}
```

As you can see, scripts upstream do not have any concept of scripts downstream, and number of dependencies can scale up without issues.

### ControllerBase
Similar to readyable monobehaviours, this also allows for the easy setup of Controllers/Singletons via using generics. It creates a static instance like a normal controller
Example
```
public class ExampleScript : ControllerBase<ExampleScript>
{
    protected override void ControllerAwake()
    {
        //Handle Init logic here
        SetControllerToReady();
    }
}
```
Note: ControllerAwake() replaces the use of Awake(), awake should never be used. 
Similar to Readyable monobehaviours, script downstream can handle dependencies the same way.

Note: This does not make the script a DoNotDestroy and will not persist across scenes unless specified. If script is meant to be used as a local/scene controller, remember to handle destruction properly

## Number formatting
Namespace: Tomo.Core.Formatting;

Handles formatting while taking into account locale
Refer to the microsoft docs for locale explanation. 
[Number grouping and separation](https://learn.microsoft.com/en-us/globalization/locale/number-formatting#number-grouping-and-separation)

Handles decimal places
```
Input:
NumberFormatUtil.GetFormattedNumber(1234.456f);
NumberFormatUtil.GetFormattedNumber(1234.456f, 1);
NumberFormatUtil.GetFormattedNumber(1234.456f, 2);
NumberFormatUtil.GetFormattedNumber(1234.456f, 3);
NumberFormatUtil.GetFormattedNumber(1234.456f, 4);

Output:
1234.456 //no formatting, simply cast into string
1,234.5 //1 S.F, rounds up
1,234.46 //2 S.F, rounds up
1,234.456 //3 S.F
1,234.4560 //4 S.F, adds additional 0 to fit
```

## Extensions for common structs/classes
Namespace: Tomo.Core.Extensions;

### GameObject
```
gameObject.SetLayerRecursively(int layerIndex);
```
As name suggests, sets the layer of the gameobject specified and all the child via layer index

### Transform
```
transform.GetAllChildren(bool shouldIncludeSelf);
```
Gets all children of the specified transform recursively and return the children as List<Transform>. Will include self is bool is true, by default it is false.

### Vector2/Vector3
```
new Vector3(1, 2, 3).XZZeroY(); // Returns a Vector3 with y; zeroed (1, 0, 3)
new Vector3(1, 2, 3).ToXZ(); // Converts Vector3 into a Vector2 while maintaining XZ; (1, 3)
new Vector2(1, 2).ToVector3XZ(); // Converts a Vector2 into Vector3 using XZ as X and Y; (1, 0, 2)
```

### Array
Mostly for 2d array handling
```
int[,] m_ExampleIntArray;

m_ExampleIntArray.Get(new Vector2Int(1, 2)); // Gets value from m_ExampleIntArray[1][2];
m_ExampleIntArray.Set(new Vector2Int(1, 2), 1337); // Sets value in m_ExampleIntArray[1][2]; 
```

While example given is of a int 2d array, the type is generic and can support 2d array of any type


