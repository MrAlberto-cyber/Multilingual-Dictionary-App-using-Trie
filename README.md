#include <iostream>
#include <unordered_map>
#include <vector>
#include <string>
#include <algorithm>
#include <cctype>

// Trie node structure
class TrieNode {
public:
    std::unordered_map<char, TrieNode*> children; // Children nodes
    bool isEndOfWord; // Marks end of a word
    std::unordered_map<std::string, std::string> translations; // Language code -> translation

    TrieNode() : isEndOfWord(false) {}
    ~TrieNode() {
        for (auto& child : children) {
            delete child.second;
        }
    }
};

// Trie class for the multilingual dictionary
class MultilingualDictionary {
private:
    TrieNode* root;

    // Helper function to collect all words with a given prefix
    void collectWords(TrieNode* node, std::string prefix, std::vector<std::pair<std::string, std::unordered_map<std::string, std::string>>>& results) {
        if (node->isEndOfWord) {
            results.push_back({prefix, node->translations});
        }
        for (auto& child : node->children) {
            collectWords(child.second, prefix + child.first, results);
        }
    }

    // Helper function to convert string to lowercase
    std::string toLower(const std::string& str) {
        std::string result = str;
        std::transform(result.begin(), result.end(), result.begin(), [](unsigned char c) { return std::tolower(c); });
        return result;
    }

public:
    MultilingualDictionary() {
        root = new TrieNode();
    }

    ~MultilingualDictionary() {
        delete root;
    }

    // Insert a word with its translations
    void insert(const std::string& word, const std::unordered_map<std::string, std::string>& translations) {
        TrieNode* current = root;
        std::string lowerWord = toLower(word);

        // Traverse or create nodes for each character
        for (char c : lowerWord) {
            if (current->children.find(c) == current->children.end()) {
                current->children[c] = new TrieNode();
            }
            current = current->children[c];
        }
        current->isEndOfWord = true;
        current->translations = translations;
    }

    // Search for a word and return its translations
    std::unordered_map<std::string, std::string> search(const std::string& word) {
        TrieNode* current = root;
        std::string lowerWord = toLower(word);

        for (char c : lowerWord) {
            if (current->children.find(c) == current->children.end()) {
                return {};
            }
            current = current->children[c];
        }
        return current->isEndOfWord ? current->translations : std::unordered_map<std::string, std::string>();
    }

    // Get all words starting with a prefix
    std::vector<std::pair<std::string, std::unordered_map<std::string, std::string>>> startsWith(const std::string& prefix) {
        std::vector<std::pair<std::string, std::unordered_map<std::string, std::string>>> results;
        TrieNode* current = root;
        std::string lowerPrefix = toLower(prefix);

        // Traverse to the prefix node
        for (char c : lowerPrefix) {
            if (current->children.find(c) == current->children.end()) {
                return results;
            }
            current = current->children[c];
        }

        // Collect all words in the subtree
        collectWords(current, lowerPrefix, results);
        return results;
    }

    // Delete a word from the trie
    bool deleteWord(const std::string& word) {
        std::string lowerWord = toLower(word);
        return deleteHelper(root, lowerWord, 0);
    }

private:
    // Helper function for deletion
    bool deleteHelper(TrieNode* current, const std::string& word, int index) {
        if (index == word.length()) {
            if (!current->isEndOfWord) {
                return false; // Word not found
            }
            current->isEndOfWord = false;
            current->translations.clear();
            return current->children.empty(); // Return true if node can be deleted
        }

        char c = word[index];
        auto it = current->children.find(c);
        if (it == current->children.end()) {
            return false; // Word not found
        }

        bool shouldDeleteCurrentNode = deleteHelper(it->second, word, index + 1);

        if (shouldDeleteCurrentNode) {
            delete it->second;
            current->children.erase(c);
            return current->children.empty() && !current->isEndOfWord;
        }
        return false;
    }
};

// Main function with console interface
int main() {
    MultilingualDictionary dict;
    int choice;
    std::string word, lang, translation, prefix;

    // Sample data
    dict.insert("cat", {{"en", "cat"}, {"es", "gato"}, {"fr", "chat"}});
    dict.insert("dog", {{"en", "dog"}, {"es", "perro"}, {"fr", "chien"}});
    dict.insert("car", {{"en", "car"}, {"es", "coche"}, {"fr", "voiture"}});

    do {
        std::cout << "\nMultilingual Dictionary Menu:\n";
        std::cout << "1. Insert a word\n";
        std::cout << "2. Search for a word\n";
        std::cout << "3. Find words by prefix\n";
        std::cout << "4. Delete a word\n";
        std::cout << "5. Exit\n";
        std::cout << "Enter your choice: ";
        std::cin >> choice;
        std::cin.ignore(); // Clear newline from input buffer

        switch (choice) {
            case 1: {
                std::cout << "Enter word: ";
                std::getline(std::cin, word);
                std::unordered_map<std::string, std::string> translations;
                std::cout << "Enter translations (lang:translation, type 'done' to finish):\n";
                while (true) {
                    std::cout << "Language code (e.g., en, es, fr) or 'done': ";
                    std::getline(std::cin, lang);
                    if (lang == "done") break;
                    std::cout << "Translation: ";
                    std::getline(std::cin, translation);
                    translations[lang] = translation;
                }
                dict.insert(word, translations);
                std::cout << "Word inserted successfully!\n";
                break;
            }
            case 2: {
                std::cout << "Enter word to search: ";
                std::getline(std::cin, word);
                auto translations = dict.search(word);
                if (translations.empty()) {
                    std::cout << "Word not found!\n";
                } else {
                    std::cout << "Translations:\n";
                    for (const auto& t : translations) {
                        std::cout << t.first << ": " << t.second << "\n";
                    }
                }
                break;
            }
            case 3: {
                std::cout << "Enter prefix: ";
                std::getline(std::cin, prefix);
                auto results = dict.startsWith(prefix);
                if (results.empty()) {
                    std::cout << "No words found with prefix '" << prefix << "'\n";
                } else {
                    std::cout << "Words with prefix '" << prefix << "':\n";
                    for (const auto& result : results) {
                        std::cout << result.first << ": ";
                        for (const auto& t : result.second) {
                            std::cout << t.first << "=" << t.second << " ";
                        }
                        std::cout << "\n";
                    }
                }
                break;
            }
            case 4: {
                std::cout << "Enter word to delete: ";
                std::getline(std::cin, word);
                if (dict.deleteWord(word)) {
                    std::cout << "Word deleted successfully!\n";
                } else {
                    std::cout << "Word not found!\n";
                }
                break;
            }
            case 5:
                std::cout << "Exiting...\n";
                break;
            default:
                std::cout << "Invalid choice! Try again.\n";
        }
    } while (choice != 5);

    return 0;
}
