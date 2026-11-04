# Design Plan: GCSE Edexcel Biology Revision App

**Module:** COM5013 Algorithms and Data Structures (CW1)

**Author:** [Hakil Rashid / 80013703]

**Language:** Python

# 1. Scenario and requirements

The app is a revision tool for a GCSE Biology student. Decks are expected to hold hundreds to a few thousand cards, with a 1,000,000-card stress test. During a session, the most frequent operations are looking up a card by ID and retrieving the next most urgent card; sorting and saving are occasional. Key features are card management, topic browsing, prefix search, scheduled revision with rescheduling on wrong answers, and undo.

**Key features**

| # | Feature        | Description                                             |
|--------|--------------------|-------------------------------------------------------------|
| F1     | Manage cards       | Add, edit, delete and retrieve cards by ID                  |
| F2     | Topic organisation | Cards grouped as topic, then subtopic, then card            |
| F3     | Keyword search     | Find cards by keyword or prefix (e.g. "mito")               |
| F4     | Revision session   | Show the most urgent card next; wrong answers return sooner |
| F5     | Sort and report    | List cards by topic, difficulty, or last-revised date       |
| F6     | Undo               | Reverse the last recorded answer                            |
| F7     | Persistence        | Save and load all cards to a JSON file                      |

**Non-functional requirements:** handle an empty deck, blank or zero values, and very large decks (up to 1,000,000 cards) without crashing.

# 2. Operations and candidate options

n = number of cards, m = length of a search key.

| Operation                | Option A                                           | Option B                                       | Option C                                               |
|------------------------------|--------------------------------------------------------|----------------------------------------------------|------------------------------------------------------------|
| Lookup by ID (F1)            | Hash map: avg O(1), worst O(n)                         | BST: avg O(log n), worst O(n)                      | Sorted array + binary search: O(log n) lookup, O(n) insert |
| Keyword / prefix search (F3) | Trie: O(m)                                             | Hash map (exact match only): avg O(1)              | Linear scan: O(n)                                          |
| Sort cards (F5)              | Bubble sort: best O(n)\*, avg/worst O(n²)              | Merge sort: O(n log n) all cases, O(n) extra space | Quick sort: avg O(n log n), worst O(n²)                    |
| Get most urgent card (F4)    | Min-heap: insert O(log n), extract O(log n), peek O(1) | Sorted list: insert O(n), extract O(1) at the end  | Unsorted list: insert O(1), extract O(n)                   |
| Topic hierarchy (F2)         | Tree + DFS/BFS: O(n) traversal                         | Flat list filtered by topic: O(n) per query        | Hash map of topic to cards: O(1) topic lookup              |
| Undo (F6)                    | Stack: push/pop O(1)                                   | List with index tracking: O(1)                     | n/a                                                        |

\*Best case O(n) for bubble sort assumes an early-exit check for an already-sorted list.

# 3. Provisional choices and justification

| Feature   | Chosen structure / algorithm                       | Why (context-based)                                                                                                                  |
|---------------|--------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------|
| F1 Lookup     | Hash map (Python dict, keyed by card ID)               | Frequent lookups by ID during sessions; no ordering needed; collisions are rare with unique Ids                                          |
| F2 Topics     | Tree (topic, subtopic, card IDs)                       | Matches the structure of the specification; supports browsing by topic                                                                   |
| F3 Search     | Trie                                                   | Supports prefix and autocomplete search, which a hash map cannot do; cost depends on key length, not deck size                           |
| F4 Scheduling | Min-heap keyed on due_time                             | Needs the most urgent card repeatedly with frequent reschedules; O(log n) beats an O(n) sorted-list insert at scale                      |
| F5 Sorting    | Merge sort (benchmarked against bubble and quick sort) | Guaranteed O(n log n) and stable (preserves order of equal items, useful for multi-key sorts); bubble sort retained only as a comparison |
| F6 Undo       | Stack                                                  | Last in, first out matches "undo the most recent answer" exactly                                                                         |
| F7 Storage    | JSON file                                              | Simple, human-readable, no database setup needed                                                                                         |

**Scheduling rule (draft):** a correct answer doubles the card's review interval; a wrong answer resets it to a short interval. The card's due_time is updated and the heap re-ordered.

# 4. Planned classes

Card, Deck (hash map), TopicTree, TrieIndex, Scheduler (heap), Sorter, UndoStack, Storage, plus cli.py for the menu and session loop. Code follows PEP 8 and snake_case.

# 5. Test plan

| Test                | Expected behaviour                                                 |
|-------------------------|------------------------------------------------------------------------|
| Empty deck              | Revision session reports "no cards" without crashing                   |
| Null / zero values      | Blank answers and score 0 are handled or rejected with a clear message |
| 1,000,000 cards         | Lookup, search, heap and merge sort complete; timings recorded         |
| Values over 64 bits     | Very large IDs and timestamps stored and retrieved correctly           |
| Duplicate Ids           | Rejected or overwritten consistently                                   |
| Search with no match    | Returns an empty list                                                  |
| Undo with empty history | Does nothing, with a clear message                                     |

# 6. Workflow

- Commit small and often, including bug fixes.

- Record benchmark results as they are produced
