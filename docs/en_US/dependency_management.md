- [Simple Instance Manager](#simple-instance-manager)
- [Options](#options)
- [Bindings](#bindings)
  - [How to use](#how-to-use)
- [SmartManagement](#smartmanagement)

## Simple Instance Manager

Get has a simple and powerful dependency manager that allows you to retrieve the same class as your Bloc or Controller with just 1 lines of code, no Provider context, no inheritedWidget:

```dart
Controller controller = Get.put(Controller()); // Rather Controller controller = Controller();
```

- Note: If you are using Get's State Manager, pay more attention to the bindings api, which will make easier to connect your view to your controller.

Instead of instantiating your class within the class you are using, you are instantiating it within the Get instance, which will make it available throughout your App.
So you can use your controller (or Bloc class) normally

**Tip:** Get dependency management is decloupled from other parts of the package, so if for example your app is already using a state manager (any one, it doesn't matter), you don't need to rewrite it all, you can use this dependency injection with no problems at all

```dart
controller.fetchApi();
```

Imagine that you have navigated through numerous routes, and you need a data that was left behind in your controller, you would need a state manager combined with the Provider or Get_it, correct? Not with Get. You just need to ask Get to "find" for your controller, you don't need any additional dependencies:

```dart
Controller controller = Get.find();
//Yes, it looks like Magic, Get will find your controller, and will deliver it to you. You can have 1 million controllers instantiated, Get will always give you the right controller.
```

And then you will be able to recover your controller data that was obtained back there:

```dart
Text(controller.textFromApi);
```

Looking for lazy loading? You can declare all your controllers, and it will be called only when someone needs it. You can do this with:

```dart
Get.lazyPut<Service>(()=> ApiMock());
/// ApiMock will only be called when someone uses Get.find<Service> for the first time
```

If you want to register an asynchronous instance, you can use Get.putAsync.

```dart
Get.putAsync<SharedPreferences>(() async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setInt('counter', 12345);
    return prefs;
});
```

usage:

```dart
int count = Get.find<SharedPreferences>().getInt('counter');
print(count); // out: 12345
```

To remove a instance of Get:

```dart
Get.delete<Controller>();
```

## Instancing methods

Although Getx already delivers very good settings for use, it is possible to refine them even more so that it become more useful to the programmer. The methods and it's configurable parameters are:

- Get.put():

```dart
Get.put<S>(
  // mandatory: the class that you want to get to save, like a controller or anything
  // note: that "S" means that it can be anything
  S dependency

  // optional: this is for when you want multiple classess that are of the same type
  // since you normally get a class by using Get.find<Controller>(),
  // you need to use tag to tell which instance you need
  // must be unique string
  String tag,

  // optional: by default, get will dispose instances after they are not used anymore (example,
  // the controller of a view that is closed), but you might need that the instance
  // to be kept there throughout the entire app, like an instance of sharedPreferences or something
  // so you use this
  // defaults to false
  bool permanent = false,

  // optional: allows you after using an abstract class in a test, replace it with another one and follow the test.
  // defaults to false
  bool overrideAbstract = false,

  // optional: allows you to create the dependency using function instead of the dependency itself.
  InstanceBuilderCallback<S> builder,
)
```

- Get.lazyPut:

```dart
Get.lazyPut<S>(
  // mandatory: a method that will be executed when your class is called for the first time
  // Example: Get.lazyPut<Controller>( () => Controller() )
  InstanceBuilderCallback builder,
  
  // optional: same as Get.put(), it is used for when you want multiple different instance of a same class
  // must be unique
  String tag,

  // optional: It is similar to "permanent", the difference is that the instance is discarded when
  // is not being used, but when it's use is needed again, Get will recreate the instance
  // just the same as "SmartManagement.keepFactory" in the bindings api
  // defaults to false
  bool fenix = false
  
)
```

- Get.putAsync:

```dart
Get.putAsync<S>(

  // mandatory: an async method that will be executed to instantiate your class
  // Example: Get.putAsync<YourAsyncClass>( () async => await YourAsyncClass() )
  AsyncInstanceBuilderCallback<S> builder,

  // optional: same as Get.put(), it is used for when you want multiple different instance of a same class
  // must be unique
  String tag,

  // optional: same as in Get.put(), used when you need to maintain that instance alive in the entire app
  // defaults to false
  bool permanent = false
```

- Get.create:

```dart
Get.create<S>(
  // required: a function that returns a class that will be "fabricated" every
  // time `Get.find()` is called
  // Example: Get.create<YourClass>(() => YourClass())
  FcBuilderFunc<S> builder,

  // optional: just like Get.put(), but it is used when you need multiple instances
  // of a of a same class
  // Useful in case you have a list that each item need it's own controller
  // needs to be a unique string. Just change from tag to name
  String name,

  // optional: just like int`Get.put()`, it is for when you need to keep the
  // instance alive thoughout the entire app. The difference is in Get.create
  // permanent is true by default
  bool permanent = true
```

### Diferences between methods:

First, let's of the `fenix` of Get.lazyPut and the `permanent` of the other methods.

The fundamental difference between `permanent` and `fenix` is how you want to store your instances.
Reinforcing: by default, GetX deletes instances when they are not is use.
It means that: If screen 1 has controller 1 and screen 2 has controller 2 and you remove the first route from stack, (like if you use `Get.off()` or `Get.offName()`) the controller 1 lost it's use so it will be erased.
But if you want to opt to `permanent:true`, then the controller will not be lost in this transition - which is very usefult for services that you want to keep alive thoughout the entire application.
`fenix` in the other hand is for services that you don't worry in losing between screen changes, but when you need that service, you expect that it is alive. So basically, it will dispose the unused controller/service/class, but when you need that, it will "recreate from the ashes" a new instance.

Proceeding with the differences between methods: 

- Get.put and Get.putAsync follow the same creation order, with the difference that asyn opt for applying a asynchronous method: those two methods create and initialize the instance. That one is inserted directly in the memory, using the internal method `insert` with the parameters `permanent: false` and `isSingleton: true` (this isSingleton parameter only porpuse is to tell if it is to use the dependency on `dependency` or if it is to use the dependency on `FcBuilderFunc`). After that, `Get.find()` is called that immediately initialize the instances that are on memory.

- Get.create: As the name implies, it will "create" your dependency! Similar to `Get.put()`, it also call the internal method `insert` to instancing. But `permanent` became true and `isSingleton` became false (since we are "creating" our dependency, there is no way for it to be a singleton instace, that's why is false). And because it has `permanent: true`, we have by default the benefit of not losing it between screens! Also, `Get.find()` is not called immediately, it wait to be used in the screen to be called. It is created this way to make use of the parameter `permanent`, since then, worth noticing, `Get.create()` was made with the goal of create not shared instances, but don't get disposed, like for example a button in a listView, that you want a unique instance for that list - because of that, Get.create must be used together with GetWidget.

- Get.lazyPut: As the name implies, it is a lazy proccess. The instance is create, but it is not called to be used immediately, it remains waiting to be called. Contrary to the other methods, `insert` is not called here. Instead, the instance is inserted in another part of the memory, a part responsable to tell if the instance can be recreated or not, let's call it "factory". If we want to create something to be used later, it will not be mix with things been used right now. And here is where `fenix` magic enters: if you opt to leaving `fenix: false`, and your `smartManagement` are not `keepFactory`, then when using `Get.find` the instance will change the place in the memory from the "factory" to common instance memory area. Right after that, by default it is removed from the "factory". Now, if you opt for `fenix: true`, the instance continues to exist in this dedicated part, even going to the common area, to be called again in the future.

## Bindings

One of the great differentials of this package, perhaps, is the possibility of full integration of the routes, state manager and dependency manager.
When a route is removed from the Stack, all controllers, variables, and instances of objects related to it are removed from memory. If you are using streams or timers, they will be closed automatically, and you don't have to worry about any of that.
In version 2.10 Get completely implemented the Bindings API.
Now you no longer need to use the init method. You don't even have to type your controllers if you don't want to. You can start your controllers and services in the appropriate place for that.
The Binding class is a class that will decouple dependency injection, while "binding" routes to the state manager and dependency manager.
This allows Get to know which screen is being displayed when a particular controller is used and to know where and how to dispose of it.
In addition, the Binding class will allow you to have SmartManager configuration control. You can configure the dependencies to be arranged when removing a route from the stack, or when the widget that used it is laid out, or neither. You will have intelligent dependency management working for you, but even so, you can configure it as you wish.

### How to use

- Create a class and implements Binding

```dart
class HomeBinding implements Bindings {

}
```

Your IDE will automatically ask you to override the "dependencies" method, and you just need to click on the lamp, override the method, and insert all the classes you are going to use on that route:

```dart
class HomeBinding implements Bindings{
  @override
  void dependencies() {
    Get.lazyPut<ControllerX>(() => ControllerX());
    Get.lazyPut<Service>(()=> Api());
  }
}
```

Now you just need to inform your route, that you will use that binding to make the connection between route manager, dependencies and states.

- Using named routes:

```dart
namedRoutes: {
  '/': GetRoute(Home(), binding: HomeBinding())
}
```

- Using normal routes:

```dart
Get.to(Home(), binding: HomeBinding());
```

There, you don't have to worry about memory management of your application anymore, Get will do it for you.

The Binding class is called when a route is called, you can create an "initialBinding in your GetMaterialApp to insert all the dependencies that will be created.

```dart
GetMaterialApp(
  initialBinding: SampleBind(),
  home: Home(),
);
```

## SmartManagement

Always prefer to use standard SmartManagement (full), you do not need to configure anything for that, Get already gives it to you by default. It will surely eliminate all your disused controllers from memory, as its refined control removes the dependency, even if a failure occurs and a widget that uses it is not properly disposed.
The "full" mode is also safe enough to be used with StatelessWidget, as it has numerous security callbacks that will prevent a controller from remaining in memory if it is not being used by any widget, and disposition is not important here. However, if you are bothered by the default behavior, or just don't want it to happen, Get offers other, more lenient options for intelligent memory management, such as SmartManagement.onlyBuilders, which will depend on the effective removal of widgets using the controller. tree to remove it, and you can prevent a controller from being deployed using "autoRemove: false" in your GetBuilder/GetX.
With this option, only controllers started in "init:" or loaded into a Binding with "Get.lazyPut" will be disposed, if you use Get.put or any other approach, SmartManagement will not have permissions to exclude this dependency.
With the default behavior, even widgets instantiated with "Get.put" will be removed, unlike SmartManagement.onlyBuilders.
SmartManagement.keepFactory is like SmartManagement.full, with one difference. SmartManagement.full purges the factories from the premises, so that Get.lazyPut() will only be able to be called once and your factory and references will be self-destructing. SmartManagement.keepFactory will remove its dependencies when necessary, however, it will keep the "shape" of these, to make an equal one if you need an instance of that again.
Instead of using SmartManagement.keepFactory you can use Bindings.
Bindings creates transitory factories, which are created the moment you click to go to another screen, and will be destroyed as soon as the screen-changing animation happens. It is so little time that the analyzer will not even be able to register it. When you navigate to this screen again, a new temporary factory will be called, so this is preferable to using SmartManagement.keepFactory, but if you don't want to create Bindings, or want to keep all your dependencies on the same Binding, it will certainly help you . Factories take up little memory, they don't hold instances, but a function with the "shape" of that class you want. This is very little, but since the purpose of this lib is to get the maximum performance possible using the minimum resources, Get removes even the factories by default. Use whichever is most convenient for you.

- NOTE: DO NOT USE SmartManagement.keepFactory if you are using multiple Bindings. It was designed to be used without Bindings, or with a single Binding linked in the GetMaterialApp's initialBinding.

- NOTE2: Using Bindings is completely optional, you can use Get.put() and Get.find() on classes that use a given controller without any problem.
However, if you work with Services or any other abstraction, I recommend using Bindings for a larger organization.
