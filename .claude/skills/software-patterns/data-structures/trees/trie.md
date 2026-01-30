# Trie (Prefix Tree)

## Overview

A trie (pronounced "try" from retrieval) is a tree-like data structure used for storing strings where each node represents a character. The path from root to any node spells out a prefix, making tries ideal for prefix-based operations like autocomplete, spell checking, and IP routing.

## Properties

- **Character-based nodes**: Each node represents one character
- **Root is empty**: Root node has no character
- **Path = prefix**: Path from root to node spells a prefix
- **End markers**: Nodes mark where valid words end
- **Shared prefixes**: Common prefixes share nodes
- **No duplicate storage**: Each prefix stored once

## Time Complexity

| Operation        | Complexity |
|------------------|------------|
| Insert           | O(m)       |
| Search           | O(m)       |
| Delete           | O(m)       |
| Prefix search    | O(m)       |
| Autocomplete     | O(m + k)   |

Where m = length of key, k = number of results

## Space Complexity

O(ALPHABET_SIZE * m * n) worst case, where:
- ALPHABET_SIZE = number of possible characters (26 for lowercase)
- m = average key length
- n = number of keys

Can be significantly less with shared prefixes.

## Operations

### Node Structure

```pseudocode
class TrieNode {
    children = {}       // Map of character -> TrieNode
    isEndOfWord = false // Marks complete words
    // Optional: count, value, metadata

    function constructor() {
        this.children = new Map()
        this.isEndOfWord = false
    }
}

// Array-based alternative (fixed alphabet)
class TrieNodeArray {
    children[26]        // For lowercase letters
    isEndOfWord = false

    function constructor() {
        this.children = new Array(26).fill(null)
    }

    function getIndex(char) {
        return char - 'a'
    }
}
```

### Insert

```pseudocode
function insert(root, word) {
    current = root

    for char in word {
        if char not in current.children {
            current.children[char] = new TrieNode()
        }
        current = current.children[char]
    }

    current.isEndOfWord = true
}
```

### Search (Exact Match)

```pseudocode
function search(root, word) {
    node = findNode(root, word)
    return node != null and node.isEndOfWord
}

function findNode(root, prefix) {
    current = root

    for char in prefix {
        if char not in current.children {
            return null
        }
        current = current.children[char]
    }

    return current
}
```

### Starts With (Prefix Search)

```pseudocode
function startsWith(root, prefix) {
    return findNode(root, prefix) != null
}
```

### Delete

```pseudocode
function delete(root, word) {
    deleteHelper(root, word, 0)
}

function deleteHelper(node, word, depth) {
    if node == null {
        return false
    }

    if depth == word.length {
        // Found the word
        if node.isEndOfWord {
            node.isEndOfWord = false
            // Return true if node has no children (can be deleted)
            return node.children.isEmpty()
        }
        return false
    }

    char = word[depth]
    if char not in node.children {
        return false
    }

    shouldDelete = deleteHelper(node.children[char], word, depth + 1)

    if shouldDelete {
        node.children.remove(char)
        // Return true if this node can also be deleted
        return not node.isEndOfWord and node.children.isEmpty()
    }

    return false
}
```

### Autocomplete (Get All Words with Prefix)

```pseudocode
function autocomplete(root, prefix) {
    results = []
    node = findNode(root, prefix)

    if node != null {
        collectWords(node, prefix, results)
    }

    return results
}

function collectWords(node, prefix, results) {
    if node.isEndOfWord {
        results.append(prefix)
    }

    for (char, childNode) in node.children {
        collectWords(childNode, prefix + char, results)
    }
}
```

## Implementation

```pseudocode
class Trie {
    root = new TrieNode()

    function insert(word) {
        current = this.root

        for char in word {
            if char not in current.children {
                current.children[char] = new TrieNode()
            }
            current = current.children[char]
        }

        current.isEndOfWord = true
    }

    function search(word) {
        node = findNode(word)
        return node != null and node.isEndOfWord
    }

    function startsWith(prefix) {
        return findNode(prefix) != null
    }

    function findNode(prefix) {
        current = this.root

        for char in prefix {
            if char not in current.children {
                return null
            }
            current = current.children[char]
        }

        return current
    }

    function delete(word) {
        deleteHelper(this.root, word, 0)
    }

    function deleteHelper(node, word, depth) {
        if depth == word.length {
            if node.isEndOfWord {
                node.isEndOfWord = false
                return node.children.isEmpty()
            }
            return false
        }

        char = word[depth]
        if char not in node.children {
            return false
        }

        shouldDelete = deleteHelper(node.children[char], word, depth + 1)

        if shouldDelete {
            node.children.remove(char)
            return not node.isEndOfWord and node.children.isEmpty()
        }

        return false
    }

    function autocomplete(prefix, maxResults = 10) {
        results = []
        node = findNode(prefix)

        if node != null {
            collectWords(node, prefix, results, maxResults)
        }

        return results
    }

    function collectWords(node, prefix, results, maxResults) {
        if results.length >= maxResults {
            return
        }

        if node.isEndOfWord {
            results.append(prefix)
        }

        for (char, childNode) in node.children {
            collectWords(childNode, prefix + char, results, maxResults)
        }
    }

    function countWordsWithPrefix(prefix) {
        node = findNode(prefix)
        if node == null {
            return 0
        }
        return countWords(node)
    }

    function countWords(node) {
        count = 0
        if node.isEndOfWord {
            count = 1
        }
        for childNode in node.children.values() {
            count = count + countWords(childNode)
        }
        return count
    }
}
```

## Variants

### Compressed Trie (Radix Tree / Patricia Trie)

Compresses chains of single-child nodes:

```pseudocode
class CompressedTrieNode {
    label = ""          // String instead of single char
    children = {}
    isEndOfWord = false
}

// "testing" and "tested" stored as:
//        root
//         |
//      "test"
//       / \
//    "ed" "ing"
```

### Suffix Trie

Store all suffixes of a string for substring operations.

### Ternary Search Trie

Three children per node (less, equal, greater) - more memory efficient.

## Classic Algorithms

### Longest Common Prefix

```pseudocode
function longestCommonPrefix(words) {
    if words.isEmpty() {
        return ""
    }

    trie = new Trie()
    for word in words {
        trie.insert(word)
    }

    prefix = ""
    current = trie.root

    while current.children.size() == 1 and not current.isEndOfWord {
        char = current.children.keys()[0]
        prefix = prefix + char
        current = current.children[char]
    }

    return prefix
}
```

### Word Search in Grid

```pseudocode
function findWords(board, words) {
    trie = new Trie()
    for word in words {
        trie.insert(word)
    }

    results = new Set()

    for i from 0 to board.rows - 1 {
        for j from 0 to board.cols - 1 {
            dfs(board, i, j, trie.root, "", results)
        }
    }

    return results
}

function dfs(board, i, j, node, path, results) {
    if i < 0 or i >= board.rows or j < 0 or j >= board.cols {
        return
    }

    char = board[i][j]
    if char == '#' or char not in node.children {
        return
    }

    path = path + char
    node = node.children[char]

    if node.isEndOfWord {
        results.add(path)
    }

    board[i][j] = '#'  // Mark visited

    dfs(board, i + 1, j, node, path, results)
    dfs(board, i - 1, j, node, path, results)
    dfs(board, i, j + 1, node, path, results)
    dfs(board, i, j - 1, node, path, results)

    board[i][j] = char  // Unmark
}
```

## Use Cases

- **Autocomplete**: Search suggestions, IDE completion
- **Spell checking**: Dictionary lookup, suggestions
- **IP routing**: Longest prefix matching
- **Word games**: Scrabble, Boggle solvers
- **Search engines**: Query suggestion, indexing
- **Bioinformatics**: DNA sequence matching
- **Phone directories**: Contact search
- **URL routing**: Web frameworks

## Advantages

- **Fast prefix operations**: O(m) regardless of dictionary size
- **No hash collisions**: Unlike hash tables
- **Alphabetical ordering**: Natural sorted traversal
- **Shared storage**: Common prefixes stored once
- **Prefix counting**: Easy to count words with prefix

## Disadvantages

- **Memory intensive**: Especially for sparse tries
- **Cache unfriendly**: Scattered memory access
- **Pointer overhead**: Many null pointers in array implementation
- **Not good for exact match**: Hash table is O(1) vs O(m)
- **Character set dependent**: Fixed alphabet size affects implementation

## Comparison with Alternatives

| Aspect          | Trie        | Hash Table  | BST         | Sorted Array |
|-----------------|-------------|-------------|-------------|--------------|
| Exact search    | O(m)        | O(1)        | O(m log n)  | O(m log n)   |
| Prefix search   | O(m)        | O(n)        | O(m log n)  | O(m log n)   |
| Insert          | O(m)        | O(m)        | O(m log n)  | O(n)         |
| Space           | O(m*n*A)    | O(m*n)      | O(m*n)      | O(m*n)       |
| Sorted order    | Yes         | No          | Yes         | Yes          |

Where m = key length, n = number of keys, A = alphabet size

## Common Pitfalls

- **Memory waste**: Empty children arrays consume space
- **Case sensitivity**: Forgetting to normalize case
- **End marker**: Forgetting to check isEndOfWord
- **Delete complexity**: Not cleaning up empty branches
- **Unicode**: Assuming ASCII when supporting Unicode
- **Prefix vs word**: Confusing startsWith with exact search

## Related Structures

- **Radix Tree (Patricia Trie)**: Compressed trie
- **Suffix Tree**: Trie of all suffixes
- **Suffix Array**: Space-efficient suffix structure
- **Ternary Search Tree**: Memory-efficient trie variant
- **DAWG (Directed Acyclic Word Graph)**: Minimal trie
- **Burst Trie**: Hybrid trie with bucket nodes

---

*Based on concepts from "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (CLRS), MIT Press.*
