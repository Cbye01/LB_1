#include <windows.h>
#include <iostream>

int main() {
    STARTUPINFO si;
    PROCESS_INFORMATION pi;
    ZeroMemory(&si, sizeof(si));
    si.cb = sizeof(si);
    ZeroMemory(&pi, sizeof(pi));

    LPCSTR applicationName = "C:\\Windows\\System32\\notepad.exe";
    
    if (!CreateProcess(applicationName, NULL, NULL, NULL, FALSE, 0, NULL, NULL, &si, &pi)) {
        std::cerr << "Не вдалося створити процес. Код помилки: " << GetLastError() << std::endl;
        return 1;
    }
    std::cout << "Процес запущено. PID: " << pi.dwProcessId << std::endl;

    DWORD waitResult = WaitForSingleObject(pi.hProcess, 10000);  // Чекаємо 10 секунд

    if (waitResult == WAIT_TIMEOUT) {
        std::cout << "Час очікування вичерпано. Завершення процесу..." << std::endl;
        TerminateProcess(pi.hProcess, 1);  // Завершуємо процес з кодом помилки
    } else if (waitResult == WAIT_OBJECT_0) {
        DWORD exitCode;
        if (GetExitCodeProcess(pi.hProcess, &exitCode)) {
            std::cout << "Процес завершено. Код завершення: " << exitCode << std::endl;
        } else {
            std::cerr << "Не вдалося отримати код завершення процесу. Код помилки: " << GetLastError() << std::endl;
        }
    } else {
        std::cerr << "Помилка очікування. Код помилки: " << GetLastError() << std::endl;
    }

    CloseHandle(pi.hProcess);
    CloseHandle(pi.hThread);

    return 0;
}
