# VerveBridge
//Bank management system
#include <iostream>
#include <fstream>
#include <cstring>  
#include <vector>
#include <cstdlib>  
#include <sstream>  
using namespace std;

// Class definition for BankAccount
class BankAccount {
    char name[100], address[100], accountType; // Basic account information
    int balance; // Current balance of the account
    char password[50]; // Account password
    vector<string> transactions; // To track transactions

    // Private methods for account management
    void loadAccountData(); // Load account data from file
    void saveAccountData(); // Save account data to file
    void addTransaction(const string& transaction); // Add a transaction to the record
    void deleteAccountFiles(); // Delete account data files

public:
    // Constructor initializes balance to 0
    BankAccount() : balance(0) {}

    // Public methods for interacting with the account
    void createAccount(); // Create a new account
    bool authenticate(); // Authenticate user with password
    void depositMoney(); // Deposit money into the account
    void withdrawMoney(); // Withdraw money from the account
    void displayAccount(); // Display account details
    void closeAccount(); // Close the account and delete files

    // Getters and setters for account attributes
    const char* getName() const { return name; }
    const char* getPassword() const { return password; }
    void setName(const char* n) { strcpy(name, n); }
    void setPassword(const char* p) { strcpy(password, p); }
    void setAddress(const char* a) { strcpy(address, a); }
    void setAccountType(char at) { accountType = at; }
    void setBalance(int bal) { balance = bal; }
    int getBalance() const { return balance; }
    void setTransactions(const vector<string>& trans) { transactions = trans; }
};

// Load account data from file
void BankAccount::loadAccountData() {
    ifstream file(name); // Open file named after the account holder's name
    if (file) {
        file.getline(name, 100); // Read name
        file.getline(address, 100); // Read address
        file >> accountType >> balance; // Read account type and balance
        file.ignore(); // Ignore the newline after balance
        file.getline(password, 50); // Read password
        file.close();
    } else {
        // Initialize with default values if file does not exist
        strcpy(name, "");
        strcpy(address, "");
        accountType = 's';  // Default to saving account
        balance = 0;
        strcpy(password, "");
    }
}

// Save account data to file
void BankAccount::saveAccountData() {
    ofstream file(name); // Open file named after the account holder's name
    if (file) {
        file << name << endl;
        file << address << endl;
        file << accountType << endl;
        file << balance << endl;
        file << password << endl;
        file.close();
    } else {
        cerr << "Error: Unable to save account information for " << name << endl;
    }
}

// Add a transaction to the transaction history
void BankAccount::addTransaction(const string& transaction) {
    transactions.push_back(transaction); // Store transaction in vector
    ofstream transFile(string(name) + "_transactions.txt", ios::app); // Open transaction file in append mode
    if (transFile) {
        transFile << transaction << endl;
        transFile.close();
    } else {
        cerr << "Error: Unable to save transaction for " << name << endl;
    }
}

// Delete account and transaction files
void BankAccount::deleteAccountFiles() {
    if (remove(name) != 0) { // Remove account data file
        cerr << "Error deleting account file for " << name << endl;
    }
    if (remove((string(name) + "_transactions.txt").c_str()) != 0) { // Remove transaction file
        cerr << "Error deleting transactions file for " << name << endl;
    }
}

// Create a new account
void BankAccount::createAccount() {
    cout << "Enter your full name: ";
    cin.ignore(); // Ignore newline from previous input
    cin.getline(name, 100); // Read account holder's name

    cout << "Enter your address: ";
    cin.getline(address, 100); // Read address

    cout << "What type of account do you want to open (s for saving, c for current): ";
    cin >> accountType; // Read account type

    cout << "Set your account password: ";
    cin >> password; // Set account password

    cout << "Enter initial deposit amount: ";
    cin >> balance; // Set initial deposit amount

    addTransaction("Account opened with initial deposit: " + to_string(balance)); // Record the initial deposit
    saveAccountData(); // Save account details to file

    cout << "Your account is created successfully!" << endl;
}

// Authenticate user by checking password
bool BankAccount::authenticate() {
    char enteredPassword[50];
    cout << "Enter your account password to authenticate: ";
    cin >> enteredPassword; // Read entered password

    if (strcmp(password, enteredPassword) == 0) {
        return true; // Password matches
    } else {
        cout << "Authentication failed. Incorrect password." << endl;
        return false; // Password does not match
    }
}

// Deposit money into the account
void BankAccount::depositMoney() {
    int depositAmount;
    cout << "Enter the amount you want to deposit: ";
    cin >> depositAmount; // Read deposit amount

    if (depositAmount < 0) {
        cout << "Invalid amount. Please enter a positive value." << endl;
        return;
    }

    balance += depositAmount; // Update balance
    addTransaction("Deposited: " + to_string(depositAmount)); // Record deposit
    saveAccountData(); // Save updated details
    cout << "Your total balance is now: " << balance << endl;
}

// Withdraw money from the account
void BankAccount::withdrawMoney() {
    int withdrawAmount;
    cout << "Enter the amount you want to withdraw: ";
    cin >> withdrawAmount; // Read withdrawal amount

    if (withdrawAmount < 0) {
        cout << "Invalid amount. Please enter a positive value." << endl;
        return;
    }

    if (withdrawAmount > balance) {
        cout << "Insufficient funds. Your current balance is: " << balance << endl;
        return;
    }

    balance -= withdrawAmount; // Update balance
    addTransaction("Withdrew: " + to_string(withdrawAmount)); // Record withdrawal
    saveAccountData(); // Save updated details
    cout << "Your remaining balance is: " << balance << endl;
}

// Display account details and transaction history
void BankAccount::displayAccount() {
    cout << "Name: " << name << endl;
    cout << "Address: " << address << endl;
    cout << "Account Type: " << (accountType == 's' ? "Saving" : "Current") << endl;
    cout << "Balance: " << balance << endl;

    cout << "Transaction History:" << endl;
    ifstream transFile(string(name) + "_transactions.txt"); // Open transaction file
    if (transFile) {
        string transaction;
        while (getline(transFile, transaction)) {
            cout << transaction << endl; // Display each transaction
        }
        transFile.close();
    } else {
        cout << "No transactions found." << endl;
    }
}

// Close the account and delete files
void BankAccount::closeAccount() {
    char confirm;
    cout << "Are you sure you want to close your account? (y/n): ";
    cin >> confirm; // Read confirmation

    if (confirm == 'y' || confirm == 'Y') {
        deleteAccountFiles(); // Delete account and transaction files
        cout << "Account closed successfully." << endl;
        exit(0); // Terminate the program since the account is closed
    } else {
        cout << "Account closure canceled." << endl;
    }
}

// Main function to interact with user
int main() {
    vector<BankAccount> accounts; // List of accounts
    int choice;
    char continueChoice;
    int selectedAccountIndex = -1; // Index of selected account

    do {
        // System clear command is commented out for portability
        // system("cls");
        cout << "01) Create a new account" << endl;
        cout << "02) Select an account" << endl;
        cout << "03) Deposit money" << endl;
        cout << "04) Withdraw money" << endl;
        cout << "05) Display account" << endl;
        cout << "06) Close account" << endl;
        cout << "07) Exit" << endl;
        cout << "Please select an option: ";
        cin >> choice; // Read user choice
        switch (choice) {
            case 1: {
                BankAccount newAccount;
                newAccount.createAccount(); // Create a new account
                accounts.push_back(newAccount); // Add to list of accounts
                break;
            }
            case 2: {
                cout << "Select an account by index (0 to " << accounts.size() - 1 << "): ";
                cin >> selectedAccountIndex; // Select an account
                if (selectedAccountIndex < 0 || selectedAccountIndex >= accounts.size()) {
                    cout << "Invalid index. Please try again." << endl;
                    selectedAccountIndex = -1;
                }
                break;
            }
            case 3:
                if (selectedAccountIndex >= 0 && selectedAccountIndex < accounts.size()) {
                    if (accounts[selectedAccountIndex].authenticate()) {
                        accounts[selectedAccountIndex].depositMoney(); // Deposit money into selected account
                    }
                } else {
                    cout << "No account selected. Please select an account first." << endl;
                }
                break;
            case 4:
                if (selectedAccountIndex >= 0 && selectedAccountIndex < accounts.size()) {
                    if (accounts[selectedAccountIndex].authenticate()) {
                        accounts[selectedAccountIndex].withdrawMoney(); // Withdraw money from selected account
                    }
                } else {
                    cout << "No account selected. Please select an account first." << endl;
                }
                break;
            case 5:
                if (selectedAccountIndex >= 0 && selectedAccountIndex < accounts.size()) {
                    if (accounts[selectedAccountIndex].authenticate()) {
                        accounts[selectedAccountIndex].displayAccount(); // Display selected account details
                    }
                } else {
                    cout << "No account selected. Please select an account first." << endl;
                }
                break;
            case 6:
                if (selectedAccountIndex >= 0 && selectedAccountIndex < accounts.size()) {
                    accounts[selectedAccountIndex].closeAccount(); // Close selected account
                    accounts.erase(accounts.begin() + selectedAccountIndex); // Remove from list
                    selectedAccountIndex = -1; // Reset selected account index
                } else {
                    cout << "No account selected. Please select an account first." << endl;
                }
                break;
            case 7:
                cout << "Exiting..." << endl;
                break;
            default:
                cout << "Invalid choice, please try again." << endl;
        }
        if (choice != 7) {	
            cout << "\nDo you want to continue? (y/n): ";
            cin >> continueChoice; // Ask if user wants to continue
        } else {
            continueChoice = 'n'; // Exit the loop if user chooses to exit
        }
    } while (continueChoice == 'y' || continueChoice == 'Y'); // Repeat until user chooses to exit
    return 0;
}

