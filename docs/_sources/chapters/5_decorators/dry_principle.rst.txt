3. The DRY principle with decorators
************************************

We have seen how decorators allow us to abstract away certain logic into a separate
component. The main advantage of this is that we can then apply the decorator multiple
times into different objects in order to reuse code. This follows the **Don't Repeat Yourself
(DRY)** principle since we define certain knowledge once and only once.

The retry mechanism implemented in the previous sections is a good example of a
decorator that can be applied multiple times to reuse code. Instead of making each
particular function include its retry logic, we create a decorator and apply it several times.
This makes sense once we have made sure that the decorator can work with methods and
functions equally.

The class decorator that defined how events are to be represented also complies with the
DRY principle in the sense that it defines one specific place for the logic for serializing an
event, without needing to duplicate code scattered among different classes. Since we expect
to reuse this decorator and apply it to many classes, its development (and complexity) pay
off.

This last remark is important to bear in mind when trying to use decorators in order to
reuse code: we have to be absolutely sure that we will actually be saving code.

Any decorator (especially if it is not carefully designed) adds another level of indirection to
the code, and hence more complexity. Readers of the code might want to follow the path of
the decorator to fully understand the logic of the function (although these considerations
are addressed in the following section), so keep in mind that this complexity has to pay off.
If there is not going to be too much reuse, then do not go for a decorator and opt for a
simpler option (maybe just a separate function or another small class is enough).

But how do we know what too much reuse is? Is there a rule to determine when to refactor
existing code into a decorator? There is nothing specific to decorators in Python, but we
could apply a general rule of thumb in software engineering that states that a
component should be tried out at least three times before considering creating a generic
abstraction in the sort of a reusable component.

The bottom line is that reusing code through decorators is acceptable, but only when you
take into account the following considerations:

- Do not create the decorator in the first place from scratch. Wait until the pattern emerges and the abstraction for the decorator becomes clear, and then refactor.
- Consider that the decorator has to be applied several times (at least three times) before implementing it.
- Keep the code in the decorators to a minimum.
