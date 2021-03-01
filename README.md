# Python Aho-Corasick implementation

`noahong` is a Python implementation of the [Aho-Corasick](https://en.wikipedia.org/wiki/Aho%E2%80%93Corasick_algorithm) algorithm for string matching, based on a fork of the [NoAho](https://github.com/JDonner/NoAho) C++ implementation.


`noahong` supports macOS, Linux, Windows on Python 3.6+.


## API

The first thing to do is to instantiate a `NoAho` object and add some keys to it (optionally with different payloads for each).

```python3
from noahong import NoAho

trie = NoAho()

# fill with .add()
trie.add("foo", "id_foo")
trie.add("foobar", "id_foobar")

# or fill with __setitem__
trie["bar"] = "id_bar"
```

Once you have added the different keys and their payloads, the `NoAho` object needs to be compiled:

```python3
trie.compile()
```

Once it is compiled, keys can no longer be added to the `NoAho` object.

`noahong` then exposes four functions to find matching substrings in text:

### `find_short`

`trie.find_short(text)` finds the first substring of `text` that is matched by a key added to the `trie`. 

It returns a tuple `(start, stop, payload)` such that: 
- `payload` is the object inserted with `trie.add()`
- `start` and `stop` are indices of the match in the `text`: `text[start:stop] == key` 

For example, using the above `trie`:

```python3
trie.find_short("something foo")
# returns (10, 13, 'id_foo')
# "something foo"[10:13] == "foo"
```

and returns the first match even though a longer match may start at the same position:

```python3
trie.find_short("something foobar")
# returns (10, 13, 'id_foo')
```

### `find_long`

`trie.find_long(text)` finds the first longest substring of `text` that is matched by a key added to the `trie`. 

For example, using the above `trie`:

```python3
trie.find_long("something foobar")
# returns (10, 16, 'id_foobar')
```

### `findall_*`

Both `find_short` and `find_long` have a `findall_short` and `findall_long` counterparts that allow you to iterate on all non-overlapping matches found
in the text:

```python3
for x in trie.findall_long("something foo bar foobar"): 
    print(x)       

# prints                          
# (10, 13, 'id_foo')
# (14, 17, 'id_bar')
# (18, 24, 'id_foobar')
```

Because matches are non-overlapping:

```python3
list(trie.findall_short("foobar")) == [(0, 3, "id_foo"), (3, 6, "id_bar")]
```

whereas:

```python3
list(trie.findall_long("foobar")) == [(0, 6, "id_foobar")]
```