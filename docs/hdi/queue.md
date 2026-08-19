# Use the Queue class

Lyra has an optional queue system that works seamlessly with the library. This queue system introduce quality-of-life features that every music application should ideally have like queue shuffling, queue jumping, and looping.


To use the queue system with Lyra, you must first subclass the `Player` class within your application like so:

```py
from lava_lyra import Player


class CustomPlayer(Player): ...
```

After you have initialized your subclass, you can add a `queue` variable to your class so you can access your queue when you initialize your player:

```py
from lava_lyra import Player, Queue


class CustomPlayer(Player):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.queue = Queue()
```

There are also properties the `Queue` class has to access certain values:

:::{list-table}
:header-rows: 1

* - Property
  - Type
  - Description

* - `Queue.size`
  - `int`
  - Returns the amount of items in the queue.

* - `Queue.is_empty`
  - `bool`
  - Returns `True` if the queue has no members.

* - `Queue.is_full`
  - `bool`
  - Returns `True` if the queue's item count has reached `max_size`.

* - `Queue.is_looping`
  - `bool`
  - Returns `True` if the queue is looping either a track or the whole queue.

* - `Queue.loop_mode`
  - `Optional[LoopMode]`
  - Returns the `LoopMode` enum currently set on the queue, or `None` if looping is disabled.

:::

## Adding a song to the queue

To add a song to the queue, we must use `Queue.put()`

```py
Queue.put()
```

After you have initialized your function, we need to pass the `Track` positionally — `put()` doesn't accept `item` as a keyword argument:

```py

Queue.put(<your Track here>)

```

After running the function, your track should be in the queue.

### Adding a song to a specific position

If you want to insert a track at a specific position instead of the end of the queue, use
`Queue.put_at_index()`

```py
Queue.put_at_index(...)
```

After you have initialized your function, we need to include the `index` and `item` parameters:

```py

Queue.put_at_index(index=<your index here>, item=<your Track here>)

```

If you just want to insert a track at the very front of the queue (so it plays next), you can use
`Queue.put_at_front()` instead, which only needs the `item` parameter:

```py

Queue.put_at_front(item=<your Track here>)

```

### Adding multiple tracks at once

To add several tracks to the end of the queue in one call (for example, all the tracks from a
search result), we must use `Queue.extend()`

```py
Queue.extend(...)
```

After you have initialized your function, we need to fill in the proper parameters:

:::{list-table}
:header-rows: 1

* - Name
  - Type
  - Description

* - `iterable`
  - `Iterable[Track]`
  - The tracks to add.

* - `atomic`
  - `bool`
  - If set to `True`, either every track is added or none are — if the queue's `max_size` would be exceeded, `QueueFull` is raised and nothing is added. If set to `False`, as many tracks are added as fit and the rest are silently dropped, with no exception raised. Default value is `True`.

:::

```py

Queue.extend(iterable=<your list of Tracks here>, atomic=<True/False>)

```

:::{important}

Even with `atomic=True`, if overflow is enabled for the queue (see `Queue.max_size`), items can
still be dropped to make room rather than raising `QueueFull`.

:::

## Getting a track from the queue

To get a track from the queue, we need to do a few things.

To get a track using its position within the queue, you first need to get the position as a number, also known as its index. If you dont have the index and instead want to search for its index using keywords, you have to implement a fuzzy searching algorithm to find said track using a search query as an input and have it compare that query against the titles of the tracks in the queue. After that, you can get the `Track` object by [getting it with its index](queue.md#getting-track-with-its-index)

### Getting index of track

If you have the `Track` object and want to get its index within the queue, we can use `Queue.find_position()`

```py
Queue.find_position()
```

After you have initialized your function, we need to include the `item` parameter, which is a `Track`:

```py

Queue.find_position(item=<your Track here>)

```

After running the function, it should return the position of the track as an integer.


### Getting track with its index

If you have the index of the track and want to get the `Track` object, you first need to get the raw queue list:

```py
queue = Queue.get_queue()
```

After you have your queue, you can use basic list splicing to get the track object:

```py

track = queue[<index>]

```

## Getting the next track in the queue

To get the next track in the queue, we need to use `Queue.get()`

```py
Queue.get()
```

After running this function, it'll return the first track from the queue and remove it.

:::{note}

If you have a queue loop mode set, this behavior will be overridden since the queue is not allowed to remove tracks from the queue if its looping.

:::

### Peeking at the next track

If you want to know what `Queue.get()` would return next, without actually removing it from the
queue or advancing the loop state, use `Queue.peek_next()`

```py
Queue.peek_next()
```

This is useful for preloading a gapless "next track" ahead of time. Calling `Queue.get()` twice
for this purpose would double-advance the queue under `LoopMode.QUEUE`, silently skipping tracks —
`Queue.peek_next()` avoids that.

:::{important}

Raises `QueueEmpty` if there is nothing to play next.

:::

### Popping a track from the queue

To remove and return a track at a specific index (default: the last item), use `Queue.pop()`

```py
Queue.pop(...)
```

```py

Queue.pop(index=<your index here>)

```

:::{important}

Raises `QueueEmpty` if there are no items in the queue, and `IndexError` if the index is out of range.

:::

## Removing a track from the queue


To remove a track from the queue, we must use `Queue.remove()`

```py
Queue.remove()
```

After you have initialized your function, we need to include the `item` parameter, which is a `Track`:

```py

Queue.remove(item=<your Track here>)

```

:::{important}

Your `Track` object must be in the queue if you want to remove it. Make sure you follow [](queue.md#getting-a-track-from-the-queue) before running this function.

:::

After running this function, your track should be removed from the queue.


## Shuffling the queue

To shuffle the queue, we must use `Queue.shuffle()`

```py
Queue.shuffle()
```

After running this function, your queue should be in a different order than it was originally.

:::{tip}

This function works best if theres atleast **3** tracks in the queue. The more tracks, the more variation the shuffle has.

:::


## Looping the queue

To loop the queue, we must use `Queue.set_loop_mode()`

```py
Queue.set_loop_mode(...)
```

After you have initialized your function, we need to include the `mode` parameter, which is a `LoopMode` enum:

```py

Queue.set_loop_mode(mode=LoopMode.<mode>)

```

The two types of `LoopMode` enums are `LoopMode.QUEUE` and `LoopMode.TRACK`. `QUEUE` loops the entire queue and `TRACK` loops the current track.

After running the function, your queue will now loop using the mode you specify.

### Resetting the loop mode

To reset the loop mode, we must use `Queue.disable_loop()`

```py
Queue.disable_loop()
```

:::{important}

You must have a loop mode set before using this function. It will **not work** if you do not have a loop mode set

:::

After running the function, your queue should return to its normal functionality.

## Jumping to a track in the queue

To jump to a track in the queue, we must use `Queue.jump()`


```py
Queue.jump(...)
```

After you have initialized your function, we need to include the `item` parameter, which is a `Track`:

```py

Queue.jump(item=<your Track here>)

```

:::{important}

Your `Track` object must be in the queue if you want to jump to it. Make sure you follow [](queue.md#getting-a-track-from-the-queue) before running this function.

:::

:::{important}

Raises `QueueException` if the queue is currently looping a single track (`LoopMode.TRACK`).

:::

After running this function, any items before the specified item will be removed, effectively "jumping" to the specified item in the queue. The next item obtained using `Queue.get()` will be your specified track.

## Clearing the queue

To remove all items from the queue at once, we must use `Queue.clear()`

```py
Queue.clear()
```

After running this function, the queue will be empty. Unlike `Queue.disable_loop()`, this does not
change the current loop mode.

## Copying the queue

To create an independent copy of a queue, including its members, loop mode, and current track,
use `Queue.copy()`

```py
new_queue = Queue.copy()
```

This returns a new `Queue` instance — mutating the copy does not affect the original queue.

## Manually syncing the current track

`Queue.get()` normally keeps track of what's "currently playing" for you as you pull tracks from
the queue. If you play a track outside of that normal flow (for example, replaying a track pulled
from an external history stack), you can use `Queue.set_current()` to manually tell the queue
what's currently playing, so that loop-mode aware lookups like `Queue.peek_next()` stay in sync:

```py
Queue.set_current(...)
```

After you have initialized your function, we need to include the `item` parameter, which is a
`Track` (or `None` to clear it):

```py

Queue.set_current(item=<your Track here>)

```

## Clearing track filters

`Track` objects can carry their own per-track filters. To clear the filters set on every track
currently in the queue, use `Queue.clear_track_filters()`

```py
Queue.clear_track_filters()
```
