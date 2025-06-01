#include <windows.h>
#include <iostream>
#include <iomanip>

int main() {
    HWND hwnd = FindWindowA(NULL, "AssaultCube");
    if (!hwnd) {
        std::cout << "AssaultCube 실행 중인지 확인하세요.\n";
        return 1;
    }

    DWORD pid = 0;
    GetWindowThreadProcessId(hwnd, &pid);

    HANDLE hProc = OpenProcess(PROCESS_VM_READ | PROCESS_QUERY_INFORMATION, FALSE, pid);
    if (!hProc) {
        std::cout << "프로세스 열기 실패\n";
        return 1;
    }

    DWORD playerBase = 0;
    uintptr_t pointerAddr = 0x00F2A254; // Cheat Engine에서 찾은 포인터

    BOOL success = ReadProcessMemory(hProc, (LPCVOID)pointerAddr, &playerBase, sizeof(DWORD), NULL);
    if (!success || playerBase == 0) {
        std::cout << "playerBase 읽기 실패 또는 값이 0입니다.\n";
        CloseHandle(hProc);
        return 1;
    }

    float x, y, z;
    ReadProcessMemory(hProc, (LPCVOID)(playerBase + 0x04), &x, sizeof(x), NULL);
    ReadProcessMemory(hProc, (LPCVOID)(playerBase + 0x08), &y, sizeof(y), NULL);
    ReadProcessMemory(hProc, (LPCVOID)(playerBase + 0x0C), &z, sizeof(z), NULL);

    std::cout << std::fixed << std::setprecision(3);
    std::cout << "내 위치 (좌표):\n";
    std::cout << "X = " << x << "\n";
    std::cout << "Y = " << y << "\n";
    std::cout << "Z = " << z << "\n";

    CloseHandle(hProc);
    return 0;
}
