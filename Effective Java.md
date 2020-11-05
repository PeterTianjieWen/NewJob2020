## CH1

* Support types in Java:

  * *Reference types*: interfaces, classes, arrays
    * Class signature doesn't include the method's return type
  * primitives

* **Static factory methods** > Constructors

  * methods have names
  * not required to create a new object each time they're invoked
  * can return an object of any subtype of their return type
    * By convention, static factory methods for an interface named *Type* were put in a *noninstantiable companion class* name *Types* (Collection vs Collections)
  * Naming convention for static factory p9

* **Consider a builder when faced with many constructor parameters**

  * Telescoping constructor pattern: hard to write client code, hard to read

  * JavaBeans pattern: allow inconsistency (not thread-safe), mandate mutability

  * **Builder pattern**: use builder object to set all parameters, then use a `build` method to generate the object

    

22. **Use interfaces only to define types**

    * Pitfall of *constant interface* : if a non-final class implements a constant interface, all of its subclasses will have their namespaces polluted by the constatnts in the interfaces

    * **Export constants:**

      * If strongly tied to an existing classs or interface, add to the class or interface
      * If best viewed as enumerated type, export with an *enum type*
      * Export with a non-instantiable *utility class*

    * Underscore character(_) have no effect on the values of numeric literals since Java 7, use when containing five or more consecutive digits(`1_234_567`).

    * Utility class requires clients to qualify constant names with a class name, i.e. `PhysicalConstants.AVOGADROS_NUMBER`. Avoid this by making use of the *static import facility* **only if **the constants are heavily used.

      ```java
      import static science.PhysicalConstants.*;
      
      public class Test {
        double atoms(double mols){
          return AVOGADROS_NUMBER * mols;
        }
      }
      ```

      

23. **Prefer class hierarchies to tagged classes**

    * *tagged classes* shortcoming: **verbose, error-prone and inefficient**

      * Cluttered with boilderplate
      * Low readability because multiple implementations are jumbled together
      * Increased memory footprint

    * Transform a tagged class into a class hierarchy

      * Define an abstract class (**root** of hierarchy) containing an <u>abstract method</u> for each method in the tagged class whose behavior depends on the tag value (`area` in this example). Put **methods/data fields** independent of the tag value in this class

      * Define a concrete class for each flavor of the original tagged class (`Circle` and `Rectangle`)

        ```java
        abstract class Figure {
        	abstract double area();
        }
        
        class Circle extends Figure {
        	//all fields are final
          final double radius;
          Circle(double radius) {this.radius = radius; }  
          @Override double area() {return Math.PI * (radius * radius); }
        }
        ```

        

    * **Advantage of class hierarchies**: they can be made to reflect natural hierarchical relationships among types, allowing for increased flexibility and better compile-time type checking.

24. **Favor static member classes over non-static**

    * Four kinds of **nested class**: *static member classes*, *nonstatic member classes*, *anonymous classes* and *local classes*

    * **Static member class**: used as a public helper class, represent components of the object represented by their enclosing class

    * **Nonstatic member class**: each instance is implicitly associated with an enclosing instance of its containing class

      * Invoke methods of enclosing instance/obtain a reference to the enclosing instance using ***qualified this*** construct.

    * Association between nonstatic member class & its enclosing instance is established when the member class is created and cannot be modified after. (Manually establish association with the expression `enclosingInstance.new MemberClass(args)`),

    * Used to define ***Adapter*** that allows an instance of the outer classs to be viewed as an instance of some unrelated class.

      ```java
      public class MySet<E> extends AbstractSet<E> {
        //omitted...
        
        @Override public Iterator<E> iterator(){
          return new MyIterator();
        }
        
        private class MyIterator implements Iterator<E> {
          ...
        }
      }
      ```

    * If declare a member class that does not require access to an enclosing instance, *always* put the `static` modifier in its declaration. 

      * e.g. `Entry` class for many `Map` implementations. Method on an entry do not need to access the map.
      * `Inner classes whose declarations do not occur in a static context may freely refer to the instance variables of their enclosing class.`

    * Anomymous classes: 

      * Preferred `lambda` over creating *function objects* and *process objects* on the fly
      * Implement static factory methods

## Ch 5

26. **Don't use raw types**

* Definition: 

  * A class or interface whose declaration has one or more *type parameters* is a *generic* class or interface
    * List interface `List<E>` (Read list of E)
  * Generic classes and interfaces are collectively known as *generic types*
  * Use parameterized collection type to make sure it's typesafe

  ```java
  private final Collection<Stamp> stamps = ...;	
  ```

  The compiler knows that `stamps` should contain only `Stamp` instances and *guarantees* it to be true, <u>**assurming your entire codebase compiles without emitting (or suppressing) any warnings**</u>.

* Java permit raw types because it had to be legal to pass instances of parameterized types to methods that were designed for use with raw types, and vice versa. (*migration compatibility* -> support raw types and to implement generics using *erasure*)

* It is fine to use types that were parameterized to allow insertion of arbitrary objects, such as `List<Object>` .

  * You can pass a `List<String>` to a parameter of type `List`, but cannot pass `List<String>` to `List<Object>` because of the subtyping rules for generics. 

* Use *unbounded wilcard* if you want to use a generic type but you don't know or care what the actual type parameter is. 

  * `Set<?>` : Set of some type

  ```java
  static int numElementsInCommon(Set<?> set1, Set<?> set2) { ... }
  ```

* Exception to use raw types: 

  * use raw types in class literals, i.e. `List.class`, `String[].class`

  * `instance of` operator

    ```java
    if (o instanceof Set){
      Set<?> s = (Set<?>) o;
      ...
    }
    ```

* Summary:

  * **Safe**`Set<Object>` is a parameterized type representing a set that can contain objects of any type
  * **Safe**`Set<?>` is a wildcard type representing a set that contain only objects of some unknown type
  * `Set` is a raw type

27. **Eliminate unchecked warnings**

    * unchecked cast warnings, unchecked method invocation warnings, unchecked parameterized vararg type warnings, unchecked conversion warnings

      ```java
      //The compiler will infer the correct actual type parameter
      //with the diamond operator <> (i.e. Lark)
      Set<Lark> exaltation = new HashSet<>();
      ```

    * **Make sure the code is typesafe**

      * Eliminate every unchecked warning that you can
      * If you cannot eliminate a warning, but you can **prove that the code that provoked the warning is typesafe,** then (and only then) suppress the warning with an `@SuppressWarnings("unchekced")` annotation.
        * If you suppress without proving that the code is typesafe, you are giving yourself a false sense of security
        * If not suppress the warnings that you know to be safe (instead of suppressing them), you won't notice when a new warning crops up that represents a real problem.
      * Always use the `SuppressWarnings` annotation on the smallest scope possible (Never use on an entire class). Add a comment to justify the annotation.

28. **Prefer lists to arrays**
    * **Arrays** are covariant: if `Sub` is a subtype of `Super`, `Sub[]` is a subtype of `Super[]`. 
      * ***Reified***: know and enforce element type at runtime
    * **Lists** are invariant: `List<Sub>` and `List<Super>` are two distinct types
      * ***Erasure*:** enforce type constraints only at compile time and discard (or *erase*) their element type information at runtime.
      * Erasure allowed generic types to interoperate freely with legacy code that didn't use generics, ensuring a smooth transition to generics in Java 5.
    * Types such as `E`, `List<E>` and `List<String>` are technically known as *non-reifiable* types whose runtime representation contains less information than its compile-time representation.
    * The only parameterized types that are reifiable are unbounded wildcard types such as `List<?>` and `Map<?,?>`.
    
29. **Favor generic types**

    * Workaround of a non-reifiable type array

      * Create an array of `Object` and cast it to the generic array type.
        * Cause *heap pollution*: the runtime type of the array does not match its compile-time types 
      * Change the type of the field elements from `E[]` to `Object[]`
      * Unchecked cast must not compromise the type safety of the program

    * Advantages: more readable, more concise

    * Java does not support lists natively, so some generic types **must** be implemented atop arrays.

    * Generic types such as `HashMap` are implemented atop arrays for performance.

    * Generic types that restrict the permissible values of their type parameters

      ```java
      class DelayQueue<E extends Delayed> implements BlockingQueue<E>
      ```

    * **Bounded type parameter**: `<E extends Delayed>` requires that the actual type parameter `E` be a subtype of `java.util.concurrent.Delayed` 

