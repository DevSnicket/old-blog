Separation of concerns is an important consideration at all magnitudes of scope during software development. The skill of applying it appropriately is transferable across these magnitudes, but the goals aren't necessarily the same, and so the approach and mind-set can be different.

At the larger scales the benefits are, multiple and so smaller trunks in source control, faster builds, clearer and co-ordinated parallel working of teams, earlier integration testing and flexibility in deployment. To gain these benefits the boundaries created when separating must suit the content. This doesn't necessarily match the most appropriate structure to present to the user. This often happens from there being cross-cutting concerns and by re-using sub-systems. There is no one size fits all process to this separation, but there are characteristics you can look out for that help in discovery and creation of the definitions.
<h5>One-to-many Unidirectional</h5>
One classification is a component that will have one-to-many relationships and all those relationships are unidirectional towards the many. Expanding upon that definition, more than one other component will depend upon it, but it doesn't depend on any components which aren't of the same classification. That may or may not sound familiar, but it is often considered a framework. For example:

```
.NET Framework
      ┃
  ┏━━━┻━━━┓
```

Components in a system that exist to reuse often get called libraries. This term can also apply to frameworks, the distinction is that of extensibility. However, this term might mislead as a framework can exist in a skeletal form where its incomplete and the act of extending it in another component completes it into a piece of functionality. 

When it seems like a component is going to be more framework-like it can help direct the approach taken to defining the contract exposed for the components that use will use it. For instance your code is likely to expose types for the consuming component to implement.

```
One important characteristic of a framework is that the methods defined by the user to tailor the framework will often be called from within the framework itself, rather than from the user's application code. The framework often plays the role of the main program in coordinating and sequencing application activity. This inversion of control gives frameworks the power to serve as extensible skeletons. The methods supplied by the user tailor the generic algorithms defined in the framework for a particular application.
```
&mdash; [Designing Reusable Classes](http://www.laputan.org/drc/drc.html)

Terminology like API get used for creating such a contract although that might imply one can be added without affecting the implementation. However, appropriately structured framework code is intrinsically linked to how it will be used, and so how you think about the contract will have implications on the implementation. Where thinking of it as an API might be useful, for example, might be to independently version the contract. This can make it easier to support multiple versions while validating code built against previous version still has the expected behaviour.

It can be tempting to define the contract in as an abstract way as possible in order to satisfy as many hypothetical usages as possible. Unless your product is the framework and so the usages are customers doing so is likely to go against the YAGNI principle ([You aren't gonna need it](http://en.wikipedia.org/wiki/You_aren't_gonna_need_it)). However, if you initial implementations do target a limited number of usages its important keep the boundary and so the responsibilities clear. If you find concepts from the consuming components appearing in the framework component its worth working out if there is a fundamental concept which has been missed.

In recent years web based client development has promoted using many smaller complementary frameworks. If you decided to create a framework for your organisation or peers try and replicate this philosophy. It needs to be a framework and not the framework ([definite article](http://en.wikipedia.org/wiki/Article_(grammar)#Definite_article)). Being strict with the scope and so responsibility is especially important where there are execution entry points (i.e. where the system starts). They are a convenient location for any framework code and when uncontrolled encourage a progression to a singular monolithic framework.
<h5>Piers</h5>
Blah
