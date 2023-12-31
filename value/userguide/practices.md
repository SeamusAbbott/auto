# Best practices


## <a name="interchangeable"></a>"Equals means interchangeable"

Use AutoValue when you want value semantics. Under value semantics, if `a` and
`b` are instances of the same AutoValue class, and `a.equals(b)`, then `a` and
`b` are considered interchangeable, and `a` can be used in place of `b`
everywhere and vice versa. If your AutoValue use case does not satisfy these
contracts, then AutoValue may not be a good fit.

## <a name="mutable_properties"></a>Avoid mutable property types

Avoid mutable types, including arrays, for your properties, especially if you
make your accessor methods `public`. The generated accessors don't copy the
field value on its way out, so you'd be exposing your internal state.

Note that this doesn't mean your factory method can't *accept* mutable types as
input parameters. Example:

```java
@AutoValue
public abstract class ListExample {
  abstract ImmutableList<String> names();

  public static ListExample create(List<String> mutableNames) {
    return new AutoValue_ListExample(ImmutableList.copyOf(mutableNames));
  }
}
```

## <a name="simple"></a>Keep behavior simple and dependency-free

Your class can (and should) contain *simple* intrinsic behavior. But it
shouldn't require complex dependencies and shouldn't access static state.

You should essentially *never* need an alternative implementation of your
hand-written abstract class, whether hand-written or generated by a mocking
framework. If your behavior has enough complexity (or dependencies) that it
actually needs to be mocked or faked, split it into a separate type that is
*not* a value type. Otherwise it permits an instance with "real" behavior and
one with "mock/fake" behavior to be `equals`, which does not make sense.

## <a name="one_reference"></a>One reference only

Other code in the same package will be able to directly access the generated
class, but *should not*. It's best if each generated class has one and only one
reference from your source code: the call from your static factory method to the
generated constructor. If you have multiple factory methods, have them all
delegate to the same hand-written method, so there is still only one point of
contact with the generated code. This way, you have only one place to insert
precondition checks or other pre- or postprocessing.

## <a name="final"></a>Mark all concrete methods `final`

Consider that other developers will try to read and understand your value class
while looking only at your hand-written class, not the actual (generated)
implementation class. If you mark your concrete methods `final`, they won't have
to wonder whether the generated subclass might be overriding them. This is
especially helpful if you are *[underriding](howto.md#custom)* `equals`,
`hashCode` or `toString`!

## <a name="constructor"></a>Maybe add an explicit, inaccessible constructor

There are a few small advantages to adding a package-private, parameterless constructor to your abstract class. It prevents unwanted subclasses, and prevents an undocumented public constructor showing up in your generated API documentation. Whether these benefits are worth the extra noise in the file is a matter of your judgment.