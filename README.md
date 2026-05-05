# KTV點歌管理系統

## Proposal Report

### 動機與目標
<!-- 說明為什麼想做這個專題 -->身為一名熱愛歌唱的使用者，我發現在現行的KTV點歌系統中，常存在介面不夠直覺、點歌流程繁瑣等不便，導致唱歌的興致被操作設備的負擔所干擾。因此，本專案目標是開發一套「以人為本」的簡易KTV點歌管理系統。

### 競品比較
<!-- 比較目前已經存在可取得的類似工具或應用 -->目前現存各大KTV的點歌系統通常不會讓人知道點的所有歌曲總共多長，也無法自由調換歌曲順序，只能插播到下一首。

### 預期功能
<!-- 列出預計實作的功能 -->
1.突破傳統只能「插播到第一首」的限制，使用者可指定歌曲並任意穿插至序列中的任何位置。

2.在介面上提供即時數據更新，使用者可自行設定要常駐顯示哪些資訊如：下一首歌曲資訊、目前待播放的剩餘總曲數、待播放歌曲的累積總時數、包廂剩餘可用時間、自動時數預算（比對「剩餘歌曲總時數」與「包廂剩餘時數」，若剩餘時間不足以唱完歌單，會發出提示）等。
### 使用技術
<!-- 使用的語言、框架、工具等 -->
程式語言：C++

核心標準庫 (STL)

list 或 deque ：用於實現歌曲的任意位置穿插與高效排序。

string：處理歌名、歌手等文字資訊。

chrono：計算歌曲總時數與包廂剩餘時間的精確運算。
### Prototype 預計可驗證內容
1.動態排序邏輯： 驗證使用 std::list 或 std::deque 是否能順利在播放隊列的中間位置插入歌曲，並確保播放順序即時更新。

2.精確時間運算： 透過 chrono 庫計算多首歌曲的總時長，並與設定的「包廂剩餘時間」進行比對，驗證當時間不足時，系統是否能觸發提示訊息。

---

## Prototype Report

### 目前進度
<!-- 完成了什麼 -->
1.核心資料結構建立：已完成基於 C++ STL 的播放清單開發，實現基礎的「加入」與「任意插播」功能。

2.邏輯運算實作：已整合計時功能，可自動加總隊列中所有歌曲的預估播放總時間。

3.基礎介面開發：完成初步互動介面，可顯示目前的待播放清單與剩餘時間預算。

#include <iostream>
#include <string>
#include <list>
#include <vector>
#include <fstream>
#include <iomanip>
#include <algorithm>

using namespace std;

struct Song {
    string title;
    string artist;
    int durationSec;
};

class KTVSystem {
private:
    list<Song> playlist;
    vector<Song> songBank; // 模擬 KTV 的歌曲庫
    int remainingRoomTimeSec;

public:
    KTVSystem(int roomMinutes) : remainingRoomTimeSec(roomMinutes * 60) {
        // 預載更多熱門歌曲
        songBank = {
            {"Thank U, Next", "Ariana Grande", 331}, {"泡沫", "鄧紫棋", 259},
            {"太聰明", "陳綺貞", 263}, {"大人中", "盧廣仲", 282},
            {"Manchild", "Sabrina Carpenter", 213}, {"RUDE!", "Hearts2Hearts", 200},
            {"我怕練太壯", "曾博恩", 261}, {"Abracadabra", "Lady Gaga", 223},
            {"Not that good", "黃大謙", 258}, {"ISTJ", "NCT DREAM", 202},
            {"Blue Valenine", "NMIXX", 195}, {"明室", "洪佩瑜", 259}
        };
    }

    void showSongBank() {
        cout << "\n--- 可點歌曲庫 ---" << endl;
        for (int i = 0; i < songBank.size(); ++i) {
            cout << i + 1 << ". " << songBank[i].title << " - " << songBank[i].artist << endl;
        }
    }

    void addSongFromBank(int bankIdx, int position = -1) {
        if (bankIdx < 1 || bankIdx > songBank.size()) return;
        Song selected = songBank[bankIdx - 1];

        if (position <= 0 || position > playlist.size()) {
            playlist.push_back(selected);
            cout << ">> 已加入: " << selected.title << endl;
        } else {
            auto it = playlist.begin();
            advance(it, position - 1);
            playlist.insert(it, selected);
            cout << ">> 已插播至第 " << position << " 順位: " << selected.title << endl;
        }
    }

    void moveSong(int fromIdx, int toIdx) {
        if (fromIdx < 1 || fromIdx > playlist.size() || toIdx < 1 || toIdx > playlist.size()) {
            cout << ">> 錯誤：無效的順序編號。" << endl;
            return;
        }
        auto itFrom = playlist.begin();
        advance(itFrom, fromIdx - 1);
        Song target = *itFrom;
        
        playlist.erase(itFrom);

        auto itTo = playlist.begin();
        advance(itTo, toIdx - 1);
        playlist.insert(itTo, target);
        cout << ">> 已將第 " << fromIdx << " 首移動到第 " << toIdx << " 首。" << endl;
    }

    // --- 新增功能：刪除歌曲 ---
    void removeSong(int idx) {
        if (idx < 1 || idx > playlist.size()) {
            cout << ">> 錯誤：找不到該歌曲編號。" << endl;
            return;
        }
        auto it = playlist.begin();
        advance(it, idx - 1);
        string deletedTitle = it->title;
        playlist.erase(it);
        cout << ">> 已成功刪除歌曲: " << deletedTitle << endl;
    }

    void displayStatus() {
        int totalTime = 0;
        for (const auto& s : playlist) totalTime += s.durationSec;

        cout << "\n================ [當前播放狀態] ================" << endl;
        cout << "包廂剩餘: " << remainingRoomTimeSec / 60 << "分 | 歌單總計: " << totalTime / 60 << "分" << endl;
        if (totalTime > remainingRoomTimeSec) cout << "⚠️  警告：剩餘時間不足！" << endl;
        cout << "-----------------------------------------------" << endl;

        int idx = 1;
        for (const auto& s : playlist) {
            cout << idx++ << ". [" << s.artist << "] " << s.title << endl;
        }
        if (playlist.empty()) cout << "(目前歌單空空如也)" << endl;
        cout << "===============================================\n" << endl;
    }
};

int main() {
    KTVSystem myKTV(20);
    int choice, songIdx, pos, pos2;

    while (true) {
        myKTV.displayStatus();
        cout << "1.點歌 2.插播 3.調整順序 4.刪除歌曲 5.離開系統\n請輸入指令: ";
        cin >> choice;

        if (choice == 1) {
            myKTV.showSongBank();
            cout << "請選擇歌曲編號: ";
            cin >> songIdx;
            myKTV.addSongFromBank(songIdx);
        }
        else if (choice == 2) {
            myKTV.showSongBank();
            cout << "請選擇歌曲編號: ";
            cin >> songIdx;
            cout << "想要插到第幾順位? ";
            cin >> pos;
            myKTV.addSongFromBank(songIdx, pos);
        }
        else if (choice == 3) {
            cout << "請輸入要移動的歌曲編號: ";
            cin >> pos;
            cout << "要移到哪一個位置? ";
            cin >> pos2;
            myKTV.moveSong(pos, pos2);
        }
        // --- 處理刪除指令 ---
        else if (choice == 4) {
            cout << "請輸入要刪除的歌曲編號: ";
            cin >> pos;
            myKTV.removeSong(pos);
        }
        else break;
    }
    return 0;
}

### 遇到的困難
<!-- 遇到什麼問題、如何解決或打算如何解決 -->
1.時間即時更新：在控制台介面下，要實現「不影響操作的前提下即時刷新剩餘時間」具有挑戰。

2.記憶體管理優化：當歌曲清單過長時，需確認 list 的頻繁插入與刪除對效能的影響，確保在低配硬體上也能流暢運行。
### 下一步計畫
<!-- 接下來要做什麼 -->
1.優化使用者介面：讓使用者可自行設定要常駐顯示哪些資訊如，下一首歌曲資訊、目前待播放的剩餘總曲數、待播放歌曲的累積總時數、包廂剩餘可用時間、自動時數預算。

2.整合智慧提醒功能：強化「自動時數預算」功能，當預算不足時，主動建議使用者刪減歌曲或優先保留特定曲目。

---

## Final Report

### 專案說明
<!-- 完整描述你的專案做了什麼 -->

### 使用方式
<!-- 如何編譯、執行、使用你的程式 -->
