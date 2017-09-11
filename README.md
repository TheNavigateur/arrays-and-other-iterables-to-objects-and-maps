# Arrays and other iterables to objects and maps

Main proposed methods:

- `Array.prototype.toObject`
- `Array.prototype.toMap`

Complementary proposed methods:

- `Object.fromIterable`
- `Map.fromIterable`

Complementary proposed convenience methods:

- `Array.prototype.toObjectByPropertyName`
- `Array.prototype.toMapByPropertyName`
- `Object.fromIterableByPropertyName`
- `Map.fromIterableByPropertyName`

This proposal (hopefully) satisfies all of the requirements outlined in the discussion here: https://esdiscuss.org/topic/array-prototype-toobjectbyproperty-element-element-property

# Existing way

Caching typical array content in an object for quick access to its elements by one of an element's property values can currently be achieved as follows:

```javascript
const elementsById =
    elements.reduce(
        (elementsById, element)=>{
            elementsById[element.id] = element;
            return elementsById;
        },
        Object.create(null)
    )
```

For a map, the following applies:

```javascript
const elementsById =
    new Map(
        elements.map(
            element=>[element.id, element]
        )
    )
```

# Proposed way

For both, a unified proposed type of mapping looks instead like this:

```javascript
const elementsById = elements.toObject(element=>element.id)
```

and for a map:

```javascript
const elementsById = elements.toMap(element=>element.id)
```

# Signature

The full signature in this proposal includes, in addition to the key callback, an optional value callback and optional starting object or map, for completeness, and is as follows:

```javascript
Array.prototype.to[Object|Map](keyFromElement[, valueFromElement[, startingDictionary]])
```

...where `valueFromElement` defaults to `element=>element` and `startingDictionary` defaults to an empty new `Object` or `Map`, if not provided.

# Why an optional value callback?

If the required dictionary value for each element is not the element itself, e.g. if I have an array of ids from which I need to page into another dictionary to get the values, I can use this to tersely construct my required dictionary.

# Why an optional starting object or map?

If I am collecting array data in separate batches, and I want to cache the data to the same dictionary each time, I can use this to tersely add to an existing dictionary instead of having to merge them separately.

# Static `Object.fromIterable` and `Map.fromIterable`

The advantage of having this in the `Array` `prototype` is the ability to chain this after array transformation methods (like `filter` etc.).

However, for iterables in general, the following can be additionally exposed:

```javascript
[Object|Map].fromIterable(iterable, keyFromElement[, valueFromElement[, startingDictionary]])
```

Example usage:

```javascript
const elementsById = Object.fromIterable(iterable, element=>element.id)
```
# By property name

An addition is to allow a string to be provided as the property name in place of `keyFromElement`. This can either be an overload in the same method or via separate methods `to[Object|Map]ByPropertyName` and `fromIterableByPropertyName`, which would otherwise have the same signature as their counterpart, e.g.:

```javascript
const elementsById = elements.toObjectByPropertyName('id')
```

# Why both objects and maps?

Objects are still a compelling choice for dictionaries: they have great performance, are well understood and have a very terse access syntax for both getting and setting values. Maps are compelling, on the other hand, when insertion ordering is required.

# Alternatives examined

`Object.fromEntries` was proposed as an analogy to the `Map` constructor. The usage would look as follows:

```javascript
const elementsById =
   Object.fromEntries(
       elements.map(
           element=>[element.id, element]
       )
   )
```

However, this is not necessarily preferable even over the already available `reduce` way of doing it, let alone the proposed way, because internally it requires an extra iteration step and creates memory pressure that may not be wanted in the case of large volumes of data.

In addition, it is not yet clear what use cases `Object.fromEntries` would fit besides the stated ones, and for the stated use cases it is more verbose to use than the proposed approach, and without adding any additional control.

# Overall justification

The need to cache array data for quick access is common and spans different types of applications. The data types dealt with by the proposal are fundamental. The proposal provides a complete way of constructing data, in the required dictionary type (map or object), from arrays and other iterables, from a very terse way in the most common types of use cases, to a progressively less terse way as more and more control is required.
