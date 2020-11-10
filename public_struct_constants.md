# Public Struct Constants

The `const` modifier can only be applied to numbers, characters, strings and booleans. A typical scenario is where we want to create and share a constant with a struct type.

There are 2 possibilities:

- Declare it as`var`.
- Use a constant function.

The problem with a `var` declaration is clear: you can modify the value of the variable and your code behavior will change.

Another possibility is to use a constant function as follows:

	type T struct {i int}
		
	func ConstantT() T {
		return T{i: 0}
	}

