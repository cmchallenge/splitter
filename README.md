Container Trip Splitting
------------------------

At ClearMetal, we track millions of containers around the world real-time.

Many of our customers are tracking goods in containers crossing the pacific,
for example from a factory outside of Shanghai to the Port of Oakland.

Since containers are generally owned by third parties and used for multiple
unrelated trips and it is important for us to be able to distinguish between
multiple trips of the same container.  Our customers only care about about
movements of a specific container while it is part of their "container trip"
(while their goods are inside, not previous or subsequent trips of the same box).

Unfortunately, there is no unique identifier which can directly split events
into multiple "container trips". Clearmetal's solution currently leverages
machine learning to provide a robust solution to this problem.

In this code challenge you will build a simplified version of this "splitter".

You will be given a list of container events which you will group into into
multiple lists of events each defining a single container move.

Input
-----

Each container event object will contain:

field      | type      | description                          | example values
---------- | --------- | ------------------------------------ | ---------------------------------
id         | int       | unique event identifier              | 12345678, 11235813
container  | str       | unique container identifier          | 'CSQU3054383', 'HDMU2108510'
location   | str       | location of event                    | 'LA', 'San_Francisco'
event_time | str       | time of event (ISO8601 format)       | '2009-10-26T04:47:09'
event_type | str       | type of event                        | 'arrive', 'depart', 'delivered'
orders     | List[str] | list of associated order numbers     | ['order_a'], ['order_b', 'order_c']


Each `container` identifier will be associated with at most 100 events.


Output
------

List[List[int]]: A list of each group of event ids.
Each inner list is ordered by the time of the associated event.  The outer list is ordered by the minimum time of each group's events.


Splitting Logic
---------------

To split events into trips, you must implement the following rules:

1. Only one container number per split; events of different containers are independent
2. Split after an event_type is `'delivered'`
3. Split between two consecutive events > 90 days apart with different locations
4. Split between two consecutive events > 180 days apart
5. After all of a container's events are split as described above, all trips which share orders should be merged.
    1. order joining should be transitive: 3 trips respectively with `['order_a']`, `['order_a', 'order_b']`, `['order_b']` should all be joined together
    2. can join trips that both include a "delivered" event
    3. can join trips that occur at non-adjacent times
    4. does not join between trips with different container numbers

Example
-------

INPUT:
```
[
  {
    "id": 1,
    "container": "container_2",
    "location": "Savannah",
    "time": "2018-10-06T00:00:00",
    "type": "depart",
    "orders": ["order_c"]
  },
  {
    "id": 11,
    "container": "container_123456789",
    "location": "Savannah",
    "time": "2018-10-07T00:00:00",
    "type": "delivered",
    "orders": []
  },
  {
    "id": 2,
    "container": "container_2",
    "location": "Long_Beach",
    "time": "2018-10-09T00:00:00",
    "type": "delivered",
    "orders": ["order_d"]
  },
  {
    "id": 3,
    "container": "container_2",
    "location": "Hong_Kong",
    "time": "2019-11-13T00:00:00",
    "type": "arrive",
    "orders": []
  },
  {
    "id": 4,
    "container": "container_2",
    "location": "San_Francisco",
    "time": "2019-12-02T00:00:00",
    "type": "delivered",
    "orders": ["order_d"]
  },
]
```

OUTPUT:
```
[[1, 2, 4], [11], [3]]
```


Instructions
------------

Implement a function `run(input_filename, output_filename)` which will:
1. load JSON formatted input data from the provided filename (e.g. `input.json`)
2. split the container events into a list of container moves
3. write the list of event id groupings ordered by event time as JSON into the provided filename (e.g. `output.json`)


Code requirements:
- Written in Python (Python3 encouraged!)
- Add at least one unit test to `test_splitter.py`
- Only use Python standard libraries (no external pip requirements or requirements.txt, etc)
- Your code should be executable with `python run.py`

Please write code as you would in a production environment.

Tips
----

- There are many utilities availible for running unit tests.  Python v2.7+ supports the following command to run tests:
```
python -m unittest test_splitter.py
```

- We provide a sample input and corresponding output file for testing: `input.json` and `expected.json`.

- The `event_time` string can be parsed to a `datetime` using:
```
from datetime import datetime
datetime.strptime(raw_event_time_string, '%Y-%m-%dT%H:%M:%S')
```
