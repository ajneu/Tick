Design Notes
============

Trait-based
-----------

The concept predicates in Tick are defined as regular type traits(ie they are integrel constants) instead of a `constexpr bool` function. They are almost functionally the same in use. However, as a trait, it allows for better flexibility and expressiveness, through higher-order programming. This is what enables passing the traits to other functions which can be used to match return types to traits as well as types.

No tag-dispatching
------------------

Currently, Tick does not support tag-dispatching for overloading functions. That is because tag-dispatching doesn't work well with specialization. So instead of tag-dispatching, conditional overloading can be used. Here is example of implementing `advance` using the [`conditional_adaptor`](http://pfultz2.github.io/Fit/doc/html/conditional/) from the Fit library:

    TICK_TRAIT(is_incrementable)
    {
        template<class T>
        auto requires(T&& x) -> TICK_VALID(
            x++,
            ++x
        );
    };

    TICK_TRAIT(is_advanceable, is_incrementable<_>)
    {
        template<class T, class I>
        auto requires(T&& x, I&& i) -> TICK_VALID(
            x += i
        );
    };

    struct advance_advanceable
    {
        template<class Iterator, TICK_REQUIRES(is_advanceable<Iterator, int>())>
        void operator()(Iterator& it, int n) const
        {
            it += n;
        }
    };

    struct advance_incrementable
    {
        template<class Iterator, TICK_REQUIRES(is_incrementable<Iterator>())>
        void operator()(Iterator& it, int n) const
        {
            while (n--) ++it;
        }
    };

    static fit::static_<fit::conditional_adaptor<advance_advanceable, advance_incrementable>> advance = {};

So if the `advance_advanceable` function is not callable then it will try to call the `advance_incrementable` function.