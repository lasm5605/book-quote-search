# Book Quote Search

A command-line C++ program that finds every quote in a book containing a word you search for, built on a suffix tree I implemented from scratch. Type in a word like `possession` and it returns each sentence from the text that contains it.

Built as a final project for a data structures course (2270). The sample text is Chapter 1 of *Pride and Prejudice*.

## Features

- **Word search across a book**: enter a word and get back every sentence that contains it
- **Sentence parsing**: splits the text into sentences on `.`, `!` and `?`, and tidies up leading spaces and stray quotation marks
- **One suffix tree per sentence**: each sentence gets its own tree, so a match maps straight back to the quote it came from
- **Tree printout**: before the search prompt, the program prints every edge of each tree (and the leaf value where an edge ends in a leaf), so you can see how the trees are built
- **Hand-built suffix tree**: no libraries for the data structure, just structs and pointers

## Tech stack

- **C++** with the standard library only (`iostream`, `string`, `vector`, `fstream`)
- **g++** to compile
- **[VisuAlgo's suffix tree tool](https://visualgo.net/en/suffixtree)** to check my trees' output while building and testing

## Data structure

Each suffix tree is made from three structs, defined in `SuffixTree.h`:

```cpp
struct node {
    bool is_leaf;   // leaf node, or internal node with child edges
    int leaf_val;   // for leaves: the index of the suffix this leaf ends
};

struct edge {
    node* parent;                    // internal node where the edge starts
    node* child;                     // internal or leaf node where the edge ends
    string data;                     // the substring along this edge
    vector<edge*> child_edges;       // edges one tier below on the same path
    vector<edge*> send_down_edges;   // edges waiting to be pushed down a tier
};

struct SuffixTree {
    node* root;                      // the tree's starting node
    node* leaf;                      // the initial child node
    vector<edge*> parent_edges;      // top-tier edges coming straight from the root
    string output;                   // built up during a search
};
```

Suffix tree invariants the structure aims to follow:

1. Exactly *n* leaves for an input of length *n*
2. Apart from the root, every internal node has at least 2 children
3. Every edge is labelled with a non-empty substring of the input
4. No two edges leaving the same node start with the same character
5. Concatenating the labels from root to leaf spells out that leaf's suffix

## Files

- `main.cpp`: entry point. Reads the book, builds a tree for each sentence, prompts for a search and prints matching quotes
- `SuffixTree.h`: the `node`, `edge` and `SuffixTree` structs
- `SuffixTree.cpp`: building, cleaning up, printing and searching the trees
- `Pride&Prejudice(Chapter 1).txt`: the sample text, cleaned of unusual characters and spacing

## How it works

1. **Parse**: `parse_sentences` reads the text file and saves each sentence to a vector.
2. **Build**: each sentence gets a `$` terminator and goes to `BuildTree`, which creates the root and first child and hands them to `init_SuffixTree`. That sets up the first edge, then calls `insert_SuffixTree` to add the remaining suffixes.
3. **Clean up**: `find_for_cleanup` looks for sibling edges whose labels start with the same character. `clean_up_parent_edges` and `clean_up_child_edges` then add a new internal node for the shared character and move those edges beneath it.
4. **Search**: `SearchTree` walks each tree, appending matching edge labels to `output`. If `output` equals the search word, the sentence for that tree is printed as a result.

## Running locally

You'll need a C++ compiler such as g++.

```bash
git clone https://github.com/lasm5605/BookQuoteSearch.git
cd BookQuoteSearch
g++ main.cpp -o main
./main
```

The program prints the tree edges first, then asks you to enter a search word. Run it from the repo folder so it can find the text file.

### Words to try

```
good
girls
Mister
possession
```

Once you've seen a few results you'll have a sense of what's in the chapter, so try your own words too.

## Known limitations

- Only Chapter 1 is included, since each text file has to be cleaned by hand first.
- The tree builds correctly to a depth of about 2. Deeper branching isn't handled yet.
- The search matches one word at a time and is case-sensitive.
- The tree is traversed through vectors of edge pointers rather than node pointers, which costs time and memory compared with a standard suffix tree.

## Future ideas

- Rebuild the tree with **Ukkonen's algorithm** for O(n) construction
- Traverse through node pointers instead of edge vectors
- Handle trees of any depth
- Search for multi-word phrases, case-insensitively
- Load a whole book, or let the user pick which text to search
- Add a flag to hide the tree printout so results show up right away
