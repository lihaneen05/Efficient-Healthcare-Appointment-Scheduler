#include <iostream>
#include <vector>
#include <unordered_map>
#include <string>
#include <limits> // For numeric_limits

using namespace std;

// Struct to represent an appointment
struct Appointment {
    string patientName;
    string disease;
    string doctorName;
    int urgency; // Higher value indicates higher urgency
    string timeSlot;
};

// Helper function to check if a string is alphabetic (without using any built-in function)
bool isAlphabetic(const string& input) {
    for (char c : input) {
        if (!((c >= 'a' && c <= 'z') || (c >= 'A' && c <= 'Z') || c == ' ')) {
            return false;
        }
    }
    return true;
}

// Function to check if input is a valid integer
bool isValidInteger(const string& str) {
    for (char c : str) {
        if (c < '0' || c > '9') {
            return false;
        }
    }
    return true;
}

// Node structure for Binary Search Tree (BST)
struct BSTNode {
    Appointment appointment;
    BSTNode* left;
    BSTNode* right;

    BSTNode(const Appointment& app) : appointment(app), left(nullptr), right(nullptr) {}
};

// Binary Search Tree to store appointments sorted by patient name
class BSTree {
private:
    BSTNode* root;

    BSTNode* insert(BSTNode* node, const Appointment& app) {
        if (node == nullptr) {
            return new BSTNode(app);
        }

        if (app.patientName < node->appointment.patientName) {
            node->left = insert(node->left, app);
        }
        else {
            node->right = insert(node->right, app);
        }

        return node;
    }

    void inorder(BSTNode* node) {
        if (node != nullptr) {
            inorder(node->left);
            cout << "Patient: " << node->appointment.patientName << ", Disease: " << node->appointment.disease
                << ", Doctor: Dr. " << node->appointment.doctorName << ", Time Slot: " << node->appointment.timeSlot
                << ", Urgency: " << node->appointment.urgency << endl;
            inorder(node->right);
        }
    }

public:
    BSTree() : root(nullptr) {}

    void insert(const Appointment& app) {
        root = insert(root, app);
    }

    void displayAppointments() {
        inorder(root);
    }
};

// Class to implement a custom priority queue
class CustomPriorityQueue {
private:
    vector<Appointment> heap;

    void heapifyUp(int index) {
        if (index == 0) return;
        int parent = (index - 1) / 2;
        if (heap[index].urgency > heap[parent].urgency) {
            swap(heap[index], heap[parent]);
            heapifyUp(parent);
        }
    }

    void heapifyDown(int index) {
        int left = 2 * index + 1;
        int right = 2 * index + 2;
        int largest = index;

        if (left < heap.size() && heap[left].urgency > heap[largest].urgency)
            largest = left;

        if (right < heap.size() && heap[right].urgency > heap[largest].urgency)
            largest = right;

        if (largest != index) {
            swap(heap[index], heap[largest]);
            heapifyDown(largest);
        }
    }

public:
    void push(const Appointment& appointment) {
        heap.push_back(appointment);
        heapifyUp(heap.size() - 1);
    }

    void pop() {
        if (heap.empty()) return;
        heap[0] = heap.back();
        heap.pop_back();
        heapifyDown(0);
    }

    Appointment top() const {
        if (!heap.empty()) return heap[0];
        return { "", "", "", -1, "" };
    }

    bool empty() const {
        return heap.empty();
    }
};

// Node structure for Linked List
struct ListNode {
    Appointment appointment;
    ListNode* next;

    ListNode(const Appointment& app) : appointment(app), next(nullptr) {}
};

// Linked List to store appointments for each doctor
class LinkedList {
private:
    ListNode* head;

public:
    LinkedList() : head(nullptr) {}

    void addAppointment(const Appointment& app) {
        ListNode* newNode = new ListNode(app);
        if (!head) {
            head = newNode;
        }
        else {
            ListNode* temp = head;
            while (temp->next) {
                temp = temp->next;
            }
            temp->next = newNode;
        }
    }

    void displayAppointments() {
        ListNode* temp = head;
        while (temp) {
            cout << "Patient: " << temp->appointment.patientName << ", Disease: " << temp->appointment.disease
                << ", Doctor: Dr. " << temp->appointment.doctorName << ", Time Slot: " << temp->appointment.timeSlot
                << ", Urgency: " << temp->appointment.urgency << endl;
            temp = temp->next;
        }
    }
};

// Custom stack class for managing the appointment history (undo functionality)
class CustomStack {
private:
    struct StackNode {
        Appointment appointment;
        StackNode* next;

        StackNode(const Appointment& app) : appointment(app), next(nullptr) {}
    };

    StackNode* topNode;

public:
    CustomStack() : topNode(nullptr) {}

    void push(const Appointment& app) {
        StackNode* newNode = new StackNode(app);
        newNode->next = topNode;
        topNode = newNode;
    }

    void pop() {
        if (topNode == nullptr) return;

        StackNode* temp = topNode;
        topNode = topNode->next;
        delete temp;
    }

    Appointment top() const {
        if (topNode != nullptr) {
            return topNode->appointment;
        }
        return { "", "", "", -1, "" };
    }

    bool empty() const {
        return topNode == nullptr;
    }
};

// Class to manage appointments
class AppointmentScheduler {
private:
    CustomPriorityQueue appointmentQueue;
    BSTree appointmentTree;
    unordered_map<string, vector<string>> doctorAvailability; // Doctor -> Time slots
    unordered_map<string, LinkedList> doctorAppointments; // Doctor -> Linked list of appointments
    CustomStack appointmentHistory; // Stack for undo functionality

public:
    // Add a doctor and their available time slots
    void addDoctor(const string& doctorName, const vector<string>& timeSlots) {
        doctorAvailability[doctorName] = timeSlots;
    }

    // Schedule an appointment
    void scheduleAppointment() {
        string patientName, disease, doctorName;
        int urgency;

        // Get valid patient name
        do {
            cout << "Enter patient name (alphabets only): ";
            getline(cin, patientName);
            if (!isAlphabetic(patientName)) {
                cout << "Invalid input. Please enter alphabets only.\n";
            }
        } while (!isAlphabetic(patientName));

        // Get valid disease
        do {
            cout << "Enter disease (alphabets only): ";
            getline(cin, disease);
            if (!isAlphabetic(disease)) {
                cout << "Invalid input. Please enter alphabets only.\n";
            }
        } while (!isAlphabetic(disease));

        // Get valid doctor name
        do {
            cout << "Enter doctor name (alphabets only): ";
            getline(cin, doctorName);
            if (!isAlphabetic(doctorName)) {
                cout << "Invalid input. Please enter alphabets only.\n";
            }
        } while (!isAlphabetic(doctorName));

        // Get valid urgency
        do {
            cout << "Enter urgency (1-10, higher is more urgent): ";
            string urgencyStr;
            getline(cin, urgencyStr);
            if (isValidInteger(urgencyStr)) {
                urgency = stoi(urgencyStr);
                if (urgency < 1 || urgency > 10) {
                    cout << "Invalid urgency. Please enter a value between 1 and 10.\n";
                }
            }
            else {
                cout << "Invalid input. Please enter a number for urgency.\n";
            }
        } while (urgency < 1 || urgency > 10);

        if (doctorAvailability.find(doctorName) == doctorAvailability.end() || doctorAvailability[doctorName].empty()) {
            cout << "No available slots for Dr. " << doctorName << "." << endl;
            return;
        }

        // Assign the first available time slot
        string timeSlot = doctorAvailability[doctorName].front();
        doctorAvailability[doctorName].erase(doctorAvailability[doctorName].begin());

        // Create and add appointment
        Appointment app = { patientName, disease, doctorName, urgency, timeSlot };
        appointmentQueue.push(app);
        appointmentTree.insert(app);
        doctorAppointments[doctorName].addAppointment(app);
        appointmentHistory.push(app);

        cout << "Appointment scheduled successfully!\n"
            << "Patient: " << patientName << "\n"
            << "Disease: " << disease << "\n"
            << "Doctor: Dr. " << doctorName << "\n"
            << "Time Slot: " << timeSlot << "\n"
            << "Urgency: " << urgency << "\n";
    }

    // Process the next appointment
    void processNextAppointment() {
        if (appointmentQueue.empty()) {
            cout << "No appointments to process." << endl;
            return;
        }

        Appointment next = appointmentQueue.top();
        appointmentQueue.pop();

        cout << "Processing appointment:\n"
            << "Patient: " << next.patientName << "\n"
            << "Disease: " << next.disease << "\n"
            << "Doctor: Dr. " << next.doctorName << "\n"
            << "Time Slot: " << next.timeSlot << "\n"
            << "Urgency: " << next.urgency << "\n";
    }

    // Search for appointments by patient name
    void searchAppointment(const string& patientName) {
        cout << "\nAppointments for " << patientName << ":\n";
        appointmentTree.displayAppointments();
    }

    // Display doctor availability
    void displayDoctorAvailability() {
        cout << "\nDoctor Availability:\n";
        for (const auto& entry : doctorAvailability) {
            const auto& doctor = entry.first;
            const auto& slots = entry.second;

            cout << "Dr. " << doctor << ": ";
            for (const auto& slot : slots) {
                cout << slot << " ";
            }
            cout << endl;
        }
    }

    // Undo the last scheduled appointment
    void undoLastAppointment() {
        if (!appointmentHistory.empty()) {
            Appointment lastAppointment = appointmentHistory.top();
            appointmentHistory.pop();
            cout << "Undoing last appointment:\n"
                << "Patient: " << lastAppointment.patientName << "\n"
                << "Disease: " << lastAppointment.disease << "\n"
                << "Doctor: Dr. " << lastAppointment.doctorName << "\n"
                << "Time Slot: " << lastAppointment.timeSlot << "\n"
                << "Urgency: " << lastAppointment.urgency << "\n";

            // Remove from the queue, BST, and doctor appointments
            CustomPriorityQueue tempQueue;
            while (!appointmentQueue.empty()) {
                Appointment current = appointmentQueue.top();
                appointmentQueue.pop();
                if (current.patientName != lastAppointment.patientName) {
                    tempQueue.push(current);
                }
            }
            appointmentQueue = tempQueue;
        }
    }
};

int main() {
    AppointmentScheduler scheduler;

    // Add doctors and their time slots
    scheduler.addDoctor("Sara", { "9:00 AM", "10:00 AM", "11:00 AM" });
    scheduler.addDoctor("Haneen", { "2:00 PM", "3:00 PM" });

    int choice;
    do {
        cout << "\n1. Schedule an Appointment\n";
        cout << "2. Process Next Appointment\n";
        cout << "3. Search Appointment by Patient Name\n";
        cout << "4. Display Doctor Availability\n";
        cout << "5. Undo Last Appointment\n";
        cout << "6. Exit\n";
        cout << "Enter your choice: ";
        string choiceStr;
        getline(cin, choiceStr);

        if (isValidInteger(choiceStr)) {
            choice = stoi(choiceStr);
        }
        else {
            cout << "Invalid input. Please enter a number.\n";
            continue;
        }

        switch (choice) {
        case 1:
            scheduler.scheduleAppointment();
            break;
        case 2:
            scheduler.processNextAppointment();
            break;
        case 3: {
            string patientName;
            cout << "Enter patient name: ";
            getline(cin, patientName);
            scheduler.searchAppointment(patientName);
            break;
        }
        case 4:
            scheduler.displayDoctorAvailability();
            break;
        case 5:
            scheduler.undoLastAppointment();
            break;
        case 6:
            cout << "Exiting program.\n";
            break;
        default:
            cout << "Invalid choice. Try again.\n";
        }
    } while (choice != 6);

    return 0;
}
