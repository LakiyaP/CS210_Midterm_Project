# CS210_Midterm_Project
# Lakiya Perry
#include <iostream>
#include <fstream>
#include "CSVReader.h"
#include <vector>
#include <string>
using namespace std;

int main() {
    string filename = "schools.csv"; // Path to your CSV file
    vector<vector<string>> csvData = CSVReader::readCSV(filename);

    // Create a SchoolList instance
    SchoolList schoolList;

    // Load data into the linked list (here we insert at the end)
    for (const auto& row : csvData) {
        // CSV columns: NAME, ADDRESS, CITY, STATE, COUNTY
        School school(row[0], row[1], row[2], row[3], row[4]);
        schoolList.insertLast(school);
    }

    cout << "All schools loaded:\n";
    schoolList.display();

    // Allow user to search for a school by name
    cout << "\nEnter a school name to search: ";
    string searchName;
    getline(cin, searchName);
    School* found = schoolList.findByName(searchName);
    if (found) {
        cout << "\nSchool found:\n";
        cout << "Name: " << found->name << "\n"
             << "Address: " << found->address << "\n"
             << "City: " << found->city << "\n"
             << "State: " << found->state << "\n"
             << "County: " << found->county << "\n";
    } else {
        cout << "\nSchool with name \"" << searchName << "\" not found.\n";
    }

    // Allow user to delete a school by name
    cout << "\nEnter a school name to delete: ";
    string deleteName;
    getline(cin, deleteName);
    if (schoolList.deleteByName(deleteName)) {
        cout << "School \"" << deleteName << "\" deleted successfully.\n";
    } else {
        cout << "School \"" << deleteName << "\" not found. No deletion performed.\n";
    }

    // Display the list after deletion
    cout << "\nRemaining schools:\n";
    schoolList.display();

    return 0;
}

#ifndef SCHOOL_H
#define SCHOOL_H

#include <string>

struct School {
    std::string name;
    std::string address;
    std::string city;
    std::string state;
    std::string county;
    School* next;

    School(std::string n, std::string a, std::string c, std::string s, std::string co)
        : name(n), address(a), city(c), state(s), county(co), next(nullptr) {}
};

#endif // SCHOOL_H
#include <iostream>
#include <fstream>
#include <string>
using namespace std;
#ifndef SCHOOLLIST_H
#define SCHOOLLIST_H

#include "School.h"
#include <iostream>

class SchoolList {
private:
    School* head;
public:
    SchoolList() : head(nullptr) {}

    // Insert at beginning of the list
    void insertFirst(const School& schoolData) {
        School* newNode = new School(schoolData.name, schoolData.address, schoolData.city, schoolData.state, schoolData.county);
        newNode->next = head;
        head = newNode;
    }

    // Insert at end of the list
    void insertLast(const School& schoolData) {
        School* newNode = new School(schoolData.name, schoolData.address, schoolData.city, schoolData.state, schoolData.county);
        if (!head) {
            head = newNode;
            return;
        }
        School* current = head;
        while (current->next) {
            current = current->next;
        }
        current->next = newNode;
    }

    // Delete a school by name (first occurrence)
    bool deleteByName(const string& schoolName) {
        School* current = head;
        School* prev = nullptr;
        while (current) {
            if (current->name == schoolName) {
                if (prev) {
                    prev->next = current->next;
                } else {
                    head = current->next;
                }
                delete current;
                return true;
            }
            prev = current;
            current = current->next;
        }
        return false; // Not found
    }

    // Find a school by name and return pointer (or nullptr if not found)
    School* findByName(const string& schoolName) {
        School* current = head;
        while (current) {
            if (current->name == schoolName) {
                return current;
            }
            current = current->next;
        }
        return nullptr;
    }

    // Display all schools
    void display() {
        School* current = head;
        while (current) {
            cout << "Name: " << current->name << "\n"
                 << "Address: " << current->address << "\n"
                 << "City: " << current->city << "\n"
                 << "State: " << current->state << "\n"
                 << "County: " << current->county << "\n"
                 << "---------------------------\n";
            current = current->next;
        }
    }

    // Destructor to free all nodes
    ~SchoolList() {
        School* current = head;
        while (current) {
            School* nextNode = current->next;
            delete current;
            current = nextNode;
        }
    }
};

#endif // SCHOOLLIST_H
#include <iostream>
#include <fstream>
#include <sstream>
#include <vector>
#include <string>
using namespace std;

#ifndef CSVREADER_H
#define CSVREADER_H

class CSVReader {
public:
    static vector<vector<string>> readCSV(const string& filename) {
        ifstream file(filename);
        vector<vector<string>> data;
        string line, word;

        if (!file.is_open()) {
            cerr << "Error: Could not open file " << filename << endl;
            return data;
        }

        // Skip header line
        if(getline(file, line)) {
            // header is read and discarded
        }

        // Read each subsequent line
        while (getline(file, line)) {
            stringstream ss(line);
            vector<string> row;
            while (getline(ss, word, ',')) {
                row.push_back(word);
            }
            // Only add rows with expected number of fields
            if (row.size() >= 5)
                data.push_back(row);
        }
        file.close();
        return data;
    }
};
#endif //CSVREADER_H

#ifndef SCHOOLBST_H
#define SCHOOLBST_H

#include "School.h"
#include <iostream>

class BSTNode {
public:
    School school;
    BSTNode* left;
    BSTNode* right;

    BSTNode(School s) : school(s), left(nullptr), right(nullptr) {}
};

class SchoolBST {
private:
    BSTNode* root;

    BSTNode* insert(BSTNode* node, School school) {
        if (node == nullptr) return new BSTNode(school);

        if (school.name < node->school.name)
            node->left = insert(node->left, school);
        else if (school.name > node->school.name)
            node->right = insert(node->right, school);

        return node;
    }

    BSTNode* findMin(BSTNode* node) {
        while (node->left) node = node->left;
        return node;
    }

    BSTNode* deleteByName(BSTNode* node, std::string name) {
        if (!node) return node;

        if (name < node->school.name)
            node->left = deleteByName(node->left, name);
        else if (name > node->school.name)
            node->right = deleteByName(node->right, name);
        else {
            if (!node->left) {
                BSTNode* temp = node->right;
                delete node;
                return temp;
            } else if (!node->right) {
                BSTNode* temp = node->left;
                delete node;
                return temp;
            }

            BSTNode* temp = findMin(node->right);
            node->school = temp->school;
            node->right = deleteByName(node->right, temp->school.name);
        }
        return node;
    }

    BSTNode* findByName(BSTNode* node, std::string name) {
        if (!node || node->school.name == name) return node;
        if (name < node->school.name) return findByName(node->left, name);
        return findByName(node->right, name);
    }

    void displayInOrder(BSTNode* node) {
        if (!node) return;
        displayInOrder(node->left);
        std::cout << "Name: " << node->school.name << ", City: " << node->school.city << ", State: " << node->school.state << "\n";
        displayInOrder(node->right);
    }

public:
    SchoolBST() : root(nullptr) {}

    void insert(School school) { root = insert(root, school); }
    void deleteByName(std::string name) { root = deleteByName(root, name); }

    void findByName(std::string name) {
        BSTNode* result = findByName(root, name);
        if (result)
            std::cout << "Found: " << result->school.name << ", City: " << result->school.city << ", State: " << result->school.state << "\n";
        else
            std::cout << "School not found.\n";
    }

    void displayInOrder() { displayInOrder(root); }
};

#endif // SCHOOLBST_H
