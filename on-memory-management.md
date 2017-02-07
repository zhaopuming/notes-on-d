# Notes on Memory Management

in the [post](https://forum.dlang.org/post/mailman.1154.1476359814.2994.digitalmars-d-learn@puremagic.com), Jonathan Davis wrote:

> It's also possible to completely disable use of the GC in D, but you lose out on a few features (and while the std lib doesn't use the GC heavily, it does use it, so if you remove the GC from the runtime, you can't use Phobos), so it's not particularly advisable. But you can get a _long_ way just by being smart about your GC use. A number of D programmers have managed to use D with full use of the GC in high performance code simply by doing stuff like make sure that a collection cycle doesn't kick in in hot spots in the program (e.g. by calling GC.disable when entering the hot spot and then GC.enable when leaving), and for programs that need to do real-time stuff that can't afford to have a particular thread be stopped by a GC collection, you just use a thread that's not managed by the GC for that critical thread, and it's able to keep going even if the rest of the program is temporarily stopped by a collection.


## Temporarily ignore function/templates without @nogc attribute

```d

import std.traits : isFunctionPointer, isDelegate, functionAttributes, FunctionAttribute, SetFunctionAttributes, functionLinkage;
static auto ngcptr(T)(T f) if (isFunctionPointer!T ||
            isDelegate!T)
{
    enum attrs = functionAttributes!T | FunctionAttribute.nogc;
    return cast(SetFunctionAttributes!(T, functionLinkage!T, attrs)) f;
};

/// For normal functions
auto ngc(alias Func, T...)(T xs) @nogc { return ngcptr(&Func)(xs); }
/// For templates
auto tngc(alias Func, T...)(T xs) @nogc { return ngcptr(&Func!T)(xs); }

/// test

int myadd(int a, int b) { return a + b; }

@nogc
void main() {
  tngc!writeln("hello"); 
  ngc!myadd(1, 2);
}
```


