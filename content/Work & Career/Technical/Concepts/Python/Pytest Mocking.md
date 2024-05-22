https://pytest-mock.readthedocs.io/en/latest/

`patch.object()` and `patch()` are pretty much the same, but `patch.object()` requires the target to be imported

`patch` will replace the function call or attribute with a `Mock` object


https://stackoverflow.com/questions/8180769/mocking-a-class-mock-or-patch

when doing dependency injection, it is preferred to use `Mock()` instead of patching

https://stackoverflow.com/questions/8180769/mocking-a-class-mock-or-patch