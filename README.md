# EnumExtender Module

EnumExtender is a module that allows you to extend the global `Enum` so that you can add your own enumerators to it, while preserving the standard enumerators that are already available. Furthermore, the added user-made enumerators have the same functionality (as close as possible) and error checking/handling as the standard enumerators. Typically the module would be used by overwriting the global `Enum` of the current environment as I will display in the code later on.

------
This is the structure of objects both for the standard Enum as well as this module:

- **Enums** - collection of enumerators (= the `Enum` global)
- **Enum** - individual enumerator
- **EnumItem** - value of an enumerator
  - `Value` - numerical value of EnumItem
  - `Name` - name of EnumItem

(Note that the `Enum` global is not the same as the `Enum` object, the contents of the `Enum` global is an object of type `Enums`.)

------

**The most important API function: (see code for examples)**

`void Enums.new(string name, table list)`

Creates a new enumerator, where list is a table with numerical indices and string values.

------

`EnumItem Enums:FromValue(string enum, int value)`

Given a certain name of an enumerator, returns the EnumItem of that enumerator with a certain value, or nil if there is no EnumItem for that enumerator with that value. Also works for standard enumerators.

`table Enum:GetEnumItems()`

Identical to the same function that standard enumerators have, returns a table of EnumItems that an enumerator can take, sorted on their Value.

`Enum Enums:Find(string name)`

Check if a certain enumerator (user-made or standard) exists given the name for the enumerator to find.

`userdata Enums:GetStandardEnums()`

Returns the original contents of the Enum global, which may be useful in case existing variables are overwritten, (which is an option in the module) and you need to access the original values for some reason.

------

- `ENUM_OVERWRITE_ENABLED`
Whether user-made enumerators can be overwritten.

- `RBX_ENUM_OVERWRITED_ENABLED`
Whether standard enumerators can be overwritten.

- `EMPTY_ENUM_ENABLED`
Whether enumerators without values are allowed

- `VARIABLE_STYLE_NAMING`
Enforce variable styled naming of enum items, meaning they can start with a character or underscore and the rest of the name should consist of alphanumerical characters or underscores.

- `WARNINGS_ENABLED`
Whether or not warnings will be printed.

These options must be edited within the module itself and are located right after the documentation.

**Example code:**
```lua
-- You can access your own enumerators globally by calling this:
-- (typically you will want to overwrite "Enum" for convenience)
local Enum = require(<path to the module>)

-- The standard enumerators in Enum are preserved and can be accessed:
assert(Enum.FormFactor)

-- Food is not a standard Enum and therefore doesn't exist yet:
assert(not Enum:Find("Food"))

-- Make an Enum "Food" that can take the values "Apple", "Banana", or "Cherry":
Enum.new("Food", {"Apple", "Banana", "Cherry"})

-- Food is an existing Enum now:
assert(Enum:Find("Food"))

-- Values can be picked:
local a = Enum.Food.Apple
local b = Enum.Food.Banana
local c = Enum.Food.Cherry

-- Values can be compared:
assert(a == a)
assert(a ~= b)
assert(a ~= c)

-- And printed:
print("Apple  = ", a) -- Apple = Enum.Food.Apple
print("Banana = ", b) -- Banana = Enum.Food.Banana
print("Cherry = ", c) -- Cherry = Enum.Food.Cherry

-- The set of values of a custom Enum can be accessed via GetEnumItems:

Enum.new("Letters", {"A", "B", "C", "D", "E", "F", "G", "H", "I", "J"})

for _, EnumItem in pairs(Enum.Letters:GetEnumItems()) do
	print(tostring(EnumItem) .. " has name '" .. EnumItem.Name
		.. "' and a value of " .. EnumItem.Value .. "!")
	-- Enum.Letters.A has name 'A' and a value of 1!
	-- Enum.Letters.B has name 'B' and a value of 2!
	-- etc
end

-- You can also give custom number values to the enumerator values:

Enum.new("Numbers", {[1] = "One", [100] = "Hundred", [1000] = "Thousand"})

for _, EnumItem in pairs(Enum.Numbers:GetEnumItems()) do
	print(tostring(EnumItem) .. " has name '" .. EnumItem.Name
		.. "' and a value of " .. EnumItem.Value .. "!")
	-- Enum.Letters.A has name 'One' and a value of 1!
	-- Enum.Letters.B has name 'Hundred' and a value of 100!
	-- Enum.Letters.B has name 'Thousand' and a value of 1000!
end

-- Enum.FormFactor.Symmetric has a value of 0:
local item1 = Enum:FromValue("FormFactor", 0)
assert(item1 == Enum.FormFactor.Symmetric)

local item2 = Enum:FromValue("Numbers", 1000)
assert(item2 == Enum.Numbers.Thousand)

-- The FromValue function is useful, e.g. if you need to save/load enumerator values.
```

------

- **Make your code clear and understandable:** Some people will use numbers or strings as replacement for enumerators or to create enumerators that aren't included within the set of standard enumerators, in the following way:


```lua
-- Numbers as "enumerators":

if type == 0 then
   (...)
elseif type == 1 then
   (...)
end

-- Strings as "enumerators":

if type == "Fruit" then
   (...)
elseif type == "Vegetables" then
   (...)
end

-- Both are very limited :-(

-- Much better:
if type == Enum.FoodType.Fruit then
   (...)
elseif type == Enum.FoodType.Vegetables then
   (...)
end
```
- However, when simply using numbers as enumerators, your code is harder to understand, since you may not remember after a while what each number in the enumerator meant. When you use strings as enumerators this problem is less severe, but since these "enumerators" are not restricted in any way and no error checking/handling is enforced, it may be more difficult to debug your code still.

- **Powerful for debugging, error checking and handling:** This module mimics the functionality of the standard Enum. That means attempting to change any part of the enumerators will cause an error, submitting invalid information to these methods will generate meaningful errors/warnings, which is very useful for debugging your game and for finding mistakes in your code / game logic that may not be evidently visible when you use simply strings or numbers as "enumerators", which are not bound to any rules or restrictions.

-----

The code is located in /src/EnumExtender.lua.
