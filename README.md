# ExConstructorValidator

Reduces code duplication by adding some boilerplate struct methods to your
module.

### The 'Problem'

If you want to write some validation for your struct, you need to write the
boilerplate `new` and `put` methods manually.

```elixir
defmodule Point do
  @enforce_keys [:x, :y]
  defstruct [:x, :y, :z]

  def new(args) do
    args = Keyword.new(args)

    __MODULE__
    |> Kernel.struct!(args)
    |> validate_struct()
  end

  def put(struct, args) do
    args = Keyword.new(args)

    struct
    |> Kernel.struct!(args)
    |> validate_struct()
  end

  def validate_struct(struct) do
    if struct.x < 0 or struct.y < 0 or struct.z < 0 do
      raise ArgumentError
    end

    struct
  end
end
```

And if you don't want to bother with validation yet, you might want to still
add `new` and `put` methods to be consistent (or to make it easier to add
validation later).

```elixir
defmodule PointNoValidation do
  @enforce_keys [:x, :y]
  defstruct [:x, :y, :z]

  def new(args) do
    args = Keyword.new(args)

    __MODULE__
    |> Kernel.struct!(args)
    |> validate_struct()
  end

  def put(struct, args) do
    args = Keyword.new(args)

    struct
    |> Kernel.struct!(args)
    |> validate_struct()
  end

  def validate_struct(struct) do
    struct
  end
end
```

And you have to write this boilerplate for every module you have! We only do
that in Java! That can be a lot of duplication!

### The Solution

By the magic of Elixir macros, we can remove the duplication!

```elixir
defmodule Point do
  @enforce_keys [:x, :y]
  defstruct [:x, :y, :z]

  use ExConstructorValidator # Adds `new` and `put` dynamically

  def validate_struct(struct) do
    if struct.x < 0 or struct.y < 0 or struct.z < 0 do
      raise ArgumentError
    end

    struct
  end
end

Point.new(x: 1, y: 2)
# => %Point{x: 1, y: 2, z: nil} # Still works!
Point.new(x: -1, y: 2)
# Fails validation, as expected
```

And when we don't want validation...

```elixir
defmodule PointNoValidation do
  @enforce_keys [:x, :y]
  defstruct [:x, :y, :z]

  use ExConstructorValidator # Adds `new` and `put` dynamically
end

Point.new(x: 1, y: 2)
# => %Point{x: 1, y: 2, z: nil} # Still works!
```

## Configuration

The `use` has optional arguments. See the [top of
`ExConstructorValidator.__using__/1` to see all their default
values](https://github.com/dylan-chong/ex_constructor_validator/blob/master/lib/ex_constructor_validator.ex#L7).

You can use [appcues/ExConstructor](https://github.com/appcues/exconstructor)
at the same time using:

```
defmodule PointNoValidation do
  @enforce_keys [:x, :y]
  defstruct [:x, :y, :z]

  use ExConstructorValidator, use_ex_constructor_library: true
end
```

(do not put `use ExConstructor`).

## Installation

If [available in Hex](https://hex.pm/docs/publish), the package can be installed
by adding `ex_constructor_validator` to your list of dependencies in `mix.exs`:

```elixir
def deps do
  [
    {:ex_constructor_validator, "~> 0.1.0"}
  ]
end
```

Documentation can be generated with [ExDoc](https://github.com/elixir-lang/ex_doc)
and published on [HexDocs](https://hexdocs.pm). Once published, the docs can
be found at [https://hexdocs.pm/ex_constructor_validator](https://hexdocs.pm/ex_constructor_validator).

