Lust
===

Lust is a small library that tests Lua code.

Usage
---

Copy the `lust.lua` file to a project directory and require it, which returns a table that includes all of the functionality.

```Lua
local lust = require 'lust'

lust.describe('my project', function()
  lust.before(function()
    -- This gets run before every test.
  end)

  lust.describe('module1', function() -- Can be nested
    lust.it('feature1', function()
      lust.expect(1).to.be.a('number') -- Pass
      lust.expect('astring').to.equal('astring') -- Pass
    end)

    lust.it('feature2', function()
      lust.expect(nil).to.exist() -- Fail
    end)
  end)
end)
```

Documentation
---

##### `lust.describe(name, func)`

Used to declare a group of tests.  `name` is a string used to describe the group, and `func` is a function containing all tests and `describe` blocks in the group.  Groups created using `describe` can be nested.

##### `lust.it(name, func)`

Used to declare a test, which consists of a set of assertions.  `name` is a string used to describe the test, and `func` is a function containing the assertions.

### Assertions

Lust uses "expect style" assertions.  An assertion begins with `lust.expect(value)` and other modifiers can be chained after that:

##### `lust.expect(x).to.exist()`

Fails only if `x` is `nil`.

##### `lust.expect(x).to.equal(y)`

Performs a strict equality test, failing if `x` and `y` have different types or values.  Tables are tested by recursively ensuring that both tables contain the same set of keys and values.  Metatables are not taken into consideration.

##### `lust.expect(x).to.be(y)`

Performs an equality test using the `==` operator.  Fails if `x ~= y`.

##### `lust.expect(x).to.be.truthy()` and `lust.expect(x).to.be.falsy()`

Truthy fails if `x` is `nil` or `false`.  Falsy fails if `x` is not `nil` or `false`.

##### `lust.expect(x).to.be.a(y)`

If `y` is a string, fails if `type(x)` is not equal to `y`.  If `y` is a table, walks up `x`'s metatable chain and fails if `y` is not encountered.

##### `lust.expect(x).to.have(y)`

If `x` is a table, ensures that at least one of its keys contains the value `y` using the `==` operator.  If `x` is not a table, this assertion fails.

##### `lust.expect(f).to.fail()`

Ensures that the function `f` causes an error when it is run.

##### `lust.expect(x).to_not.*`

Negates the assertion.

### Spies

##### `lust.spy(table, key, run)` and `lust.spy(function, run)`

Spies on a function and tracks the number of times it was called and the arguments it was called with.  There are 3 ways to specify arguments to this function:

- Specify `nil`.
- Specify a function.
- Specify a table and a name of a function in that table.

The return value is a table that will contain one element for each call to the function. Each element of this table is a table containing the arguments passed to that particular invocation of the function.  The table can also be called as a function, in which case it will call the function it is spying on.  The third argument, `run`, is a function that will be called immediately upon creation of the spy.  Example:

```lua
local object = {
  method = function() end
}

-- Basic usage:
local spy = lust.spy(object, 'method')
object.method(3, 4) -- spy is now {{3, 4}}
object.method('foo') -- spy is now {{3, 4}, {'foo'}}
lust.expect(#spy).to.equal(2)
lust.expect(spy[1][2]).to.equal(4)

-- Using a run function:
local run = function()
  object.method(1, 2, 3)
  object.method(4, 5, 6)
end

lust.expect(lust.spy(object, 'method', run)).to.equal({{1, 2, 3}, {4, 5, 6}})

-- Using a function input:
local add = function(a, b)
  return a + b
end

local spy = lust.spy(add)

spy(1, 2) -- => {{1, 2}}
spy('rain', 'bows') -- => {{1, 2}, {'rain', 'bows'}}

lust.expect(#spy).to.equal(2)
lust.expect(spy[2]).to.equal({'rain', 'bows'})
```

### Befores and Afters

You can define functions that are called before and after every call to `lust.it` using `lust.before` and `lust.after`.  They are scoped to the `describe` block that contains them as well as any inner `describe` blocks.

##### `lust.before(fn)`

Set a function that is called before every test inside this `describe` block.  `fn` will be passed a single string containing the name of the test about to be run.

##### `lust.after(fn)`

Set a function that is called after every test inside this `describe` block.  `fn` will be passed a single string containing the name of the test that was finished.

License
---

MIT, see LICENSE for details.
