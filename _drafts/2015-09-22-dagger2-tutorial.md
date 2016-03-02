---
title: Dagger2 Tutorial
date: 2015-09-03
published: false
---

1. **What is dependency injection?**

   The general concept behind dependency injection is called Inversion of Control. According to this concept a class should not configure its dependencies statically but should be configured from the outside.
   If the Java class creates an instance of another class via the new operator, it cannot be used (and tested) independently from this class and this is called a hard dependency.

2. **Using annotations to describe class dependencies**

   The standard Java annotations for describing the dependencies of a class are defined in the Java Specification Request 330.

3. **Where can objects be injected into a class?**

   This is the order in which dependency injection is performed on a class
     1. Constructor injection
     2. Field injection
     3. Method injection

#### Dagger 1
   * Developed by developers at Square.
   * The way Dagger works
       * Defining modules that contain provider methods for all the dependencies that you might want to inject.
       * Loading these modules into an object graph.
       * Finally injecting its contents into targets as needed.
   * Issues
       * Graph composition at runtime - hurts performance, especially in a per-request use case.
       * Reflection (i.e. Class.forName() on generated types) - makes generated code hard to follow and ProGuard a nightmare to configure.
       * Ugly generated code - especially, in comparison to similarly written manual instantiation from factories.


#### Dagger 2
  * Developed by core libraries team at Google (creators or Guava).
  * Dagger 2 is less dynamic than the others (no reflection usage at all).
  * Fully traceable source code which mimics the code that user may write by hand.
  * Advantage
     * No more reflection - everything is done as concrete calls (ProGuard works with no configuration at all).
     * No more runtime graph composition - improves performance, including the per-request cases (around 13% faster in Google's search products, according to Gregory Kick).
     * Traceable - better generated code and no reflection help make the code readable and easy to follow.
  * API of Dagger 2
  {% highlight java linenos %}
  public @interface Component {
    Class<?>[] modules() default {};
    Class<?>[] dependencies() default {};
  }
  public @interface Subcomponent {
    Class<?>[] modules() default {};
  }
  public @interface Module {
    Class<?>[] includes() default {};
  }
  public @interface Provides {
  }
  public @interface MapKey {
    boolean unwrapValue() default true;
  }
  public interface Lazy<T> {
    T get();
  }
{% endhighlight %}

Other elements defined by JSR-330

{% highlight java linenos %}
public @interface Inject { }
public @interface Scope { }
public @interface Qualifier { }
{% endhighlight %}

#### @Inject annotation
  * **Constructor injection**
    * @Inject annotation used in costructor also makes this class a part of dependencies graph

     {% highlight java linenos %}
     public class LoginActivityPresenter {
	   private LoginActivity loginActivity;
	   private UserDataStore userDataStore;
	   private UserManager userManager;@Inject public LoginActivityPresenter(LoginActivity loginActivity, UserDataStore userDataStore, UserManager userManager) {
		this.loginActivity = loginActivity;
		this.userDataStore = userDataStore;
		this.userManager = userManager;
  	}
  }
{% endhighlight %}

   * It means that it can also be injected when it’s needed i.e.
     {% highlight java linenos %}
     public class LoginActivity extends BaseActivity {
       @Inject LoginActivityPresenter presenter;
     }
    {% endhighlight %}

   * Limitation in this case is that we cannot annotate more than one constructor in class with @Inject
 * Fields injection
   * Annotate specific fields with @Inject

   * Injection process has to be called “by hand”, before this call our  dependencies are null values.

   * Limitation in fields injection is that they cannot be private.

 * Methods injection
   * Annotate specific method with @Inject

   * All method parameters are provided from dependencies graph
   * It will work in situations when we want to pass class instance itself (this reference) to injected dependencies
   * Method injection is called immediately after constructor call, so it means that we are passing fully constructed this.

 * Lazy injections
   * If there are many different injections in one target and some of them will only be used if some event occurs that doesn't always occur we can do a lazy injection.


 * Provider injections
  * Create multiple instances of an object, instead of just injecting one. For that situation you can inject a Provider<T>


#### @Module annotation : With Module Dagger will know in which place required objects are constructed.

Example
Context
System services (e.g. LocationManager)
REST service (e.g. Retrofit)
Database manager (e.g. Realm)
Message passing (e.g. EventBus)
Analytics tracker (e.g. Google Analytics)
@Provides annotation
This annotation is used in @Module
@Provides will mark those methods in Module which return dependencies.

@Component annotation
In this place we define from which modules (or other Components) we’re taking dependencies
Also here is the place to define which graph dependencies should be visible publicly (can be injected)
And where our component can inject objects.
@Component in general is a something like bridge between @Module and @Inject.

Also @Component can depend on another component,

Component Dependencies vs Subcomponents Dependencies

Component Dependencies - Use this when:

you want to keep two components independent.
you want to explicitly show what dependencies from one component is used by the other
Subcomponents - Use this when:

you want to keep two component cohesive
you may not care to explicitly show what dependencies from one component is used by the other
I will try to show this by and example. Given that we have the below modules and classes. SomeClassB1 is dependent on SomeClassA1. Notice the provieSomeClassB1 method in ModuleBwhich shows this dependency.


@Module
public class ModuleA {
    @Provides
    public SomeClassA1 provideSomeClassA1() {
        return new SomeClassA1();
    }
}

@Module
public class ModuleB {
    @Provides
    public SomeClassB1 provideSomeClassB1(SomeClassA1 someClassA1) {
        return new SomeClassB1(someClassA1);
    }
}

public class SomeClassA1 {
    public SomeClassA1() {}
}

public class SomeClassB1 {
    private SomeClassA1 someClassA1;

    public SomeClassB1(SomeClassA1 someClassA1) {
        this.someClassA1 = someClassA1;
    }
}
Note the following points in the Component Dependency example below:

SomeClassB1 is dependent on SomeClassA1. ComponentA has to explicitly define the dependency.
ComponentA doesn't need to declare ModuleB. This keeps the two components independent.

public class ComponentDependency {
    @Component(modules = ModuleA.class)
    public interface ComponentA {
        SomeClassA1 someClassA1();
    }

    @Component(modules = ModuleB.class, dependencies = ComponentA.class)
    public interface ComponentB {
        SomeClassB1 someClassB1();
    }

    public static void main(String[] args) {
        ModuleA moduleA = new ModuleA();
        ComponentA componentA = DaggerComponentDependency_ComponentA.builder()
                .moduleA(moduleA)
                .build();

        ModuleB moduleB = new ModuleB();
        ComponentB componentB = DaggerComponentDependency_ComponentB.builder()
                .moduleB(moduleB)
                .componentA(componentA)
                .build();
    }
}
Note the following points in the SubComponent example:

SomeClassB1 is dependent on SomeClassA1. ComponentA need not explicitly define the dependency.
ComponentA has to declare ModuleB. This makes the two components coupled.

public class SubComponent {
    @Component(modules = {ModuleA.class, ModuleB.class})
    public interface ComponentA {
        ComponentB componentB(ModuleB moduleB);
    }

    @Subcomponent(modules = ModuleB.class)
    public interface ComponentB {
        SomeClassB1 someClassB1();
    }

    public static void main(String[] args) {
        ModuleA moduleA = new ModuleA();
        ModuleB moduleB = new ModuleB();
        ComponentA componentA = DaggerSubComponent_ComponentA.builder()
                .moduleA(moduleA)
                .moduleB(moduleB)
                .build();

        ComponentB componentB = componentA.componentB(moduleB);
    }
}

@Scope annotation
Scopes are single-instances but related to component lifecycle (not the whole application)

 Custom scopes: Dagger 2 scopes mechanism cares about keeping single instance of class as long as its scope exists
 Instances scoped in @ApplicationScope lives as long as Application object
@ActivityScope keeps references as long as Activity exists

@MapKey
This annotation is used to define collections of dependencies
@MapKey annotation supports only two types of keys for now - Strings and Enums.
Definition

Providing dependencies

Usage

@Qualifier
Imagine that you need to provide two RestAdapter objects - one for Github API, another for facebook API. Qualifier will help you to identify the proper one:
Naming dependencies


Injecting dependencies

Annoyance #1:
The inject() method now has a strong type association with the injection target. i.e can’t use injected object from base classes.
Solution #1: Defining an abstract method in your base class to perform the actual injection





Dagger2 Example
Creating Base component
Adding sub component to base component
Adding component as dependence to base component
