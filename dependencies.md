# 依賴（Dependencies） {#dependencies}

Class A depends on class B when B is used within A i.e. when B is required for A to function.

## Bad and good dependencies {#bad-and-good-dependencies}

There are two metrics of dependencies:

* Cohesion.
* Coupling.

Cohesion means dependency on a class with related functionality.

Coupling means dependency on a class with not really related functionality.

It's preferrable to have high cohesion and low coupling. That means you should get related functionality together into a group of classes usually called a module \(which is_not_Yii module and not an actual class, just a logical boundary\). Within that module it's a good idea not to over-abstract things and use interconnected classes directly. As for classes which aren't part of the module's purpose but are used by the module, such as general purpose utilities, these should not be used directly but through interface. Via this interface module states what is needed for it to function and from this point it doesn't care how these dependencies are satisfied.

## Achieving low coupling {#achieving-low-coupling}

You can't eliminate dependencies altogether but you can make them more flexible.

## Inversion of control {#inversion-of-control}

## Dependency injection {#dependency-injection}

## Dependency container {#dependency-container}

Injecting basic dependencies is simple and easy. You're choosing a place where you don't care about dependencies, which is usually controller which you aren't going to unit-test ever, create instances of dependencies needed and pass these to dependent classes.

It works well when there aren't many dependencies overall and when there are no nested dependencies. When there are many and each dependency has dependencies itself, instantiating the whole hierarchy becomes tedious process which requires lots of code and may lead to hard to debug mistakes.

Additionally, lots of dependencies, such as certain third party API wrapper, are the same for any class using it. So it makes sense to:

* Define how to instantiate such API wrapper once.
* Instantiate it when required and only once per request.

That's what dependency containers are for.

See[official guide](http://www.yiiframework.com/doc-2.0/guide-concept-di-container.html)for more information about Yii's dependency container.

