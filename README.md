# Dr Inject

#### Dr Inject is simple Dependency Injection tool for Unity game engine.

## How to setup?

##### 1. Create custom class and extend DiContainer
``` c#
public class MyDiContainer : DiContainer
{

    public override void Provide()
    {
        
    }
}
```

In this class you will provide every class which you want to be injected.

For example, let's create class for testing ->
``` c#
public class SimpleClass
{
    public void Print()
    {
        Debug.Log("Hello from simple class!");
    }
}
```

##### 2. To use class for Injection, you first must to tell DI container how to use it
``` c#
public class MyDiContainer : DiContainer
{

    public override void Provide()
    {
        AddFromInstance(new SimpleClass());
    }
}
```
`AddFromInstance` is just one of posible solution to provide wanted class into DI (more options will be described latter)

##### 3. To use that class directly from DI, set->
``` c#
public class GameManager : MonoBehaviour
{
    [Inject]
    public SimpleClass simpleClass; 

    void Start()
    {
        // Register class for binding!
        DiContainer.Instance.Bind(this);


        simpleClass.Print();
    }
}
```
`Inject` attribute tell DI to provide class and `DiContainer.Instance.Bind(this);` tells DI container to instantete all required things from DI.
After that you can access to the object of injected class, like `simpleClass.Print();`.

Last step is to add empty game object into the sceen and to attach your container implementation (in this case `MyDiContainer`) to it.

### You have several options to provide object

### 1. From Instance
Like you see abowe, if you add iy by instance, all other classes which access to the SimpleClass injeceted by instance, will actually access to the same object.

If you add one more inject to the GameManager required SimpleClass, they will receive the same object:
``` c#
public class GameManager : MonoBehaviour
{
    [Inject]
    public SimpleClass simpleClass; 
    
    [Inject]
    public SimpleClass simpleClass2; 

    void Start()
    {
        // Register class for binding!
        DiContainer.Instance.Bind(this);


        bool isSame = simpleClass.GetHashCode() == simpleClass2.GetHashCode(); // true
    }
}
```
#### But what if i want to have different objects?

To have different object, you must use classification by name:

``` c#
public class MyDiContainer : DiContainer
{

    public override void Provide()
    {
        AddFromInstance(new SimpleClass(), "simple1"); // this will go into simple1 class
        AddFromInstance(new SimpleClass(), "simple2"); // this will go into simple2 class
    }
}

```
And than to tell your class which to use:
``` c#
public class GameManager : MonoBehaviour
{
    [Inject("simple1")] // ADD HERE FOR simple1
    public SimpleClass simpleClass;

    [Inject("simple2")] // ADD HERE FOR simple2
    public SimpleClass simpleClass2;

    void Start()
    {
        // Register class for binding!
        DiContainer.Instance.Bind(this);


        bool isSame = simpleClass.GetHashCode() == simpleClass2.GetHashCode(); // false
    }
}
```
### 2. Add by type (factory)
Sometims you want everytime new instance when you required it by inject.
In that case you must to use injection by type and tell DI to create every time new instance:
``` c#
public class MyDiContainer : DiContainer
{

    public override void Provide()
    {
        AddByType<SimpleClass>(InjectionType.AsFactory);
    }
}
```
`AddByType` will tell DI to create that type of class by himself and `InjectionType.AsFactory` will tell to create every time new instace.
Example:
``` c#
public class GameManager : MonoBehaviour
{
    [Inject]
    public SimpleClass simpleClass;

    [Inject]
    public SimpleClass simpleClass2;

    void Start()
    {
        // Register class for binding!
        DiContainer.Instance.Bind(this);


        bool isSame = simpleClass.GetHashCode() == simpleClass2.GetHashCode(); // false
    }
   
}
```
### 3. Interface creation by type
If we have interaface 
``` c#
public interface ISimpleClass
{
    void Print();
}
```
And change simple class to implement that interface:
``` c#
public class SimpleClass : ISimpleClass
{
    public void Print()
    {
        Debug.Log("Hello from simple class!");
    }
}
```

You must to say DI which class to instantiate for that interface:
``` c#
public class MyDiContainer : DiContainer
{

    public override void Provide()
    {
        AddByType<ISimpleClass,SimpleClass>(InjectionType.AsFactory);
    }
}
```
Definition for AddByType is `AddByType<For all required interface, Provde class>`

And you can use it:
``` c#
public class GameManager : MonoBehaviour
{
    [Inject]
    public ISimpleClass simpleClass; // define interface usage

    void Start()
    {
        // Register class for binding!
        DiContainer.Instance.Bind(this);


        simpleClass.Print();
    }
   
}
```

#### You can also combine classification by name with interfaces to achive more complex solution

Let create one more extension for `ISimpleClass`:
``` c#
public class NotSoSimpleClass : ISimpleClass
{
    public void Print()
    {
        Debug.Log("Not so simple class!");
    }
}
```

Now we can separate it by classification:
``` c#
public class MyDiContainer : DiContainer
{

    public override void Provide()
    {
        AddByType<ISimpleClass,SimpleClass>(InjectionType.AsFactory, "simple");
        AddByType<ISimpleClass, NotSoSimpleClass>(InjectionType.AsFactory, "notSimple");
    }
}
```

And use it:
``` c#
public class GameManager : MonoBehaviour
{
    [Inject("simple")]
    public ISimpleClass simpleClass; // This will be SimpleClass

    [Inject("notSimple")]
    public ISimpleClass simpleClass2; // This will be NotSoSimpleClass

    void Start()
    {
        // Register class for binding!
        DiContainer.Instance.Bind(this);


        simpleClass.Print();
        simpleClass2.Print();
    }
   
}
```
### 4. Add by type as singleton
Let create new class which will be used as singleton:
``` c#
public class SingletonSample
{
    public void Hello()
    {
        Debug.Log("Hello from singleton!");
    }
}

```

To use it like singleton, you can just inject it:
``` c#
[Inject]
public SingletonSample singleton;
```

### 5. Provide mono classes
There is option to provide mono classes from instance.

Create one mono class:
``` c#
public class UiExample : MonoBehaviour
{
    public void Hello()
    {
        Debug.Log("Hello from mono UI");
    }
}
```

Than provide it to the DI container:
``` c#
public class MyDiContainer : DiContainer
{

    [SerializeField] UiExample uiExample; // Access to it from scene

    public override void Provide()
    {
        AddFromInstance(uiExample); // forward it to the DI
    }
}
```
And use it in your class:
``` c#
public class GameManager : MonoBehaviour
{
    [Inject]
    public UiExample uiExample;

    void Start()
    {
        // Register class for binding!
        DiContainer.Instance.Bind(this);

        uiExample.Hello();
    }
   
}
```
### 6. Contructor injection
Contructor injection is very important and in practice very usefull(or say "most wanted").
Use case for contructor injection is when you want to inject class which also requrire some injections.
For example let's create few classes and interfaces:
``` c#
public interface IEngine
{
    void startEngine();
}
```
and implementation of that interface:
``` c#
public class DieselEngine : IEngine
{
    public void startEngine()
    {
        Debug.Log("Start Diesel engine!");
    }
}
```

Also:
``` c#
public interface IWheel
{
    void Controll();
}
```
And implementation:
``` c#
public class TigarWheel : IWheel
{
    public void Controll()
    {
        Debug.Log("Tiger wheels are OK!");
    }
}
```
Let's say we have Car class and that class requreies injection of Iwheel specific implemention and IEngine.
For that case we can use constructor injection:
``` c#
public class Car
{
    private IEngine engine;
    private IWheel wheel;

    [Inject]
    public Car(IEngine engine, IWheel wheel) // Constructor injection
    {
        this.engine = engine;
        this.wheel = wheel;
    }


    public void StartCar()
    {
        engine.startEngine();
        wheel.Controll();
    }
}

```
Now you just need to sat DI which implementation to use and DI will make rest of the job for you:
``` c#
public class MyDiContainer : DiContainer
{
    public override void Provide()
    {
        AddByType<IEngine,DieselEngine>(InjectionType.AsFactory);
        AddByType<IWheel, TigarWheel>(InjectionType.AsFactory);
        AddByType<Car>(InjectionType.AsFactory);
    }
}
```
So now you can inject car:
``` c#
public class GameManager : MonoBehaviour
{
    [Inject]
    public Car car;

    void Start()
    {
        // Register class for binding!
        DiContainer.Instance.Bind(this);

        car.StartCar();
    }
   
}
```
