#include <queue>
#include <iostream>
#include <vector>
#include <algorithm>
#include <cstdlib>
#include <ctime>

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

            // Виконуємо обраний процес
            current.start_time = current_time;
            current.completion_time = current_time + current.burst_time;
            current.waiting_time = current.start_time - current.arrival_time;
            current_time = current.completion_time;

            // Видаляємо виконаний процес
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
////////////////////////////////////
struct ProcessWithPriority {
    int id, arrival_time, burst_time, priority, start_time, completion_time, waiting_time;
};

void priorityWithAging(std::vector<ProcessWithPriority> &processes, int aging_threshold) {
    int current_time = 0;
    while (!processes.empty()) {
        // Старіння: підвищуємо пріоритет процесам, які очікують понад threshold
        for (auto &process : processes) {
            if (current_time - process.arrival_time > aging_threshold) {
                process.priority--;
            }
        }

        // Сортуємо за пріоритетом та часом прибуття
        std::sort(processes.begin(), processes.end(), [](const ProcessWithPriority &a, const ProcessWithPriority &b) {
            if (a.priority == b.priority) return a.arrival_time < b.arrival_time;
            return a.priority < b.priority;
        });

        // Виконуємо процес із найвищим пріоритетом
        ProcessWithPriority current = processes.front();
        processes.erase(processes.begin());

        current.start_time = std::max(current_time, current.arrival_time);
        current.completion_time = current.start_time + current.burst_time;
        current.waiting_time = current.start_time - current.arrival_time;
        current_time = current.completion_time;

        std::cout << "Process " << current.id
                  << " executed from " << current.start_time
                  << " to " << current.completion_time << "\n";
    }
}
///////////////////////////////////////////////////////////////////////////

std::vector<Process> generateProcesses(int count) {
    std::vector<Process> processes;
    for (int i = 1; i <= count; i++) {
        processes.push_back({i, rand() % 10, rand() % 10 + 1, 0, 0, 0});
    }
    return processes;
}

std::vector<ProcessWithPriority> generateProcessesWithPriority(int count) {
    std::vector<ProcessWithPriority> processes;
    for (int i = 1; i <= count; i++) {
        processes.push_back({i, rand() % 10, rand() % 10 + 1, rand() % 5 + 1, 0, 0, 0});
    }
    return processes;
}
/////////////////////////////////////////////////////////////////////////////
int main() {
    srand(time(0));

    // Генерація процесів
    std::vector<Process> sjfProcesses = generateProcesses(5);
    std::cout << "SJF Scheduling:\n";
    sjf(sjfProcesses);

    std::vector<ProcessWithPriority> priorityProcesses = generateProcessesWithPriority(5);
    std::cout << "\nPriority Scheduling with Aging:\n";
    priorityWithAging(priorityProcesses, 3);

    return 0;
}
  
