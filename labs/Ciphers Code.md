# Ciphers

### caesar.cpp

```cpp
#include <iostream>
#include <string>
#include <cctype>

bool nondigits(const std::string& str) {
    for (size_t i = 0; i < str.length(); ++i) {
        if (!std::isdigit(str[i])) {
            return true;
        }
    }
    return false;
}

std::string caesar(std::string text, int key, const std::string encrypt) {
    std::string result;

    for (size_t i = 0; i < text.length(); ++i) {
        char ch = text[i];
        bool iscaps = std::isupper(ch);
        char base = iscaps ? 'A' : 'a';
        key = key % 26;

        if (std::isalpha(ch)) {
            if (encrypt == "-e") {
                ch = (ch - base + key + 26) % 26 + base;
            } else if (encrypt == "-d") {
                ch = (ch - base - key + 26) % 26 + base;
            }
            result += ch;
        } else {
            result += ch;
        }
    }
    return result;
}

int main(int argc, char* argv[]) {
    if (argc != 3) {
        std::cout << "USAGE: caesar [-ed] [key]\n";
        return 1;
    }

    std::string first = argv[1];
    std::string two = argv[2];

    if (first != "-e" && first != "-d") {
        std::cout << "USAGE: caesar [-ed] [key]\n";
        return 1;
    }

    if (nondigits(two)) {
        std::cout << "USAGE: caesar [-ed] [key]\n";
        return 1;
    }
    
    std::string line;
    int key = std::stoi(two);

    while(std::getline(std::cin, line)) {
        std::string result = caesar(line, key, first);
        std::cout << result << '\n';
    }

    return 0;
}
```

### scytale.cpp

```cpp
#include <iostream>
#include <string>
#include <cctype>

bool nondigits(const std::string& str) {
    for (size_t i = 0; i < str.length(); ++i) {
        if (!std::isdigit(str[i])) {
            return true;
        }
    }
    return false;
}

char** encryptarray(std::string input, int key) {
    int width;
    if (input.length() % key != 0) {
        width = input.length() / key + 1;
    } else {
        width = input.length() / key;
    }
        
    char arr[key][width];

    int k = 0; // index iterates over input
    int length = input.length();
    for (int i = 0; i < key; ++i) {
        for (int j = 0; j < width; ++j) {
            if (k < length) {
                arr[i][j] = input[k];
                k += 1;
            } else {
                arr[i][j] = ' ';
            }
        }
    }

    for (int j = 0; j < width; ++j) {
        for (int i = 0; i < key; ++i) {
            std::cout << arr[i][j];
        }
    }
    
    return 0;
}

char** decryptarray(std::string input, int key) {
    int width;
    if (input.length() % key != 0) {
        width = input.length() / key + 1;
    } else {
        width = input.length() / key;
    }
        
    char arr[key][width];

    int k = 0; // index iterates over input
    int length = input.length();
    for (int j = 0; j < width; ++j) {
        for (int i = 0; i < key; ++i) {
            if (k < length) {
                arr[i][j] = input[k];
                k += 1;
            } else {
                arr[i][j] = ' ';
            }
        }
    }

    for (int i = 0; i < key; ++i) {
        for (int j = 0; j < width; ++j) {
            std::cout << arr[i][j];
        }
    }
    
    return 0;
}

int main(int argc, char* argv[]) {
    if (argc != 3) {
        std::cout << "USAGE: scytale [-ed] [key]\n";
        return 1;
    }

    std::string first = argv[1];
    std::string two = argv[2];

    if (first != "-e" && first != "-d") {
        std::cout << "USAGE: scytale [-ed] [key]\n";
        return 1;
    }

    if (nondigits(two)) {
        std::cout << "USAGE: scytale [-ed] [key]\n";
        return 1;
    }
    
    int key = std::stoi(two);
    if (key <= 0) {
        std::cout << "USAGE: scytale [-ed] [key]\n";
        return 1;
    }

    std::string line;

    while(std::getline(std::cin, line)) {
        if (first == "-e") {
            encryptarray(line, key);
        } else if (first == "-d") {
            decryptarray(line, key);
        }
        std::cout << '\n';
    }

    return 0;
}
```

### scrabble.cpp

```cpp
#include <iostream>
#include <string>
#include <cctype>

bool nonchar(const std::string& str) {
    int length = str.length();
    for (int i = 0; i < length; ++i) {
        if (!std::isalpha(str[i])) {
            return true;
        }
    }
    return false;
}

bool invalid(std::string line) {
    int length = line.length();
    for (int i = 0; i < length; ++i) {
        if (line[i] == ' ') {
            return true;
        }
    }

    if (nonchar(line)) {
        return true;
    }

    return false;
}

std::string lower(std::string characters) {
    std::string result;
    for (size_t i = 0; i < characters.length(); ++i) {
        char ch = characters[i];
        bool iscaps = std::isupper(ch);
        if (iscaps) {
            result += std::tolower(ch);
        } else {
            result += ch;
        }
    }
    return result;
}

bool scrabble(std::string letters, std::string line) {
    letters = lower(letters);
    line = lower(line);

    int linelen = line.length();
    int letterslen = letters.length();
    for (int i = 0; i < linelen; ++i) {
        bool found = false;
        for (int j = 0; j < letterslen; ++j) {
            if (line[i] == letters[j]) {
                letters[j] = 'n';
                found = true;
                break;
            }
        }
        if (!found) {
            return false;
        }
    }
    return true;
}

int main(int argc, char* argv[]) {
    if (argc != 2) {
        std::cout << "USAGE: scrabble [letters]\n";
        return 1;
    }

    std::string letters = argv[1];
    if (nonchar(letters)) {
        std::cout << "USAGE: scrabble [letters]\n";
        return 1;
    }

    std::string line;
    while(std::getline(std::cin, line)) {
        if (invalid(line)) {
            std::cout << "Invalid word.\n";
            continue;
        } else {
            if (scrabble(letters, line)) {
                std::cout << line << ": Yes\n";
            } else {
                std::cout << line << ": No\n";
            }
        }
    }

    return 0;
}
```

### substitution.cpp

```cpp
#include <iostream>
#include <cctype>
#include <string>

std::string lower(std::string characters) {
    std::string result;
    for (size_t i = 0; i < characters.length(); ++i) {
        char ch = characters[i];
        bool iscaps = std::isupper(ch);
        if (iscaps) {
            result += std::tolower(ch);
        } else {
            result += ch;
        }
    }
    return result;
}

bool secondarg(std::string arg) {
    arg = lower(arg);
    int arglen = arg.length();
    if (arglen != 26) {
        return false;
    }

    std::string alph = "abcdefghijklmnopqrstuvwxyz";
    int alphlen = alph.length();

    for (int i = 0; i < arglen; ++i) {
        bool found = false;
        for (int j = 0; j < alphlen; ++j) {
            if (arg[i] == alph[j]) {
                alph[j] = '0';
                found = true;
                break;
            }
        }
        if (!found) {
            return false;
        }
    }
    return true;
}

std::string encrypt(std::string key, std::string line) {
    std::string result;

    size_t linelen = line.length();
    for (size_t i = 0; i < linelen; ++i) {
        char ch = line[i];
        if (std::isalpha(ch)) {
            bool isupper = std::isupper(ch);
            int index = std::tolower(ch) - 'a';
            char encrypt = std::tolower(key[index]);
           
            if (isupper) {
                encrypt = std::toupper(encrypt);
            }
            
            result += encrypt;
        } else {
            result += ch;
        }
    }
    return result;        
}

std::string decrypt(std::string key, std::string line) {
    std::string result;
    std::string alph = "abcdefghijklmnopqrstuvwxyz";
    std::string linelower = lower(line);
    std::string newkey = lower(key);

    size_t linelen = linelower.length();
    int keylen = key.length();

    for (size_t i = 0; i < linelen; ++i) {
        char ch = linelower[i];
        char chorg = line[i];
        if (std::isalpha(ch)) {
            int index = -1;
            for (int j = 0; j < keylen; ++j) {
                if (ch == newkey[j]) {
                    index = j;
                }
            }
            
            bool isupper = std::isupper(chorg);
            char decrypt = alph[index];
            
            if (index != -1) {
                if (isupper) {
                    decrypt = std::toupper(decrypt);
                }
                
                result += decrypt;
            }
        } else {
            result += chorg;
        }
    }
    return result;        
}

int main(int argc, char* argv[]) {
    if (argc != 3) {
        std::cout << "USAGE: substitution [-ed] [key]\n";
        return 1;
    }

    std::string first = argv[1];
    std::string two = argv[2];

    if (first != "-e" && first != "-d") {
        std::cout << "USAGE: substitution [-ed] [key]\n";
        return 1;
    }

    if (secondarg(two) == false) {
        std::cout << "USAGE: substitution [-ed] [key]\n";
        return 1;
    }

    std::string line;
    while(std::getline(std::cin, line)) {
        if (first == "-e") {
            std::string result = encrypt(two, line);
            std::cout << result << '\n';
        } else if (first == "-d") {
            std::string result = decrypt(two, line);
            std::cout << result << '\n';
        }
    }

    return 0;
}
```
