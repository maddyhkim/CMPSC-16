# Library

### xps.h

```cpp
#ifndef XPS_H
#define XPS_H

// This file contains declarations and descriptions of
// the XP string functions to implement.


// This provides the size_t type:
#include <cstddef>


// ======== STRING MANIPULATION FUNCTIONS ==================

// Compare two XP strings "asciibetically."
// Return a negative number if lhs comes before rhs.
// Return a positive number if lhs comes after rhs.
// Return zero if the two strings are equal.
int xps_compare(const char* lhs, const char* rhs);

// Create a new XP string that holds the concatenation of two XP strings.
// The new string should contain all the characters
// from lhs followed by all the characters from rhs.
char* xps_concat(const char* lhs, const char* rhs);

// Find a substring inside another XP string.
// If the substring is found, return the index of the first (leftmost) match.
// If the substring is not found, return XPS_NPOS (see below).
// If the substring is empty, it matches at any index.
// If a start index is given, only consider matches occuring
// at or to the right of that index.
size_t xps_find(const char* str, const char* substr);
size_t xps_find(const char* str, const char* substr, size_t start);

// Release the memory used to hold an XP string.
// After calling this function, the memory pointed
// to by str should no longer be used.
void xps_free(char* str);

// Create a new XP string from a C string.
// The new string should contain all the characters
// from the C string, but stored in the XPS format.
char* xps_from_cstr(const char* cstr);

// Get a single character from an XP string.
// This function should not perform bounds checking;
// assume the provided index is always in bounds.
char xps_getchar(const char* str, size_t index);

// Get the length of an XP string.
size_t xps_length(const char* str);

// Set a single character in an XP string.
// Like xps_getchar(), this function should not perform
// bounds checking; just assume that index is valid.
void xps_setchar(char* str, size_t index, char c);

// Get a substring of an XP string as a new XP string.
// The new string should contain all the characters from str starting at
// index start and up to but not including the character at index stop.
// If stop is not given, the new string should contain all the characters
// from index start until the end of the original string.
// If stop is past the end of the original string, only include characters
// up to the end of the original string; if start is past the end of the
// original string, or if stop is less than or equal to start,
// return an empty string.
char* xps_slice(const char* str, size_t start);
char* xps_slice(const char* str, size_t start, size_t stop);



// ======== INPUT/OUTPUT FUNCTIONS =========================

// Print an XP string in the debug format from the readme.
void xps_debug(const char* str);

// Read a line of standard input as an XP string.
// If there is no more input, this function returns a null pointer.
char* xps_readline();

// Write an XP string to standard output.
void xps_write(const char* str);

// Write an XP string to standard output, followed by a newline.
void xps_writeline(const char* str);



// ======== OTHER ====================================

// A marker to represent "pattern not found."
// This is the largest value that fits in a size_t.
const size_t XPS_NPOS = 0xffffffffffffffffULL;

#endif
```

### xps.cpp

```cpp
#include "xps.h"

// Get a single character from an XP string.
char xps_getchar(const char* str, size_t index) {
    return str[index + index/15 + 1];
}

// Set a single character in an XP string.
void xps_setchar(char* str, size_t index, char c) {
    str[index + index/15 + 1] = c;
}

// Get the length of an XP string.
size_t xps_length(const char* str) {
    size_t index = 0;
    size_t count = 0;
        // finds length

    while (true) {
        unsigned char chunklen = (unsigned char)str[index];
        if (chunklen == 0) {
            break;
        }

        count += chunklen;
        index += chunklen + 1;
        if (chunklen < 15) {
            break;
        }
    }
    return count;
}

// Create a new XP string from a C string.
char* xps_from_cstr(const char* cstr) {
    size_t cstrlen = 0;
    while (cstr[cstrlen] != '\0') {
        ++cstrlen;
    }
    size_t chunks = cstrlen / 15 + 1;
    size_t xplen = cstrlen + chunks;

    char* xps = new char[xplen + 1];
    
    size_t prefixindex = 0;
    for (size_t i = 0; i < chunks; ++i) {
        size_t chunklen;
        if ((i + 1) * 15 <= cstrlen) {
            xps[prefixindex] = (char)15;
            prefixindex += 16;
        } else {
            chunklen = cstrlen % 15;
            xps[prefixindex] = (char)chunklen;
            prefixindex += 1 + chunklen;
        }
    }

    size_t xpsindex = 0;
    for (size_t i = 0; i < cstrlen; ++i) {
        xps_setchar(xps, xpsindex, cstr[i]);
        ++xpsindex;
    }

    return xps;
}

// Compare two XP strings "asciibetically."
int xps_compare(const char* lhs, const char* rhs) {
    size_t length = 0;
    size_t Llen = xps_length(lhs);
    size_t Rlen = xps_length(rhs);
    if (Llen < Rlen) {
        length = Llen;
    } else if (Llen > Rlen) {
        length = Rlen;
    } else {
        length = Llen;
    }

    for (size_t i = 0; i < length; ++i) {
        char left = xps_getchar(lhs, i);
        char right = xps_getchar(rhs, i);

        if (left != right) {
            return (left - right);
        }
        if (left == '\0' && right == '\0') {
            return 0;
        }
    }
    return (Llen - Rlen);
}

// Create a new XP string that holds the concatenation of two XP strings.
char* xps_concat(const char* lhs, const char* rhs) {
    size_t Llen = xps_length(lhs);
    size_t Rlen = xps_length(rhs);
    size_t len = Llen + Rlen;

    char* result = new char[len + len/15 + 1];
    
    size_t index = 0;
    while (len >= 15) {
        result[index] = 15;
        index += 16;
        len -= 15;
    }
    result[index] = len % 15;

    for (size_t i = 0; i < Llen; ++i) {
        xps_setchar(result, i, xps_getchar(lhs, i));
    }
    for (size_t j = 0; j < Rlen; ++j) {
        xps_setchar(result, j + Llen, xps_getchar(rhs, j));
    }

    return result;
}

// Find a substring inside another XP string.
size_t xps_find(const char* str, const char* pattern) {
    if (!pattern[0]) {
        return 0;
    }
    
    size_t strlen = xps_length(str);
    size_t patternlen = xps_length(pattern);

    if (patternlen == 0) {
        return 0;
    }
    if (strlen < patternlen) {
        return XPS_NPOS;
    }

    size_t i, j;
    for (i = 0; i <= strlen - patternlen; ++i) {
        bool match = true;
        for (j = 0; j < patternlen; ++j) {
            if (xps_getchar(str, i + j) != xps_getchar(pattern, j)) {
                match = false;
                break;
            }
        }
        if (match) {
            return i;
        }
    }
    return XPS_NPOS;
}

// Only consider matches occuring at or to the right of start index.
size_t xps_find(const char* str, const char* pattern, size_t start) {
    if (!pattern[0]) {
        return start;
    }
    
    size_t strlen = xps_length(str);
    size_t patternlen = xps_length(pattern);

    if (patternlen == 0) {
        return start;
    }
    if (strlen < patternlen || start > strlen - patternlen) {
        return XPS_NPOS;
    }

    size_t i, j;
    for (i = start; i <= strlen - patternlen; ++i) {
        bool match = true;
        for (j = 0; j < patternlen; ++j) {
            if (xps_getchar(str, i + j) != xps_getchar(pattern, j)) {
                match = false;
                break;
            }
        }
        if (match) {
            return i;
        }
    }
    return XPS_NPOS;
}

void xps_free(char* str) {
    delete [] str;
}

// Get a substring of an XP string as a new XP string.
char* xps_slice(const char* str, size_t start) {
    size_t len = xps_length(str);

    if (start >= len) {
        char* empty = new char[1];
        empty[0] = 0;
        return empty;
    }

    size_t slicelen = len - start;
    char* result = new char[slicelen + slicelen/15 + 1];
    
    size_t resultindex = 0;
    size_t chunkcounter = 0;
    for (size_t i = 0; i < slicelen; ++i) {
        if (chunkcounter == 0) {
            result[resultindex] = 0;
            ++resultindex;
        }
        result[resultindex] = xps_getchar(str, start + i);
        
        ++chunkcounter;
        result[resultindex - chunkcounter] = (char)(chunkcounter);
        
        ++resultindex;

        if (chunkcounter == 15) {
            chunkcounter = 0;
        }
    }

    if (slicelen % 15 == 0) {
        result[resultindex] = 0;
    }

    return result;
}

char* xps_slice(const char* str, size_t start, size_t stop) {
    size_t len = xps_length(str);

    if (start >= len || stop <= start) {
        char* empty = new char[1];
        empty[0] = 0;
        return empty;
    }

    if (stop > len || stop == 0) {
        stop = len;
    }

    size_t slicelen = stop - start;
    char* result = new char[slicelen + slicelen/15 + 1];
    
    size_t resultindex = 0;
    size_t chunkcounter = 0;
    for (size_t i = 0; i < slicelen; ++i) {
        if (chunkcounter == 0) {
            result[resultindex] = 0;
            ++resultindex;
        }
        result[resultindex] = xps_getchar(str, start + i);
        
        ++chunkcounter;
        result[resultindex - chunkcounter] = (char)(chunkcounter);
        
        ++resultindex;

        if (chunkcounter == 15) {
            chunkcounter = 0;
        }
    }

    if (slicelen % 15 == 0) {
        result[resultindex] = 0;
    }

    return result;
}
```

### xpsio.cpp

```cpp
#include "xps.h"

#include <iomanip>
#include <iostream>
#include <string>

// Implementation if the XPS input/output functions.


void xps_debug(const char* str) {
  // Save the original configuration of std::cout:
  // https://stackoverflow.com/questions/2273330
  std::ios old_state(nullptr);
  old_state.copyfmt(std::cout);

  // Reconfigure std::cout to print zero-filled hex:
  std::cout << std::hex << std::setfill('0');

  while(true) {
    unsigned char len = *str;
    std::cout << std::setw(2) << size_t(str[0]);

    if(len > 15) {
      // Error case for invalid block lengths.
      std::cout << "\n!!\n";
      break;
    }

    for(size_t i = 1; i <= len; ++i) {
      std::cout << ' ' << std::setw(2) << size_t(str[i]);
    }

    std::cout << "\n##";

    for(size_t i = 1; i <= len; ++i) {
      if(str[i] == '\0') {
        std::cout << " \\0";
      }
      else if(str[i] == '\t') {
        std::cout << " \\t";
      }
      else if(str[i] == '\n') {
        std::cout << " \\n";
      }
      else if(str[i] == '\r') {
        std::cout << " \\r";
      }
      else if(str[i] < ' ' || str[i] > '~') {
        std::cout << " ??";
      }
      else {
        std::cout << ' ' << str[i] << ' ';
      }
    }

    std::cout << '\n';

    if(len != 15) {
      break;
    }
    else {
      str += 16;
    }
  }

  // Restore std::cout to its original state:
  std::cout.copyfmt(old_state);
}

char* xps_readline() {
  std::string line;

  if(!std::getline(std::cin, line)) {
    return nullptr;
  }

  size_t len = line.length();
  char*  xps = new char[len + len / 15 + 1];

  size_t dst = 0;
  for(size_t src = 0; src < len; ++src) {
    if(src % 15 == 0) {
      xps[dst] = 15;
      dst += 1;
    }

    xps[dst] = line[src];
    dst += 1;
  }

  xps[16 * (len / 15)] = len % 15;
  return xps;
}

void xps_write(const char* str) {
  while(true) {
    unsigned char len = *str;
    std::cout.write(str + 1, len);

    if(len != 15) {
      break;
    }
    else {
      str += 16;
    }
  }
}

void xps_writeline(const char* str) {
  xps_write(str);
  std::cout << '\n';
}

```

### truncate.cpp

```cpp
#include "xps.h"

int main() {
    while (true) {
        char* input = xps_readline();
        if (input == NULL) {
            break;
        }
        size_t len = xps_length(input);

        if (len <= 20) {
            xps_writeline(input);
        } else {
            char* ten = xps_slice(input, 0, 10);
            char* seven = xps_slice(input, len - 7, len);

            const char* cstr = "...";
            char* xps = xps_from_cstr(cstr);
            char* one = xps_concat(ten, xps);
            char* two = xps_concat(one, seven);

            xps_writeline(two);
            
            xps_free(ten);
            xps_free(seven);
            xps_free(xps);
            xps_free(one);
            xps_free(two);
        }
        xps_free(input);
    }
    return 0;
}
```

### replace.cpp

```cpp

#include "xps.h"

int main(int argc, char* argv[]) {
    if (argc != 3) {
        const char* cstr = "USAGE: replace [old] [new]";
        char* xps = xps_from_cstr(cstr);
        xps_writeline(xps);
        xps_free(xps);
        return 1;
    }
    
    char* xstrold = xps_from_cstr(argv[1]);
    char* xstrnew = xps_from_cstr(argv[2]);
    
    while (true) {
        char* input = xps_readline();
        if (input == NULL) {
            break;
        }

        size_t start = 0;
        char* result = xps_from_cstr("");
        size_t find;

        while((find = xps_find(input, xstrold, start)) != XPS_NPOS) {
            char* one = xps_slice(input, start, find);
            char* two = xps_concat(result, one);
            xps_free(result);
            xps_free(one);
            result = two;

            char* three = xps_concat(result, xstrnew);
            xps_free(result);
            result = three;

            start = find + xps_length(xstrold);
        }

        char* remaining = xps_slice(input, start);
        char* finalresult = xps_concat(result, remaining);
        xps_free(remaining);
        xps_free(result);

        xps_writeline(finalresult);
        xps_free(finalresult);
        xps_free(input);
    }
    xps_free(xstrold);
    xps_free(xstrnew);
    return 0;
}
```

### match.cpp

```cpp
#include "xps.h"

bool wildcard(const char* str, const char* pattern, const char* cstrpattern) {
    size_t count = 0;
    size_t find = 0;
    size_t len = xps_length(pattern);
    
    for (size_t i = 0; i < len; ++i) {
        if (cstrpattern[i] == '*') {
            char* ch = xps_slice(pattern, count, i);
            size_t slice = xps_find(str, ch, find);

            if (slice == XPS_NPOS) {
                xps_free(ch);
                return false;
            }

            find = xps_find(str, ch, find) + 1;
            count = i + 1;
            xps_free(ch);
        }
    }

    char* last = xps_slice(pattern, count, xps_length(pattern));
    if (xps_find(str, last, find) == XPS_NPOS) {
        xps_free(last);
        return false;
    }

    xps_free(last);
    return true;
}

int main(int argc, char* argv[]) {
    if (argc != 2) {
        const char* cstr = "USAGE: match [pattern]";
        char* xps = xps_from_cstr(cstr);
        xps_writeline(xps);
        xps_free(xps);
        return 1;
    }

    char* cstrpattern = argv[1];
    char* pattern = xps_from_cstr(argv[1]);

    const char* cstr = "*";
    char* xps = xps_from_cstr(cstr);
    
    bool star = true;
    if (xps_find(pattern, xps) == XPS_NPOS) {
        star = false;
    }

    while (!star) {
        char* input = xps_readline();
        if (input == NULL) {
            break;
        }

        size_t find = xps_find(input, pattern);
        if (find == XPS_NPOS) {
            const char* cstr = "No match.";
            char* xps = xps_from_cstr(cstr);
            xps_writeline(xps);
            xps_free(xps);
        } else {
            const char* cstr = "Match!";
            char* xps = xps_from_cstr(cstr);
            xps_writeline(xps);
            xps_free(xps);
        }
        xps_free(input);
    }

    while (star) {
        char* input = xps_readline();
        if (input == NULL) {
            break;
        }

        if (wildcard(input, pattern, cstrpattern)) {
            const char* cstr = "Match!";
            char* xps = xps_from_cstr(cstr);
            xps_writeline(xps);
            xps_free(xps);
        } else {
            const char* cstr = "No match.";
            char* xps = xps_from_cstr(cstr);
            xps_writeline(xps);
            xps_free(xps);
        }
        xps_free(input);
    }

    xps_free(pattern);
    xps_free(xps);
    return 0;
}
```
