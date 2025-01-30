# Robot Trading XAUUSD Untuk Timeframe H1
Untuk membuat robot trading otomatis (Expert Advisor) di MetaTrader 4 (MT4) dan MetaTrader 5 (MT5) yang melakukan entry dan exit berdasarkan persilangan Moving Average (MA), kita akan menggunakan bahasa pemrograman MQL4 dan MQL5.  

---

### **1. Logika Trading**
Robot trading ini akan:  
- **Entry Buy** saat MA5 melintasi MA9 dari bawah ke atas.  
- **Exit Buy** saat MA5 melintasi MA9 dari atas ke bawah.  
- **Entry Sell** saat MA5 melintasi MA9 dari atas ke bawah.  
- **Exit Sell** saat MA5 melintasi MA9 dari bawah ke atas.  
- **Stop Loss (SL) dan Take Profit (TP)** akan ditentukan berdasarkan parameter yang bisa diatur.  

---

### **2. Struktur Kode**
Kode ini terdiri dari beberapa bagian utama:  
1. **Inisialisasi Parameter** – untuk mengatur periode MA, SL, TP, dan lot size.  
2. **Fungsi untuk Mengecek Persilangan MA** – menentukan apakah ada sinyal beli atau jual.  
3. **Fungsi untuk Entry dan Exit Posisi** – membuka dan menutup trade secara otomatis.  
4. **Fungsi OnTick()** – mengevaluasi kondisi pasar setiap tick harga.  

---

### **3. Script MQL4 untuk MT4**
Simpan kode berikut sebagai file **XAUUSD_MA_Trading.mq4** di folder **MQL4/Experts/**.

```mql4
//+------------------------------------------------------------------+
//| Robot Trading MA Crossover untuk XAU/USD (H1)                  |
//| Dibuat untuk MetaTrader 4                                       |
//+------------------------------------------------------------------+
#property strict

// Parameter Input
input int maShortPeriod = 5;
input int maLongPeriod = 9;
input double lotSize = 0.1;
input double stopLoss = 500;  // dalam pips
input double takeProfit = 1000; // dalam pips

// Fungsi untuk mendapatkan nilai MA
double GetMA(int period, int shift) {
    return iMA(Symbol(), PERIOD_H1, period, 0, MODE_SMA, PRICE_CLOSE, shift);
}

// Fungsi untuk Entry dan Exit
void CheckTrade() {
    double maShortCurrent = GetMA(maShortPeriod, 0);
    double maShortPrevious = GetMA(maShortPeriod, 1);
    double maLongCurrent = GetMA(maLongPeriod, 0);
    double maLongPrevious = GetMA(maLongPeriod, 1);
    
    // Buy Entry
    if (maShortPrevious < maLongPrevious && maShortCurrent > maLongCurrent) {
        if (OrdersTotal() == 0) {
            double sl = Bid - stopLoss * Point;
            double tp = Bid + takeProfit * Point;
            OrderSend(Symbol(), OP_BUY, lotSize, Ask, 3, sl, tp, "Buy Order", 0, 0, clrBlue);
        }
    }

    // Sell Entry
    if (maShortPrevious > maLongPrevious && maShortCurrent < maLongCurrent) {
        if (OrdersTotal() == 0) {
            double sl = Ask + stopLoss * Point;
            double tp = Ask - takeProfit * Point;
            OrderSend(Symbol(), OP_SELL, lotSize, Bid, 3, sl, tp, "Sell Order", 0, 0, clrRed);
        }
    }

    // Exit Buy
    if (maShortPrevious > maLongPrevious && maShortCurrent < maLongCurrent) {
        for (int i = 0; i < OrdersTotal(); i++) {
            if (OrderSelect(i, SELECT_BY_POS, MODE_TRADES) && OrderType() == OP_BUY) {
                OrderClose(OrderTicket(), OrderLots(), Bid, 3, clrRed);
            }
        }
    }

    // Exit Sell
    if (maShortPrevious < maLongPrevious && maShortCurrent > maLongCurrent) {
        for (int i = 0; i < OrdersTotal(); i++) {
            if (OrderSelect(i, SELECT_BY_POS, MODE_TRADES) && OrderType() == OP_SELL) {
                OrderClose(OrderTicket(), OrderLots(), Ask, 3, clrBlue);
            }
        }
    }
}

// Fungsi utama untuk eksekusi robot
void OnTick() {
    CheckTrade();
}
```

---

### **4. Script MQL5 untuk MT5**
Simpan kode berikut sebagai file **XAUUSD_MA_Trading.mq5** di folder **MQL5/Experts/**.

```mql5
//+------------------------------------------------------------------+
//| Robot Trading MA Crossover untuk XAU/USD (H1)                  |
//| Dibuat untuk MetaTrader 5                                       |
//+------------------------------------------------------------------+
#property strict

// Parameter Input
input int maShortPeriod = 5;
input int maLongPeriod = 9;
input double lotSize = 0.1;
input double stopLoss = 500;  // dalam pips
input double takeProfit = 1000; // dalam pips

// Fungsi untuk mendapatkan nilai MA
double GetMA(int period, int shift) {
    return iMA(Symbol(), PERIOD_H1, period, 0, MODE_SMA, PRICE_CLOSE, shift);
}

// Fungsi untuk Entry dan Exit
void CheckTrade() {
    double maShortCurrent = GetMA(maShortPeriod, 0);
    double maShortPrevious = GetMA(maShortPeriod, 1);
    double maLongCurrent = GetMA(maLongPeriod, 0);
    double maLongPrevious = GetMA(maLongPeriod, 1);
    
    MqlTradeRequest request;
    MqlTradeResult result;
    
    // Buy Entry
    if (maShortPrevious < maLongPrevious && maShortCurrent > maLongCurrent) {
        if (PositionSelect(Symbol()) == false) {
            request.action = TRADE_ACTION_DEAL;
            request.type = ORDER_TYPE_BUY;
            request.volume = lotSize;
            request.price = SymbolInfoDouble(Symbol(), SYMBOL_ASK);
            request.sl = request.price - stopLoss * Point;
            request.tp = request.price + takeProfit * Point;
            request.magic = 123456;
            request.symbol = Symbol();
            OrderSend(request, result);
        }
    }

    // Sell Entry
    if (maShortPrevious > maLongPrevious && maShortCurrent < maLongCurrent) {
        if (PositionSelect(Symbol()) == false) {
            request.action = TRADE_ACTION_DEAL;
            request.type = ORDER_TYPE_SELL;
            request.volume = lotSize;
            request.price = SymbolInfoDouble(Symbol(), SYMBOL_BID);
            request.sl = request.price + stopLoss * Point;
            request.tp = request.price - takeProfit * Point;
            request.magic = 123456;
            request.symbol = Symbol();
            OrderSend(request, result);
        }
    }

    // Exit Buy
    if (maShortPrevious > maLongPrevious && maShortCurrent < maLongCurrent) {
        if (PositionSelect(Symbol()) && PositionGetInteger(POSITION_TYPE) == POSITION_TYPE_BUY) {
            PositionClose(Symbol());
        }
    }

    // Exit Sell
    if (maShortPrevious < maLongPrevious && maShortCurrent > maLongCurrent) {
        if (PositionSelect(Symbol()) && PositionGetInteger(POSITION_TYPE) == POSITION_TYPE_SELL) {
            PositionClose(Symbol());
        }
    }
}

// Fungsi utama untuk eksekusi robot
void OnTick() {
    CheckTrade();
}
```

---

### **5. Cara Menggunakan**
1. **Buka MetaTrader 4 atau 5**.  
2. **Buka MetaEditor (Ctrl+M)** → Tambahkan script ke folder **Experts**.  
3. **Compile** script dan pastikan tidak ada error.  
4. **Tambahkan EA ke chart XAUUSD dengan timeframe H1**.  
5. **Aktifkan Auto Trading** dan biarkan robot bekerja secara otomatis.  

Robot ini akan bekerja secara otomatis mengikuti strategi persilangan MA5 dan MA9 pada timeframe H1.
