# Robot Trading Otomatis Untuk XAUUSD(H1)
Untuk membuat robot trading otomatis (Expert Advisor) di **MetaTrader 4 (MT4) dan MetaTrader 5 (MT5)**, kita akan menggunakan bahasa **MQL4/MQL5**. Robot ini akan:  

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

### **Cara Menggunakan:**
1. **Simpan file di folder `Experts` sesuai platform (`MQL4` atau `MQL5`).**  
2. **Buka MT4/MT5**, masuk ke `Navigator > Expert Advisors`, lalu `Refresh`.  
3. **Tarik EA ke chart XAU/USD H1** dan pastikan `Algo Trading` aktif.  

Sekarang robot akan berjalan otomatis berdasarkan strategi MA5 dan MA9!
