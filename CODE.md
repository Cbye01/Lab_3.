
#include <iostream>
#include <vector>
#include <algorithm>
#include <queue>
#include <random>
#include <ctime>
#include <iostream>
#include <vector>
#include <algorithm>

struct Process {
    int id, arrival_time, burst_time, start_time, completion_time, waiting_time;
};

bool compareByBurstTime(const Process &a, const Process &b) {
    return a.burst_time < b.burst_time;
}

void sjf(std::vector<Process> &processes) {
    int current_time = 0;
    std::sort(processes.begin(), processes.end(), [](Process a, Process b) {
        return a.arrival_time < b.arrival_time;
    });

    while (!processes.empty()) {
        std::vector<Process> ready_queue;
        for (const auto &process : processes) {
            if (process.arrival_time <= current_time) {
                ready_queue.push_back(process);
            }
        }
        if (!ready_queue.empty()) {
            std::sort(ready_queue.begin(), ready_queue.end(), compareByBurstTime);
            Process current = ready_queue.front();

            
            current.start_time = current_time;
            current.completion_time = current_time + current.burst_time;
            current.waiting_time = current.start_time - current.arrival_time;
            current_time = current.completion_time;

           
            auto it = std::remove_if(processes.begin(), processes.end(), [&](Process p) {
                return p.id == current.id;
            });
            processes.erase(it, processes.end());

            std::cout << "Process " << current.id
                      << " executed from " << current.start_time
                      << " to " << current.completion_time << "\n";
        } else {
            current_time++;
        }
    }
}

struct Process {
    int id;
    int arrivalTime;
    int burstTime;
    int remainingTime;
    int priority;
    int startTime;
    int finishTime;
    int waitingTime;
};

std::vector<Process> generateProcesses(int n) {
    std::vector<Process> processes;
    std::srand(std::time(0));
    for (int i = 0; i < n; i++) {
        Process p;
        p.id = i + 1;
        p.arrivalTime = std::rand() % 10;
        p.burstTime = (std::rand() % 10) + 1;
        p.remainingTime = p.burstTime;
        p.priority = std::rand() % 5 + 1;  
        processes.push_back(p);
    }
    return processes;
}

void roundRobin(std::vector<Process> processes, int quantum) {
    int time = 0;
    std::queue<Process*> processQueue;
    for (auto &p : processes) {
        if (p.arrivalTime <= time) {
            processQueue.push(&p);
        }
    }

    std::cout << "Round Robin Execution:\n";
    while (!processQueue.empty()) {
        Process* p = processQueue.front();
        processQueue.pop();

        if (p->remainingTime > quantum) {
            time += quantum;
            p->remainingTime -= quantum;
            std::cout << "Process " << p->id << ": Remaining time = " << p->remainingTime << std::endl;

            for (auto &next : processes) {
                if (next.arrivalTime <= time && next.remainingTime > 0 && &next != p) {
                    processQueue.push(&next);
                }
            }
            processQueue.push(p); 
        } else {
            time += p->remainingTime;
            p->finishTime = time;
            p->remainingTime = 0;
            std::cout << "Process " << p->id << " finished at time " << p->finishTime << std::endl;
        }
    }
}

void fcfs(std::vector<Process> processes) {
    std::sort(processes.begin(), processes.end(), [](Process a, Process b) {
        return a.arrivalTime < b.arrivalTime;
    });

    int time = 0;
    std::cout << "FCFS Execution:\n";
    for (auto &p : processes) {
        if (time < p.arrivalTime) time = p.arrivalTime;
        p.startTime = time;
        time += p.burstTime;
        p.finishTime = time;
        p.waitingTime = p.startTime - p.arrivalTime;
        std::cout << "Process " << p.id << ": Start time = " << p.startTime << ", Finish time = " << p.finishTime << ", Waiting time = " << p.waitingTime << std::endl;
    }
}

void priorityScheduling(std::vector<Process> processes) {
    std::sort(processes.begin(), processes.end(), [](Process a, Process b) {
        return a.priority < b.priority;
    });

    int time = 0;
    std::cout << "Priority Scheduling Execution:\n";
    for (auto &p : processes) {
        if (time < p.arrivalTime) time = p.arrivalTime;
        p.startTime = time;
        time += p.burstTime;
        p.finishTime = time;
        p.waitingTime = p.startTime - p.arrivalTime;
        std::cout << "Process " << p.id << ": Priority = " << p.priority << ", Finish time = " << p.finishTime << std::endl;
    }
}

void calculateAverageTimes(const std::vector<Process>& processes, const std::string& algorithmName) {
    int totalWaitingTime = 0;
    int totalTurnaroundTime = 0;
    for (const auto& p : processes) {
        int turnaroundTime = p.finishTime - p.arrivalTime;
        totalWaitingTime += p.waitingTime;
        totalTurnaroundTime += turnaroundTime;
    }
    double avgWaitingTime = static_cast<double>(totalWaitingTime) / processes.size();
    double avgTurnaroundTime = static_cast<double>(totalTurnaroundTime) / processes.size();
    std::cout << algorithmName << " Average Waiting Time: " << avgWaitingTime << std::endl;
    std::cout << algorithmName << " Average Turnaround Time: " << avgTurnaroundTime << std::endl;
}

int main() {
    int numProcesses = 5;
    int quantum = 3;
    std::vector<Process> processes = generateProcesses(numProcesses);

    
    std::vector<Process> rrProcesses = processes;
    roundRobin(rrProcesses, quantum);
    calculateAverageTimes(rrProcesses, "Round Robin");

    
    std::vector<Process> fcfsProcesses = processes;
    fcfs(fcfsProcesses);
    calculateAverageTimes(fcfsProcesses, "FCFS");

   
    std::vector<Process> priorityProcesses = processes;
    priorityScheduling(priorityProcesses);
    calculateAverageTimes(priorityProcesses, "Priority Scheduling");

  
