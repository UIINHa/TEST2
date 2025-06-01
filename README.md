#include <windows.h>
#include <tlhelp32.h>
#include <iostream>
#include <iomanip>

// 모듈 베이스 주소 가져오기 함수
uintptr_t GetModuleBaseAddress(DWORD pid, const wchar_t* modName) {
    uintptr_t baseAddress = 0;
    HANDLE hSnap = CreateToolhelp32Snapshot(TH32CS_SNAPMODULE, pid);
    if (hSnap != INVALID_HANDLE_VALUE) {
        MODULEENTRY32 modEntry;
        modEntry.dwSize = sizeof(modEntry);
        if (Module32First(hSnap, &modEntry)) {
            do {
                if (!_wcsicmp(modEntry.szModule, modName)) {
                    baseAddress = (uintptr_t)modEntry.modBaseAddr;
                    break;
                }
            } while (Module32Next(hSnap, &modEntry));
        }
    }
    CloseHandle(hSnap);
    return baseAddress;
}

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

    // 고정된 모듈 베이스 주소 + 오프셋 사용
    uintptr_t moduleBase = GetModuleBaseAddress(pid, L"ac_client.exe");
    uintptr_t playerPtrAddr = moduleBase + 0x10F4F4; // 고정 오프셋

    DWORD playerBase = 0;
    BOOL success = ReadProcessMemory(hProc, (LPCVOID)playerPtrAddr, &playerBase, sizeof(DWORD), NULL);
    if (!success || playerBase == 0) {
        std::cout << "playerBase 읽기 실패 또는 값이 0입니다.\n";
        CloseHandle(hProc);
        return 1;
    }

    float x = 0, y = 0, z = 0;
    if (!ReadProcessMemory(hProc, (LPCVOID)(playerBase + 0x04), &x, sizeof(x), NULL)) std::cout << "X 읽기 실패\n";
    if (!ReadProcessMemory(hProc, (LPCVOID)(playerBase + 0x08), &y, sizeof(y), NULL)) std::cout << "Y 읽기 실패\n";
    if (!ReadProcessMemory(hProc, (LPCVOID)(playerBase + 0x0C), &z, sizeof(z), NULL)) std::cout << "Z 읽기 실패\n";

    std::cout << std::fixed << std::setprecision(3);
    std::cout << "내 위치 (좌표):\n";
    std::cout << "X = " << x << "\n";
    std::cout << "Y = " << y << "\n";
    std::cout << "Z = " << z
