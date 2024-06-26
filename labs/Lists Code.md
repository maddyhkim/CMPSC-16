# Lists

### ListCard.h

```cpp
#ifndef CARD_H
#define CARD_H

// This file defines the Card type used to represent playing cards.
// It also includes some useful constants and operators.

#include <iostream>

enum Rank {
  TWO   =  2,
  THREE =  3,
  FOUR  =  4,
  FIVE  =  5,
  SIX   =  6,
  SEVEN =  7,
  EIGHT =  8,
  NINE  =  9,
  TEN   = 10,
  JACK  = 11,
  QUEEN = 12,
  KING  = 13,
  ACE   = 14
};

enum Suit {
  CLUBS    = 0,
  DIAMONDS = 1,
  HEARTS   = 2,
  SPADES   = 3
};


struct Card {
  // Member Variables
  Rank rank;
  Suit suit;

  // Default Constructor
  // This creates a "Zero of Clubs"
  Card() {
    rank = Rank(0);
    suit = Suit(0);
  }

  // User-Friendly Constructor 1
  Card(int rank, Suit suit) {
    this->rank = Rank(rank);
    this->suit = suit;
  }

  // User-Friendly Constructor 2
  Card(Rank rank, Suit suit) {
    this->rank = rank;
    this->suit = suit;
  }

  // A full set of comparison operators.
  bool operator <  (Card other) const {return rank <  other.rank || (rank == other.rank && suit <  other.suit);}
  bool operator <= (Card other) const {return rank <  other.rank || (rank == other.rank && suit <= other.suit);}
  bool operator >  (Card other) const {return rank >  other.rank || (rank == other.rank && suit >  other.suit);}
  bool operator >= (Card other) const {return rank >  other.rank || (rank == other.rank && suit >= other.suit);}
  bool operator == (Card other) const {return rank == other.rank && suit == other.suit;}
  bool operator != (Card other) const {return rank != other.rank || suit != other.suit;}

  // Fun with Unicode characters...
  void print() const {
    if(rank >= Rank::JACK && rank <= Rank::ACE) {
      std::cout << "JQKA"[rank - Rank::JACK];
    }
    else if(rank >= 0 && rank <= 10) {
      std::cout << rank;
    }
    else {
      std::cout << 'X';
    }

    const static char* SUITS[] = {"♣", "♦", "♥", "♠"};
    if(suit >= Suit::CLUBS && suit <= Suit::SPADES) {
      std::cout << SUITS[suit];
    }
    else {
      std::cout << '?';
    }
  }
};


struct Node {
  // Member Variables
  Card  card;
  Node* next;

  // Default Constructor
  Node() {
    // Leave it all uninitialized...
  }

  // User-Friendly Constructor
  Node(Card card, Node* next) {
    this->card = card;
    this->next = next;
  }
};

#endif
```

### ListHand.h

```cpp
#ifndef HAND_H
#define HAND_H

#include "ListCard.h"

class Hand {
    private:
    Node* headNode;
    size_t capacity;
    size_t numcards;

    Node* recinsert(Node* current, Node* newNode) {
        if (current == nullptr || newNode->card > current->card) {
            newNode->next = current;
            return newNode;
        } else if (current->next == nullptr || newNode->card > current->next->card) {
            newNode->next = current->next;
            current->next = newNode;
            return headNode;
        } else {
            recinsert(current->next, newNode);
            return headNode;
        }
    }

    Node* copy(const Node* source) {
        if (source == nullptr) {
            return nullptr;
        } else {
            Node* node = new Node(source->card, copy(source->next));
            return node;
        }
    }

    public:
    Hand();
    Hand(const Hand& other);
    ~Hand();
    void add(Card card);
    void clear();
    size_t count() const;
    Card get(size_t index) const;
    const Node* head() const;
    void print() const;
    void sort();
};

#endif
```

### ListHand.cpp

```cpp
#include "ListCard.h"
#include "ListHand.h"

// implementation of ListHand.h

// default constructor: initialize empty hand
Hand::Hand() : headNode(nullptr), numcards(0) {}

// copy constructor: initialize as copy of other
Hand::Hand(const Hand& other) : headNode(nullptr), numcards(other.numcards) {
    headNode = copy(other.headNode);
}

// free all heap memory owned by hand
Hand::~Hand() {
    clear();
}

// add card to the hand
void Hand::add(Card card) {
    Node* newNode = new Node{card, nullptr};
    headNode = recinsert(headNode, newNode);
    numcards++;
}

void recclear(Node* node) {
    if (node == nullptr) {
        return;
    }
    recclear(node->next);
    delete node;
}

// remove all cards from the hand
void Hand::clear() {
    recclear(headNode);
    headNode = nullptr;
    numcards = 0;
}

// return number of cards in the hand
size_t Hand::count() const {
    return numcards;
}

Card recget(Node* node, size_t index) {
    if (node == nullptr) {
        return Card();
    } else if (index == 0) {
        return node->card;
    } else {
        return recget(node->next, index - 1);
    }
}

// return card at index index (assume valid indices)
Card Hand::get(size_t index) const {
    if (index < numcards) {
        return recget(headNode, index);
    }

    return Card();
}

void recprint(const Node* node) {
    if (node == nullptr) {
        return;
    }

    node->card.print();

    if (node->next != nullptr) {
        std::cout << ' ';
        recprint(node->next);
    }
}

// print cards in the hand in order, separated by one space
// character. print nothing after the final card (no space
// and no newline)
void Hand::print() const {
    recprint(headNode);
}

Node* insertSort(Node* sorted, Node* node) {
    if (sorted == nullptr || sorted->card < node->card) {
        node->next = sorted;
        return node;
    } else {
        sorted->next = insertSort(sorted->next, node);
        return sorted;
    }
}

Node* sortlist(Node* head) {
    if (head == nullptr || head->next == nullptr) {
        return head;
    }
    Node* next = head->next;
    head->next = nullptr;
    Node* sorted = sortlist(next);
    return insertSort(sorted, head);
}

// sorts descending
void Hand::sort() {
    headNode = sortlist(headNode);
}

const Node* Hand::head() const {
    return headNode;
}
```
