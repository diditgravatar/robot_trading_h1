# Robot Trading Otomatis Untuk XAUUSD(H1)
Ini adalah robot trading otomatis (Expert Advisor) di **MetaTrader 4 (MT4) dan MetaTrader 5 (MT5)**, kita akan menggunakan bahasa **MQL4/MQL5**. Robot ini akan:  

1. **Entry Buy** saat MA5 melintasi MA9 dari bawah ke atas.  
2. **Exit Buy** saat MA5 melintasi MA9 dari atas ke bawah.  
3. **Entry Sell** saat MA5 melintasi MA9 dari atas ke bawah.  
4. **Exit Sell** saat MA5 melintasi MA9 dari bawah ke atas.  

Kode berikut dibuat untuk berjalan otomatis di **MT4 dan MT5**.  

---

## **1. Kode Expert Advisor (EA) untuk MT4 dan MT5**
### **Langkah-langkah:**
1. **Buka MetaTrader (MT4/MT5)**  
2. **Klik `File > Open Data Folder` > MQL4 (untuk MT4) atau MQL5 (untuk MT5) > Experts**  
3. **Buat file baru**:  
   - Untuk **MT4** → `XAUUSD_MA_Strategy.mq4`  
   - Untuk **MT5** → `XAUUSD_MA_Strategy.mq5`  
4. **Copy & Paste kode berikut** sesuai platform.

---

### **2. Kode Expert Advisor (EA)**
#### **Kode untuk MT4 (`XAUUSD_MA_Strategy.mq4`)**
```mql4
//+------------------------------------------------------------------+
//| Expert Advisor: MA Crossover for XAUUSD (MT4)                   |
//+------------------------------------------------------------------+
#property strict

input int ma_fast_period = 5;  // Periode Moving Average cepat
input int ma_slow_period = 9;  // Periode Moving Average lambat
input double lot_size = 0.1;   // Lot trading

//+------------------------------------------------------------------+
//| Fungsi untuk mendapatkan nilai Moving Average                   |
//+------------------------------------------------------------------+
double GetMA(int period, int shift) {
    return iMA(Symbol(), PERIOD_H1, period, 0, MODE_SMA, PRICE_CLOSE, shift);
}

//+------------------------------------------------------------------+
//| Fungsi untuk eksekusi order                                     |
//+------------------------------------------------------------------+
void OpenOrder(string type) {
    double lot = lot_size;
    double price = Ask;
    double stopLoss = 0, takeProfit = 0;

    if (type == "BUY") {
        price = Ask;
        stopLoss = price - 100 * Point;
        takeProfit = price + 200 * Point;
        OrderSend(Symbol(), OP_BUY, lot, price, 10, stopLoss, takeProfit, "Buy Order", 0, 0, clrGreen);
    } 
    else if (type == "SELL") {
        price = Bid;
        stopLoss = price + 100 * Point;
        takeProfit = price - 200 * Point;
        OrderSend(Symbol(), OP_SELL, lot, price, 10, stopLoss, takeProfit, "Sell Order", 0, 0, clrRed);
    }
}

//+------------------------------------------------------------------+
//| Fungsi untuk menutup semua order aktif                          |
//+------------------------------------------------------------------+
void CloseAllOrders(string type) {
    for (int i = OrdersTotal() - 1; i >= 0; i--) {
        if (OrderSelect(i, SELECT_BY_POS, MODE_TRADES)) {
            if ((type == "BUY" && OrderType() == OP_BUY) ||
                (type == "SELL" && OrderType() == OP_SELL)) {
                OrderClose(OrderTicket(), OrderLots(), OrderClosePrice(), 10);
            }
        }
    }
}

//+------------------------------------------------------------------+
//| Fungsi utama EA                                                 |
//+------------------------------------------------------------------+
void OnTick() {
    double maFastCurrent = GetMA(ma_fast_period, 0);
    double maFastPrevious = GetMA(ma_fast_period, 1);
    double maSlowCurrent = GetMA(ma_slow_period, 0);
    double maSlowPrevious = GetMA(ma_slow_period, 1);

    // Entry Buy
    if (maFastPrevious < maSlowPrevious && maFastCurrent > maSlowCurrent) {
        CloseAllOrders("SELL");
        OpenOrder("BUY");
    }
    
    // Exit Buy
    if (maFastPrevious > maSlowPrevious && maFastCurrent < maSlowCurrent) {
        CloseAllOrders("BUY");
    }

    // Entry Sell
    if (maFastPrevious > maSlowPrevious && maFastCurrent < maSlowCurrent) {
        CloseAllOrders("BUY");
        OpenOrder("SELL");
    }

    // Exit Sell
    if (maFastPrevious < maSlowPrevious && maFastCurrent > maSlowCurrent) {
        CloseAllOrders("SELL");
    }
}
```

---

#### **Kode untuk MT5 (`XAUUSD_MA_Strategy.mq5`)**
```mql5
//+------------------------------------------------------------------+
//| Expert Advisor: MA Crossover for XAUUSD (MT5)                   |
//+------------------------------------------------------------------+
#property strict

input int ma_fast_period = 5;  // Periode MA cepat
input int ma_slow_period = 9;  // Periode MA lambat
input double lot_size = 0.1;   // Lot trading

//+------------------------------------------------------------------+
//| Fungsi untuk mendapatkan nilai Moving Average                   |
//+------------------------------------------------------------------+
double GetMA(int period, int shift) {
    return iMA(Symbol(), PERIOD_H1, period, 0, MODE_SMA, PRICE_CLOSE, shift);
}

//+------------------------------------------------------------------+
//| Fungsi untuk eksekusi order                                     |
//+------------------------------------------------------------------+
void OpenOrder(string type) {
    MqlTradeRequest request;
    MqlTradeResult result;
    
    ZeroMemory(request);
    request.action = TRADE_ACTION_DEAL;
    request.type_filling = ORDER_FILLING_IOC;
    request.volume = lot_size;
    
    if (type == "BUY") {
        request.type = ORDER_TYPE_BUY;
        request.price = SymbolInfoDouble(Symbol(), SYMBOL_ASK);
    } 
    else if (type == "SELL") {
        request.type = ORDER_TYPE_SELL;
        request.price = SymbolInfoDouble(Symbol(), SYMBOL_BID);
    }

    OrderSend(request, result);
}

//+------------------------------------------------------------------+
//| Fungsi untuk menutup semua order aktif                          |
//+------------------------------------------------------------------+
void CloseAllOrders(string type) {
    for (int i = OrdersTotal() - 1; i >= 0; i--) {
        if (OrderSelect(i, SELECT_BY_POS, MODE_TRADES)) {
            if ((type == "BUY" && OrderType() == ORDER_TYPE_BUY) ||
                (type == "SELL" && OrderType() == ORDER_TYPE_SELL)) {
                MqlTradeRequest request;
                MqlTradeResult result;
                ZeroMemory(request);
                
                request.action = TRADE_ACTION_CLOSE_BY;
                request.order = OrderTicket();
                OrderSend(request, result);
            }
        }
    }
}

//+------------------------------------------------------------------+
//| Fungsi utama EA                                                 |
//+------------------------------------------------------------------+
void OnTick() {
    double maFastCurrent = GetMA(ma_fast_period, 0);
    double maFastPrevious = GetMA(ma_fast_period, 1);
    double maSlowCurrent = GetMA(ma_slow_period, 0);
    double maSlowPrevious = GetMA(ma_slow_period, 1);

    // Entry Buy
    if (maFastPrevious < maSlowPrevious && maFastCurrent > maSlowCurrent) {
        CloseAllOrders("SELL");
        OpenOrder("BUY");
    }
    
    // Exit Buy
    if (maFastPrevious > maSlowPrevious && maFastCurrent < maSlowCurrent) {
        CloseAllOrders("BUY");
    }

    // Entry Sell
    if (maFastPrevious > maSlowPrevious && maFastCurrent < maSlowCurrent) {
        CloseAllOrders("BUY");
        OpenOrder("SELL");
    }

    // Exit Sell
    if (maFastPrevious < maSlowPrevious && maFastCurrent > maSlowCurrent) {
        CloseAllOrders("SELL");
    }
}
```

---

## **Cara Menggunakan Robot Trading MA Crossover untuk XAU/USD (MT4 & MT5)**  

### **1. Persiapan Awal**  
Pastikan Anda sudah memiliki:  
✅ **MetaTrader 4 atau MetaTrader 5** terinstal.  
✅ **Akun trading** di broker yang mendukung MT4/MT5.  
✅ **Koneksi internet stabil** untuk eksekusi otomatis.  

---

### **2. Instalasi Robot Trading (EA)**
1. **Buka MetaTrader 4 atau MetaTrader 5.**  
2. **Masuk ke Folder Expert Advisors:**  
   - Klik **`File > Open Data Folder`**  
   - Masuk ke folder **`MQL4/Experts`** (untuk MT4) atau **`MQL5/Experts`** (untuk MT5).  
3. **Copy file EA sesuai platform:**  
   - Untuk **MT4** → `XAUUSD_MA_Strategy.mq4`  
   - Untuk **MT5** → `XAUUSD_MA_Strategy.mq5`  
4. **Restart MetaTrader** agar EA terdeteksi.  

---

### **3. Menjalankan EA di Chart XAU/USD**
1. **Buka Chart XAU/USD dengan Timeframe H1.**  
2. **Aktifkan EA di Navigator:**  
   - Di panel kiri, buka **`Navigator > Expert Advisors`**.  
   - Klik kanan **`XAUUSD_MA_Strategy`** lalu pilih **"Attach to Chart"**.  
3. **Aktifkan Trading Otomatis:**  
   - Klik tombol **`Algo Trading`** (MT5) atau **`AutoTrading`** (MT4) di toolbar.  
   - Pastikan ikon berubah menjadi hijau.  

---

### **4. Pengaturan Parameter EA**
Setelah EA di-attach ke chart, akan muncul jendela pengaturan.  
Anda bisa menyesuaikan parameter seperti:  
- **MA Fast Period** → `5` (default)  
- **MA Slow Period** → `9` (default)  
- **Lot Size** → `0.1` (default, bisa diubah sesuai manajemen risiko)  

Klik **OK** untuk menjalankan EA.  

---

### **5. Cara Kerja Robot Trading**
📌 **Entry Buy:**  
- Jika **MA5 melintasi MA9 dari bawah ke atas**, EA akan membuka posisi **BUY**.  

📌 **Exit Buy:**  
- Jika **MA5 melintasi MA9 dari atas ke bawah**, EA akan **menutup posisi BUY**.  

📌 **Entry Sell:**  
- Jika **MA5 melintasi MA9 dari atas ke bawah**, EA akan membuka posisi **SELL**.  

📌 **Exit Sell:**  
- Jika **MA5 melintasi MA9 dari bawah ke atas**, EA akan **menutup posisi SELL**.  

---

### **6. Monitoring dan Optimasi**
- **Pantau trade di tab `Trade`** di MT4/MT5.  
- **Gunakan fitur `Strategy Tester`** untuk backtesting di data historis.  
- **Lakukan optimasi parameter MA** untuk menyesuaikan dengan kondisi market.  

---

### **7. Troubleshooting**
❌ **EA tidak berjalan?**  
✔ Pastikan tombol **`Algo Trading`** (MT5) atau **`AutoTrading`** (MT4) aktif.  
✔ Cek di tab `Experts` atau `Journal` untuk error log.  

❌ **Tidak ada order terbuka?**  
✔ Pastikan spread broker tidak terlalu besar.  
✔ Coba ubah **lot size** atau **periode MA** di pengaturan EA.  

---

### **8. Kesimpulan**
Dengan mengikuti langkah-langkah di atas, **robot trading otomatis akan bekerja di XAU/USD (gold) pada timeframe H1.**  
Anda bisa mengoptimalkan strategi ini dengan **pengujian backtest** dan **menyesuaikan setting** sesuai preferensi trading Anda.  

🚀 **Selamat trading dengan EA XAU/USD MA Crossover!**

🚀 **Robot ini ditulis oleh Didit Farafat**
