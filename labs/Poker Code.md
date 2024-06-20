# Poker Code

### Card.h

```cpp
#ifndef CARD_H
#define CARD_H

// This file defines the Card type used to represent playing cards.
// It also includes some useful constants and operators.

#include <iostream>

// An enum lets you use a name for a constant rather than
// having to remember a value.  Instead of remembering that
// 13 is the code for kings, you can write Rank::KING, and
// your code becomes much more readable.
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

// An enum for suits.
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

  // A convenience function for pretty printing.
  void print() {
    if(rank >= Rank::JACK && rank <= Rank::ACE) {
      std::cout << "JQKA"[rank - Rank::JACK];
    }
    else {
      std::cout << rank;
    }

    const static char* SUITS[] = {"♣", "♦", "♥", "♠"};
    if(suit >= Suit::CLUBS && suit <= Suit::SPADES) {
      std::cout << SUITS[suit];
    }
    else {
      std::cout << '?' << '\n';
    }
  }
};

#endif
```

### Hand.h

```cpp
#ifndef HAND_H
#define HAND_H

#include "Card.h"

class Hand {
    private:
    Card* cards;
    size_t capacity;
    size_t numcards;
    void checkCapacity(size_t minCapacity);

    public:
    Hand();
    Hand(const Hand& other);
    ~Hand();
    void add(Card card);
    void clear();
    size_t count() const;
    Card get(size_t index) const;
    void print() const;

    void sort();
    Hand bysuit(Suit suit) const;
    Hand byrank(Rank rank) const;
    Hand highcard() const;

    Hand Straight() const;
    Hand Flush() const;
    Hand NofaKind(size_t N) const;
    Hand Omit(const Hand& omitcards);

};

Hand* make_hand();
Hand* best_hand(const Hand& cards);

#endif
```

### Hand.cpp

```cpp
#include "Card.h"
#include "Hand.h"

// implementation of Hand.h

// default constructor: initialize empty hand
Hand::Hand() {
    cards = nullptr;
    capacity = 0;
    numcards = 0;
}

// copy constructor: initialize as copy of other
Hand::Hand(const Hand& other) {
    capacity = other.capacity;
    numcards = other.numcards;

    cards = new Card[capacity];
    for (size_t i = 0; i < numcards; ++i) {
        cards[i] = other.cards[i];
    }
}

// free all heap memory owned by hand
Hand::~Hand() {
    delete [] cards;
}

// add card to the hand
void Hand::add(Card card) {
    checkCapacity(numcards + 1);
    cards[numcards++] = card;
    
    sort();
}

// remove all cards from the hand
void Hand::clear() {
    delete [] cards;
    cards = nullptr;
    capacity = 0;
    numcards = 0;
}

// return number of cards in the hand
size_t Hand::count() const {
    return numcards;
}

// return card at index index (assume valid indices)
Card Hand::get(size_t index) const {
    return cards[index];
}

// print cards in the hand in order, separated by one space
// character. print nothing after the final card (no space
// and no newline)
void Hand::print() const {
    for (size_t i = 0; i < numcards; ++i) {
        cards[i].print();
        if (i != numcards - 1) {
            std::cout << ' ';
        }
    }
}

// sorts descending
void Hand::sort() {
    for (size_t i = 1; i < numcards; ++i) {
        Card key = cards[i];
        int j = i - 1;
        
        while (j >= 0 && cards[j] < key) {
            cards[j + 1] = cards[j];
            j = j - 1;
        }
        cards[j + 1] = key;
    }
}

// increases capacity
void Hand::checkCapacity(size_t minCapacity) {
    if (capacity >= minCapacity) {
        return;
    } else {
        size_t newCapacity;
        if (capacity == 0) {
            newCapacity = 1;
        } else {
            newCapacity = capacity * 2;
        }
        Card* temp = new Card[newCapacity];
        for (size_t i = 0; i < numcards; ++i) {
            temp[i] = cards[i];
        }

    delete [] cards;
    cards = temp;
    capacity = newCapacity;
    }
}

Hand* make_hand() {
    return new Hand();
}
```

### best_hand.cpp

```cpp
#include "Card.h"
#include "Hand.h"

Hand Hand::bysuit(Suit suit) const {
    Hand sublist;
    for (size_t i = 0; i < numcards; ++i) {
        if (cards[i].suit == suit) {
            sublist.add(cards[i]);
        }
    }
    return sublist;
}

Hand Hand::byrank(Rank rank) const{
    Hand sublist;
    for (size_t i = 0; i < numcards; ++i) {
        if (cards[i].rank == rank) {
            sublist.add(cards[i]);
        }
    }
    return sublist;
}

Hand Hand::highcard() const {
    Hand sublist;
    sublist.add(cards[0]);
    return sublist;
}

Hand Hand::NofaKind(size_t N) const {
    Hand result;
    Hand copy = *this;

    if (numcards < N) {
        return result;
    }

    for (size_t i = 0; i <= numcards - N; ++i) {
        size_t count = 1;
        for (size_t j = i + 1; j < numcards; ++j) {
            if (copy.cards[i].rank == copy.cards[j].rank) {
                ++count;
                if (count == N) {
                    for (size_t k = 0; k < N; ++k) {
                        result.add(copy.cards[i + k]);
                    }
                    return result;
                }
            } else {
                break;
            }
        }
    }
    return result;
}

Hand Hand::Straight() const {
    Hand result;
    Hand acelow;
    Hand copy = *this;
    copy.sort();

    bool hasTwo = false, hasThree = false, hasFour = false, hasFive = false;
    if (numcards < 5) {
        return result;
    }

    if (copy.cards[0].rank == Rank::ACE) {
        Card twoCard, threeCard, fourCard, fiveCard;

        for (size_t i = 1; i < numcards; ++i) {
            if (copy.cards[i].rank == Rank::TWO && (!hasTwo || copy.cards[i].suit > twoCard.suit)) {
                hasTwo = true;
                twoCard = copy.cards[i];
            } else if (copy.cards[i].rank == Rank::THREE && (!hasThree || copy.cards[i].suit > threeCard.suit)) {
                hasThree = true;
                threeCard = copy.cards[i];
            } else if (copy.cards[i].rank == Rank::FOUR && (!hasFour || copy.cards[i].suit > fourCard.suit)) {
                hasFour = true;
                fourCard = copy.cards[i];
            } else if (copy.cards[i].rank == Rank::FIVE && (!hasFive || copy.cards[i].suit > fiveCard.suit)) {
                hasFive = true;
                fiveCard = copy.cards[i];
            }
        }

        if (hasTwo && hasThree && hasFour && hasFive) {
            acelow.add(fiveCard);
            acelow.add(fourCard);
            acelow.add(threeCard);
            acelow.add(twoCard);
            acelow.add(copy.cards[0]); // Add Ace
        }
    }

    for (size_t i = 0; i <= numcards - 5; ++i) {
        bool findstraight = true;
        size_t j = i;
        size_t count = 1;

        while (j < numcards - 1 && count < 5) {
            if (copy.cards[j].rank - 1 == copy.cards[j + 1].rank) {
                count++;
                j++;
            } else if (copy.cards[j].rank == copy.cards[j + 1].rank) {
                j++;
                continue;
            } else {
                findstraight = false;
                break;
            }
        }
        
        if (findstraight && count == 5) {
            size_t cardsAdded = 0;
            for (size_t k = i; k < j + 1 && cardsAdded < 5; k++) {
                if (k == i || copy.cards[k].rank != copy.cards[k - 1].rank) {
                    result.add(copy.cards[k]);
                    cardsAdded++;
                }
            }
            return result;
        }
    }
    if (hasTwo && hasThree && hasFour && hasFive) {
        return acelow;
    } else {
        return result;
    }
}

Hand Hand::Flush() const {
    Hand result;
    Hand copy = *this;
    copy.sort();

    if (numcards < 5) {
        return result;
    }

    Hand suits[4];
    size_t suittotal[4] = {0, 0, 0, 0};
    
    size_t suitcount[4] = {0};
    for (size_t i = 0; i < numcards; ++i) {
        suitcount[copy.cards[i].suit]++;
    }

    for (int s = 3; s >= 0; --s) {
        if (suitcount[s] >= 5) {
            size_t added = 0;
            for (size_t i = 0; i < numcards && added < 5; ++i) {
                if (copy.cards[i].suit == s) {
                    suits[s].add(copy.cards[i]);
                    suittotal[s] += copy.cards[i].rank;
                    added++;
                }
            }
        }
    }

    size_t maxtotal = 0;
    int maxsuit = -1;
    for (int s = 3; s >= 0; --s) {
        if (suittotal[s] > maxtotal) {
            maxtotal = suittotal[s];
            maxsuit = s;
        }
    }

    if (maxsuit != -1) {
        return suits[maxsuit];
    } else {
        return Hand();
    }
}

Hand Hand::Omit(const Hand& omitcards) {
    Hand result;

    for (size_t i = 0; i < numcards; ++i) {
        bool omit = false;
        for (size_t j = 0; j < omitcards.count(); ++j) {
            if (cards[i] == omitcards.get(j)) {
                omit = true;
                break;
            }
        }

        if (!omit) {
            result.add(cards[i]);
        }
    }
    return result;
}


Hand* best_hand(const Hand& cards) {
    if (cards.count() == 0) {
        return new Hand();
    } 
    
    Hand temp = cards;
    temp.sort();

    // straight flush
    Hand straight = temp.Straight();
    if (straight.count() >= 5) {
        Hand straightflush = straight.Flush();
        if (straightflush.count() == 5) {
            return new Hand(straightflush);
        }
    }
    Hand flush = temp.Flush();
    if (flush.count() >= 5) {
        Hand straightflush = flush.Straight();
        if (straightflush.count() == 5) {
            return new Hand(straightflush);
        }
    }

    // four of a kind
    Hand four = temp.NofaKind(4);
    if (four.count() == 4) {
        return new Hand(four);
    }

    // full house
    Hand fullhouse;
    Hand three = temp.NofaKind(3);
    if (three.count() == 3) {
        Hand omitthree = temp.Omit(three);
        Hand pair = omitthree.NofaKind(2);
        if (pair.count() == 2) {
            // add pair and ommitthree to create new hand
            for (size_t i = 0; i < 3; ++i) {
                fullhouse.add(three.get(i));
            }
            for (size_t i = 0; i < 2; ++i) {
                fullhouse.add(pair.get(i));
            }
            return new Hand(fullhouse);
        }
    }

    // flush
    if (flush.count() == 5) {
        return new Hand(flush);
    }

    // straight
    if (straight.count() == 5) {
        return new Hand(straight);
    }

    if (three.count() == 3) {
        return new Hand(three);
    }
    
    // two pair + pair
    Hand twopair;
    Hand firstpair = temp.NofaKind(2);
    if (firstpair.count() == 2) {
        Hand omitfirst = temp.Omit(firstpair);
        Hand secondpair = omitfirst.NofaKind(2);
        if (secondpair.count() == 2) {
            for (size_t i = 0; i < 2; ++i) {
                twopair.add(firstpair.get(i));
            }
            for (size_t i = 0; i < 2; ++i) {
                twopair.add(secondpair.get(i));
            }
            return new Hand(twopair);
        }
        return new Hand(firstpair);
    }

    // high card
    return new Hand(temp.highcard());
}
```
