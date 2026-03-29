#@title HK Quant Master V5.5 (整合防暴雷機制與追蹤結算) perfrect
import yfinance as yf
import pandas as pd
import numpy as np
import datetime
import json
import warnings
import time
import requests
import os
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

warnings.filterwarnings('ignore')

# ==============================================================================
# 1. 設定觀察名單、參數與中文名稱對照表
# ==============================================================================
WATCHLIST = [
    '0001.HK', '0002.HK', '0003.HK', '0005.HK', '0006.HK', '0011.HK', '0012.HK', '0016.HK', '0017.HK', '0020.HK',
    '0027.HK', '0066.HK', '0083.HK', '0101.HK', '0119.HK', '0135.HK', '0144.HK', '0151.HK', '0168.HK', '0175.HK',
    '0200.HK', '0241.HK', '0256.HK', '0267.HK', '0268.HK', '0270.HK', '0272.HK', '0285.HK', '0288.HK', '0291.HK',
    '0316.HK', '0322.HK', '0336.HK', '0345.HK', '0354.HK', '0358.HK', '0386.HK', '0388.HK', '0390.HK', '0460.HK',
    '0520.HK', '0522.HK', '0552.HK', '0576.HK', '0586.HK', '0598.HK', '0604.HK', '0656.HK', '0669.HK', '0688.HK',
    '0700.HK', '0728.HK', '0753.HK', '0762.HK', '0772.HK', '0778.HK', '0780.HK', '0813.HK', '0823.HK', '0836.HK',
    '0853.HK', '0857.HK', '0861.HK', '0868.HK', '0883.HK', '0902.HK', '0909.HK', '0914.HK', '0916.HK', '0934.HK',
    '0939.HK', '0941.HK', '0960.HK', '0968.HK', '0981.HK', '0992.HK', '0998.HK', '1024.HK', '1030.HK', '1038.HK',
    '1044.HK', '1055.HK', '1066.HK', '1071.HK', '1088.HK', '1093.HK', '1099.HK', '1109.HK', '1113.HK', '1119.HK',
    '1138.HK', '1157.HK', '1177.HK', '1193.HK', '1209.HK', '1211.HK', '1258.HK', '1299.HK', '1308.HK', '1313.HK',
    '1316.HK', '1336.HK', '1339.HK', '1347.HK', '1368.HK', '1378.HK', '1398.HK', '1516.HK', '1530.HK', '1658.HK',
    '1772.HK', '1787.HK', '1801.HK', '1810.HK', '1818.HK', '1833.HK', '1876.HK', '1898.HK', '1919.HK', '1928.HK',
    '1929.HK', '1997.HK', '2005.HK', '2007.HK', '2013.HK', '2015.HK', '2018.HK', '2020.HK', '2186.HK', '2192.HK',
    '2202.HK', '2238.HK', '2269.HK', '2313.HK', '2318.HK', '2319.HK', '2331.HK', '2333.HK', '2359.HK', '2380.HK',
    '2388.HK', '2600.HK', '2618.HK', '2628.HK', '2669.HK', '2688.HK', '2689.HK', '2727.HK', '2858.HK', '2866.HK',
    '2869.HK', '2877.HK', '2883.HK', '2899.HK', '3311.HK', '3319.HK', '3323.HK', '3328.HK', '3331.HK', '3606.HK',
    '3618.HK', '3633.HK', '3690.HK', '3692.HK', '3738.HK', '3800.HK', '3868.HK', '3888.HK', '3899.HK', '3900.HK',
    '3908.HK', '3933.HK', '3958.HK', '3968.HK', '3983.HK', '3988.HK', '3990.HK', '3993.HK', '6030.HK', '6098.HK',
    '6110.HK', '6160.HK', '6618.HK', '6690.HK', '6806.HK', '6837.HK', '6862.HK', '6865.HK', '6881.HK', '6969.HK',
    '9618.HK', '9633.HK', '9866.HK', '9868.HK', '9888.HK', '9922.HK', '9959.HK', '9988.HK', '9992.HK', '9999.HK'
]

HK_STOCK_NAMES = {
    '0001.HK': '長和', '0002.HK': '中電控股', '0003.HK': '香港中華煤氣', '0005.HK': '匯豐控股', '0006.HK': '電能實業', 
    '0011.HK': '恒生銀行', '0012.HK': '恆基地產', '0016.HK': '新鴻基地產', '0017.HK': '新世界發展', '0020.HK': '商汤-W',
    '0027.HK': '銀河娛樂', '0066.HK': '港鐵公司', '0083.HK': '信和置業', '0101.HK': '恒隆地產', '0119.HK': '保利置業集團', 
    '0135.HK': '昆侖能源', '0144.HK': '招商局港口', '0151.HK': '中國旺旺', '0168.HK': '青島啤酒股份', '0175.HK': '吉利汽車',
    '0200.HK': '新濠國際發展', '0241.HK': '阿里健康', '0256.HK': '冠城鐘錶珠寶', '0267.HK': '中信股份', '0268.HK': '金蝶國際', 
    '0270.HK': '粵海投資', '0272.HK': '瑞安房地產', '0285.HK': '比亞迪電子', '0288.HK': '萬洲國際', '0291.HK': '華潤啤酒',
    '0316.HK': '東方海外國際', '0322.HK': '康師傅控股', '0336.HK': '華寶國際', '0345.HK': '維他奶國際', '0354.HK': '中國軟件國際', 
    '0358.HK': '江西銅業股份', '0386.HK': '中國石油化工股份', '0388.HK': '香港交易所', '0390.HK': '中國中鐵', '0460.HK': '四環醫藥',
    '0520.HK': '呷哺呷哺', '0522.HK': 'ASMPT', '0552.HK': '中國通信服務', '0576.HK': '浙江滬杭甬', '0586.HK': '海螺創業', 
    '0598.HK': '中國外運', '0604.HK': '深圳控股', '0656.HK': '復星國際', '0669.HK': '創科實業', '0688.HK': '中國海外發展',
    '0700.HK': '騰訊控股', '0728.HK': '中國電信', '0753.HK': '中國國航', '0762.HK': '中國聯通', '0772.HK': '閱文集團', 
    '0778.HK': '置富產業信託', '0780.HK': '同程旅行', '0813.HK': '世茂集團', '0823.HK': '領展房產基金', '0836.HK': '華潤電力',
    '0853.HK': '微創醫療', '0857.HK': '中國石油股份', '0861.HK': '神州控股', '0868.HK': '信義玻璃', '0883.HK': '中國海洋石油', 
    '0902.HK': '華能國際電力', '0909.HK': '明源雲', '0914.HK': '海螺水泥', '0916.HK': '龍源電力', '0934.HK': '中石化冠德',
    '0939.HK': '建設銀行', '0941.HK': '中國移動', '0960.HK': '龍湖集團', '0968.HK': '信義光能', '0981.HK': '中芯國際', 
    '0992.HK': '聯想集團', '0998.HK': '中信銀行', '1024.HK': '快手-W', '1030.HK': '新城發展', '1038.HK': '長江基建集團',
    '1044.HK': '恒安國際', '1055.HK': '中國南方航空', '1066.HK': '威高股份', '1071.HK': '華電國際電力', '1088.HK': '中國神華', 
    '1093.HK': '石藥集團', '1099.HK': '國藥控股', '1109.HK': '華潤置地', '1113.HK': '長實集團', '1119.HK': '創夢天地',
    '1138.HK': '中遠海能', '1157.HK': '中聯重科', '1177.HK': '中國生物製藥', '1193.HK': '華潤燃氣', '1209.HK': '華潤萬象生活', 
    '1211.HK': '比亞迪股份', '1258.HK': '中國有色礦業', '1299.HK': '友邦保險', '1308.HK': '海豐國際', '1313.HK': '華潤建材科技',
    '1316.HK': '耐世特', '1336.HK': '新華保險', '1339.HK': '中國人民保險', '1347.HK': '華虹半導體', '1368.HK': '特步國際', 
    '1378.HK': '中國宏橋', '1398.HK': '工商銀行', '1516.HK': '融創服務', '1530.HK': '三生製藥', '1658.HK': '郵儲銀行',
    '1772.HK': '贛鋒鋰業', '1787.HK': '山東黃金', '1801.HK': '信達生物', '1810.HK': '小米集團-W', '1818.HK': '招金礦業', 
    '1833.HK': '平安好醫生', '1876.HK': '百威亞太', '1898.HK': '中煤能源', '1919.HK': '中遠海控', '1928.HK': '金沙中國',
    '1929.HK': '周大福', '1997.HK': '九龍倉置業', '2005.HK': '聯邦制藥', '2007.HK': '碧桂園', '2013.HK': '微盟集團', 
    '2015.HK': '理想汽車-W', '2018.HK': '瑞聲科技', '2020.HK': '安踏體育', '2186.HK': '綠葉製藥', '2192.HK': '醫渡科技',
    '2202.HK': '萬科企業', '2238.HK': '廣汽集團', '2269.HK': '藥明生物', '2313.HK': '申洲國際', '2318.HK': '中國平安', 
    '2319.HK': '蒙牛乳業', '2331.HK': '李寧', '2333.HK': '長城汽車', '2359.HK': '藥明康德', '2380.HK': '中國電力',
    '2388.HK': '中銀香港', '2600.HK': '中國鋁業', '2618.HK': '京東物流', '2628.HK': '中國人壽', '2669.HK': '中海物業', 
    '2688.HK': '新奧能源', '2689.HK': '玖龍紙業', '2727.HK': '上海電氣', '2858.HK': '易鑫集團', '2866.HK': '中遠海發',
    '2869.HK': '綠城服務', '2877.HK': '神威藥業', '2883.HK': '中海油田服務', '2899.HK': '紫金礦業', '3311.HK': '中國建築國際', 
    '3319.HK': '雅生活服務', '3323.HK': '中國建材', '3328.HK': '交通銀行', '3331.HK': '維達國際', '3606.HK': '福耀玻璃',
    '3618.HK': '重慶農村商業銀行', '3633.HK': '中裕能源', '3690.HK': '美團-W', '3692.HK': '翰森製藥', '3738.HK': '阜博集團', 
    '3800.HK': '協鑫科技', '3868.HK': '信義能源', '3888.HK': '金山軟件', '3899.HK': '中集安瑞科', '3900.HK': '綠城中國',
    '3908.HK': '中金公司', '3933.HK': '聯邦制藥', '3958.HK': '東方證券', '3968.HK': '招商銀行', '3983.HK': '中海石油化學', 
    '3988.HK': '中國銀行', '3990.HK': '美的置業', '3993.HK': '洛陽鉬業', '6030.HK': '中信証券', '6098.HK': '碧桂園服務',
    '6110.HK': '滔搏', '6160.HK': '百濟神州', '6618.HK': '京東健康', '6690.HK': '海爾智家', '6806.HK': '申萬宏源', 
    '6837.HK': '海通證券', '6862.HK': '海底撈', '6865.HK': '福萊特玻璃', '6881.HK': '中國銀河', '6969.HK': '思摩爾國際',
    '9618.HK': '京東集團-SW', '9633.HK': '農夫山泉', '9866.HK': '蔚來-SW', '9868.HK': '小鵬汽車-W', '9888.HK': '百度集團-SW', 
    '9922.HK': '九毛九', '9959.HK': '聯易融科技-W', '9988.HK': '阿里巴巴-SW', '9992.HK': '泡泡瑪特', '9999.HK': '網易-S'
}

print("⏳ 1/4 啟動下載 Agent 獲取最新市場數據 (具備防呆與重試機制)...")

# ── 1. 下載大盤 ──
hsi_df = yf.download("2800.HK", period="2y", progress=False, threads=False) # 延長為2年以計算250日指標
if hsi_df.empty:
    hsi_df = yf.download("^HSI", period="2y", progress=False, threads=False)

if isinstance(hsi_df.columns, pd.MultiIndex):
    hsi_col = 'Adj Close' if 'Adj Close' in hsi_df.columns.get_level_values(0) else 'Close'
    hsi_c = hsi_df[hsi_col].iloc[:, 0].ffill()
else:
    hsi_col = 'Adj Close' if 'Adj Close' in hsi_df.columns else 'Close'
    hsi_c = hsi_df[hsi_col].ffill()

# ── 2. 建立 Data Fetching Agent ──
def secured_download_agent(tickers, period="2y"): # 延長為2年
    prices_dict = {'Close': {}, 'High': {}, 'Low': {}, 'Volume': {}}
    failed_tickers = []
    batch_size = 30 
    for i in range(0, len(tickers), batch_size):
        batch = tickers[i:i + batch_size]
        print(f"   🤖 Agent 正在安全下載批次 {i//batch_size + 1} ({len(batch)} 檔標的)...")
        data = yf.download(batch, period=period, progress=False, threads=True, group_by='ticker')
        for ticker in batch:
            try:
                df_t = data if len(batch) == 1 else data[ticker]
                if df_t.empty or df_t.isna().all().all():
                    failed_tickers.append(ticker)
                    continue
                close_col = 'Adj Close' if 'Adj Close' in df_t.columns else 'Close'
                prices_dict['Close'][ticker] = df_t[close_col]
                prices_dict['High'][ticker] = df_t['High']
                prices_dict['Low'][ticker] = df_t['Low']
                prices_dict['Volume'][ticker] = df_t['Volume']
            except Exception:
                failed_tickers.append(ticker)
        time.sleep(1) 
    print(f"   ✅ Agent 任務完成！成功: {len(prices_dict['Close'])} 檔 | 失敗/下市: {len(failed_tickers)} 檔")
    return (pd.DataFrame(prices_dict['Close']).ffill(),
            pd.DataFrame(prices_dict['High']).ffill(),
            pd.DataFrame(prices_dict['Low']).ffill(),
            pd.DataFrame(prices_dict['Volume']).ffill())

closes, highs, lows, vols = secured_download_agent(WATCHLIST, period="2y")

if closes.empty:
    print("❌ 嚴重錯誤：所有標的皆無法下載，請檢查網路連線或 Yahoo API 狀態。")
else:
    print(f"📊 實際成功下載並合併了 {closes.shape[1]} 檔股票的有效數據！\n")

print("⏳ 2/4 計算技術指標與大盤狀態...")

# ── 大盤狀態判定 ──
hsi_200ma = hsi_c.rolling(200).mean()
current_hsi_price = hsi_c.iloc[-1]
current_hsi_200ma = hsi_200ma.iloc[-1]
is_bull_market = current_hsi_price > current_hsi_200ma

market_status = "🟢 牛市狀態 (基準 > 200MA)" if is_bull_market else "🔴 熊市/震盪狀態 (基準 < 200MA)"
active_strategy = "Donchian Turtle (海龜20日突破)" if is_bull_market else "Mean Reversion (RSI超賣抄底)"

# ── 個股技術指標與【基本面防護網指標】計算 ──
delta = closes.diff()
gain = (delta.where(delta > 0, 0)).rolling(window=14).mean()
loss = (-delta.where(delta < 0, 0)).rolling(window=14).mean()
rs = gain / loss.replace(0, np.nan)
rsi = 100 - (100 / (1 + rs))

sma20 = closes.rolling(20).mean()
std20 = closes.rolling(20).std()
lower_bb = sma20 - (2 * std20)
donchian_high = highs.rolling(20).max().shift(1)
avg_vol_20 = vols.rolling(20).mean()

# 新增：250日最高價與個股200MA防護網
high_250 = highs.rolling(250).max()
stock_200ma = closes.rolling(200).mean()

print("⏳ 3/4 掃描觸發訊號並深度抓取【基本面數據】...")
signals = []

def safe_list(series):
    return [None if pd.isna(x) else round(float(x), 2) for x in series.tolist()]

for ticker in closes.columns:
    if ticker not in WATCHLIST: continue

    recent_c = closes[ticker].tail(5)
    recent_rsi = rsi[ticker].tail(5)
    recent_lbb = lower_bb[ticker].tail(5)
    recent_dh = donchian_high[ticker].tail(5)

    cur_c = recent_c.iloc[-1]
    if pd.isna(cur_c): continue

    # 取得防護網指標
    cur_h250 = high_250[ticker].iloc[-1]
    cur_200ma = stock_200ma[ticker].iloc[-1]

    trigger_type = None
    trigger_days = 0
    sl_price = 0.0
    tp_price = 0.0

    # ── 判斷技術面觸發與【防暴雷體質檢驗】 ──
    if is_bull_market:
        is_triggered = recent_c > recent_dh
        if is_triggered.iloc[-1]:
            # 防護網：牛市突破，股價必須站穩年線 (200MA) 才算真強勢
            if not pd.isna(cur_200ma) and cur_c > cur_200ma:
                trigger_type = "海龜突破 (順勢)"
                sl_price = cur_c * 0.90
                tp_price = cur_c * 1.30
                for val in reversed(is_triggered.values):
                    if val: trigger_days += 1
                    else: break
            else:
                print(f"   ⚠️ 防護網攔截: {ticker} 雖突破但未站上年線，判定為假突破。")
    else:
        is_triggered = (recent_rsi < 30) & (recent_c < recent_lbb)
        if is_triggered.iloc[-1]:
            # 防護網：熊市抄底，股價絕不能跌破過去一年最高價的 40% (即回撤不能 > 60%)
            if not pd.isna(cur_h250) and cur_c > (cur_h250 * 0.40):
                trigger_type = "RSI 超跌抄底 (逆勢)"
                sl_price = cur_c * 0.88
                tp_price = cur_c * 1.20
                for val in reversed(is_triggered.values):
                    if val: trigger_days += 1
                    else: break
            else:
                print(f"   ☠️ 死亡螺旋攔截: {ticker} 股價已從高點腰斬再腰斬，拒絕接刀！")

    # ── 若技術面與防護網皆通過，抓取基本面財報資料 ──
    if trigger_type:
        
        stock_name_zh = HK_STOCK_NAMES.get(ticker, "")
        print(f"   ➤ 發現訊號: {ticker} {stock_name_zh}，正在讀取最新財報與股息資料...")

        cur_h52 = highs[ticker].max()
        drawdown_52w = ((cur_c - cur_h52) / cur_h52) * 100 if cur_h52 > 0 else 0
        cur_vol = avg_vol_20[ticker].iloc[-1]
        daily_turnover_est = (cur_vol * cur_c) / 1000000

        div_yield_pct = 0.0
        earn_label = "無資料"
        pe_ratio = "N/A"
        pb_ratio = "N/A"
        roe_pct = "N/A"

        try:
            tk_info = yf.Ticker(ticker).info
            
            if not stock_name_zh:
                stock_name_zh = tk_info.get('shortName', '')

            raw_div = tk_info.get('dividendYield') or tk_info.get('trailingAnnualDividendYield') or 0
            if raw_div > 1: div_yield_pct = round(raw_div, 2)
            else: div_yield_pct = round(raw_div * 100, 2)

            earn_growth = tk_info.get('earningsGrowth') or tk_info.get('revenueGrowth') or 0
            earn_growth_pct = round(earn_growth * 100, 2)

            if earn_growth_pct >= 15: earn_label = f"強勁成長 (+{earn_growth_pct}%)"
            elif earn_growth_pct > 0: earn_label = f"溫和復甦 (+{earn_growth_pct}%)"
            elif earn_growth_pct < 0: earn_label = f"衰退中 ({earn_growth_pct}%)"
            else: earn_label = "未公佈"

            if tk_info.get('trailingPE'): pe_ratio = round(tk_info.get('trailingPE'), 2)
            if tk_info.get('priceToBook'): pb_ratio = round(tk_info.get('priceToBook'), 2)
            if tk_info.get('returnOnEquity'): roe_pct = round(tk_info.get('returnOnEquity') * 100, 2)

        except Exception as e:
            pass

        tv_ticker = f"HKEX:{int(ticker.split('.')[0])}" if ticker.split('.')[0].isdigit() else ticker

        signals.append({
            "ticker": ticker,
            "stock_name": stock_name_zh,
            "tv_ticker": tv_ticker,
            "price": round(cur_c, 2),
            "type": trigger_type,
            "trigger_days": trigger_days,
            "sl": round(sl_price, 2),
            "tp": round(tp_price, 2),
            "rsi": round(recent_rsi.iloc[-1], 1),
            "dd_52w": round(drawdown_52w, 1),
            "turnover_m": round(daily_turnover_est, 1),
            "div_yield": div_yield_pct,
            "earn_label": earn_label,
            "pe_ratio": pe_ratio,
            "pb_ratio": pb_ratio,
            "roe": roe_pct,
            "chart_dates": closes.index[-100:].strftime('%m-%d').tolist(),
            "chart_prices": safe_list(closes[ticker].tail(100)),
            "chart_sma20": safe_list(sma20[ticker].tail(100)),
            "chart_lbb": safe_list(lower_bb[ticker].tail(100))
        })

print(f"✅ 掃描完成！今日共發現 {len(signals)} 檔標的。")

# ── 追蹤紀錄 (Tracking Log) 更新與結算處理 ──
print("⏳ 處理歷史追蹤紀錄 (更新每日價格與止損/止盈狀態)...")
log_file = "tracking_log.json"
if os.path.exists(log_file):
    try:
        with open(log_file, 'r', encoding='utf-8') as f:
            tracking_history = json.load(f)
    except:
        tracking_history = []
else:
    tracking_history = []

today_str = datetime.datetime.now().strftime('%Y-%m-%d')

for item in tracking_history:
    if 'status' not in item: item['status'] = 'Active'
    if 'current_price' not in item: item['current_price'] = item['price']
    if 'sl' not in item: item['sl'] = round(item['price'] * 0.88, 2)
    if 'tp' not in item: item['tp'] = round(item['price'] * 1.20, 2)
    if 'financial_data' not in item: item['financial_data'] = "無歷史紀錄"
    if 'exit_date' not in item: item['exit_date'] = "-" 
    
    if 'stock_name' not in item or not item['stock_name']:
        item['stock_name'] = HK_STOCK_NAMES.get(item['ticker'], "")

    if item['financial_data'] in ["無歷史紀錄", "無紀錄"]:
        matching_sig = next((s for s in signals if s['ticker'] == item['ticker']), None)
        if matching_sig:
            item['financial_data'] = f"股息: {matching_sig['div_yield']}% | 動能: {matching_sig['earn_label']}<br>P/E: {matching_sig['pe_ratio']} | P/B: {matching_sig['pb_ratio']} | ROE: {matching_sig['roe']}%"

    if item['status'] == 'Active' and item['ticker'] in closes.columns:
        latest_p = closes[item['ticker']].iloc[-1]
        if not pd.isna(latest_p):
            item['current_price'] = round(float(latest_p), 2)
            
            if item['current_price'] >= item['tp']:
                item['status'] = 'Win'
                item['exit_date'] = today_str 
            elif item['current_price'] <= item['sl']:
                item['status'] = 'Loss'
                item['exit_date'] = today_str 

existing_today_tickers = [item['ticker'] for item in tracking_history if item['date'] == today_str]
for sig in signals:
    if sig['ticker'] not in existing_today_tickers:
        financial_summary = f"股息: {sig['div_yield']}% | 動能: {sig['earn_label']}<br>P/E: {sig['pe_ratio']} | P/B: {sig['pb_ratio']} | ROE: {sig['roe']}%"
        tracking_history.append({
            "date": today_str,
            "ticker": sig['ticker'],
            "stock_name": sig['stock_name'],
            "type": sig['type'],
            "price": sig['price'],
            "current_price": sig['price'],
            "sl": sig['sl'],
            "tp": sig['tp'],
            "status": "Active",
            "exit_date": "-",
            "financial_data": financial_summary
        })

with open(log_file, 'w', encoding='utf-8') as f:
    json.dump(tracking_history, f, ensure_ascii=False, indent=2)


# ==============================================================================
# 4. 生成互動式 HTML Dashboard (V5.5)
# ==============================================================================
print("⏳ 4/4 正在生成 Dashboard HTML...")

dashboard_data = {
    "date": datetime.datetime.now().strftime('%Y-%m-%d %H:%M'),
    "market_status": market_status,
    "active_strategy": active_strategy,
    "hsi_price": round(current_hsi_price, 2),
    "hsi_200ma": round(current_hsi_200ma, 2),
    "signals": signals
}

html_content = f"""
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <title>HK Quant Master V5.5 - 每日一篇教66歲丫媽學投資</title>
    <style>
        body {{ background-color: #0f172a; color: #e2e8f0; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; }}
        .card {{ background-color: #1e293b; border: 1px solid #334155; border-radius: 12px; }}
        .bull-text {{ color: #10b981; }}
        .bear-text {{ color: #ef4444; }}
        .info-badge {{ font-size: 0.70rem; padding: 4px 6px; border-radius: 6px; background: rgba(30, 41, 59, 0.8); border: 1px solid #475569; text-align: center; }}
        
        /* Tab CSS */
        .tab-btn {{ transition: all 0.3s ease; }}
        .tab-active {{ background-color: #2563eb; color: white; border-bottom: 2px solid #60a5fa; }}
        .tab-inactive {{ background-color: transparent; color: #94a3b8; border-bottom: 2px solid transparent; hover:text-white; }}
    </style>
</head>
<body class="p-6">
    <div class="max-w-6xl mx-auto">
        
        <a href="https://www.threads.com/@teachthe66yearsoldmominvest" target="_blank" class="block w-full bg-gradient-to-r from-purple-600 to-blue-600 text-white text-center py-3 rounded-lg font-bold mb-4 shadow-lg hover:from-purple-500 hover:to-blue-500 transition">
            ✨ 追蹤【每日一篇教66歲丫媽學投資】Threads 帳號，學習更多簡單實用的投資策略！ ✨
        </a>

        <div class="card p-6 mb-6 flex flex-col md:flex-row justify-between items-center shadow-lg border-b-4 border-blue-500">
            <div>
                <h1 class="text-3xl font-black text-white mb-2">HK Quant Master V5.5 <span class="text-blue-500">體質防護版</span></h1>
                <p class="text-slate-400 text-sm">技術面發掘買點，動態防護網鎖定底氣 | 最後更新：{dashboard_data['date']}</p>
            </div>
            <div class="mt-4 md:mt-0 text-right bg-slate-900 p-4 rounded-lg border border-slate-700">
                <p class="text-xs text-slate-400 mb-1">自動判斷當前大盤狀態</p>
                <div class="text-xl font-bold {'bull-text' if is_bull_market else 'bear-text'}">{dashboard_data['market_status']}</div>
                <div class="text-sm mt-2 text-slate-300">目前啟動引擎：<span class="font-bold text-yellow-400">{dashboard_data['active_strategy']}</span></div>
            </div>
        </div>

        <div class="flex border-b border-slate-700 mb-6 space-x-2">
            <button onclick="switchTab('tab-scan')" id="btn-scan" class="tab-btn tab-active px-6 py-3 font-bold rounded-t-lg">📊 今日掃描結果</button>
            <button onclick="switchTab('tab-manual')" id="btn-manual" class="tab-btn tab-inactive px-6 py-3 font-bold rounded-t-lg">📖 系統說明書</button>
            <button onclick="switchTab('tab-log')" id="btn-log" class="tab-btn tab-inactive px-6 py-3 font-bold rounded-t-lg">📈 歷史追蹤與結算 (Win/Loss)</button>
        </div>

        <div id="tab-scan" class="tab-content block">
            <div class="flex flex-col lg:flex-row gap-6">
                <div class="lg:w-2/5 flex flex-col gap-4 overflow-y-auto max-h-[850px] pr-2">
                    <h2 class="text-xl font-bold border-b border-slate-700 pb-2 sticky top-0 bg-[#0f172a] z-10 pt-2">🎯 嚴選觸發標的 ({len(signals)})</h2>
"""

if not signals:
    html_content += f"""
                    <div class="card p-6 text-center text-slate-500">
                        <p class="text-4xl mb-2">🛡️</p>
                        <p>今日無符合條件的標的。</p>
                        <p class="text-xs text-slate-400 mt-2">系統的「死亡螺旋防護網」已開啟，為您過濾高風險訊號。</p>
                    </div>
"""
else:
    for i, sig in enumerate(signals):
        badge_color = "bg-green-900/50 text-green-400 border-green-700" if "海龜" in sig['type'] else "bg-red-900/50 text-red-400 border-red-700"
        day_text = f"🔥 最新訊號 (第1天)" if sig['trigger_days'] == 1 else f"⚡ 連續觸發 {sig['trigger_days']} 天"
        day_color = "text-yellow-400" if sig['trigger_days'] == 1 else "text-slate-400"

        dd_color = "text-red-400" if sig['dd_52w'] < -40 else "text-slate-300"
        div_color = "text-green-400 font-black" if sig['div_yield'] >= 6 else "text-slate-300"
        earn_color = "text-green-400" if "+" in sig['earn_label'] else "text-red-400" if "衰退" in sig['earn_label'] else "text-slate-400"

        html_content += f"""
                    <div class="card p-4 cursor-pointer hover:bg-slate-700 transition shadow" onclick="loadChart('{sig['ticker']}')">
                        <div class="flex justify-between items-center mb-2">
                            <div class="text-2xl font-black text-white">{sig['ticker']} <span class="text-lg text-slate-300 ml-1">{sig['stock_name']}</span></div>
                            <div class="text-xs px-2 py-1 rounded border {badge_color} whitespace-nowrap ml-2">{sig['type']}</div>
                        </div>

                        <div class="text-xs font-bold {day_color} mb-3 border-b border-slate-700 pb-2">
                            {day_text} | RSI: {sig['rsi']}
                        </div>

                        <div class="grid grid-cols-2 gap-2 mb-2 bg-slate-800 p-2 rounded">
                            <div class="info-badge">
                                <span class="text-slate-400 block mb-1">股息殖利率</span>
                                <span class="{div_color} text-sm">{sig['div_yield']}%</span>
                            </div>
                            <div class="info-badge">
                                <span class="text-slate-400 block mb-1">近期業績動能</span>
                                <span class="{earn_color} font-bold text-sm">{sig['earn_label']}</span>
                            </div>
                        </div>
                        
                        <div class="grid grid-cols-3 gap-2 mb-3 bg-slate-800 p-2 rounded">
                            <div class="info-badge">
                                <span class="text-slate-400 block mb-1">本益比(P/E)</span>
                                <span class="text-slate-300 font-bold text-sm">{sig['pe_ratio']}</span>
                            </div>
                            <div class="info-badge">
                                <span class="text-slate-400 block mb-1">股價淨值比(P/B)</span>
                                <span class="text-slate-300 font-bold text-sm">{sig['pb_ratio']}</span>
                            </div>
                            <div class="info-badge">
                                <span class="text-slate-400 block mb-1">ROE</span>
                                <span class="text-slate-300 font-bold text-sm">{sig['roe']}%</span>
                            </div>
                        </div>

                        <div class="grid grid-cols-3 gap-2 text-center bg-slate-900 p-2 rounded border border-slate-700">
                            <div>
                                <div class="text-[10px] text-slate-500">現價進場</div>
                                <div class="font-bold text-white">${sig['price']}</div>
                            </div>
                            <div>
                                <div class="text-[10px] text-red-400">嚴格止損</div>
                                <div class="font-bold text-red-400">${sig['sl']}</div>
                            </div>
                            <div>
                                <div class="text-[10px] text-green-400">目標止盈</div>
                                <div class="font-bold text-green-400">${sig['tp']}</div>
                            </div>
                        </div>
                    </div>
"""

html_content += f"""
                </div>

                <div class="lg:w-3/5 flex flex-col">
                    <div class="card p-4 mb-4 flex justify-between items-center bg-gradient-to-r from-slate-800 to-slate-900">
                        <h3 id="chart_title" class="text-xl font-bold text-white">點擊左側股票載入圖表</h3>
                        <a id="tv_link" href="#" target="_blank" class="hidden bg-blue-600 hover:bg-blue-500 text-white text-sm font-bold py-2 px-4 rounded transition shadow">
                            📊 在 TradingView 開啟完整圖表
                        </a>
                    </div>

                    <div class="card p-4 h-[400px] flex items-center justify-center relative shadow-inner">
                        <p id="chart_placeholder" class="text-slate-500 absolute z-0">圖表顯示區 (近100日走勢)</p>
                        <canvas id="myChart" class="w-full h-full relative z-10 hidden"></canvas>
                    </div>

                    <div class="mt-4 card p-5 text-sm bg-slate-800">
                        <h3 class="font-bold text-blue-400 mb-3 border-b border-slate-700 pb-2 text-lg">🛡️ V5.5 基本面體質防護網啟動中</h3>
                        <div class="space-y-3">
                            <p class="text-slate-300">
                                <span class="bg-red-900 text-red-400 px-2 py-1 rounded text-xs mr-2 font-bold">拒絕死亡螺旋</span>
                                過去系統容易在熊市中買到「不斷破底」的地雷股。V5.5 新增了嚴格的體質防護：當準備執行「RSI 抄底」時，若該股價已經從過去一年最高點暴跌超過 60%（腰斬再腰斬），系統將強制拋棄該訊號，避免成為價值陷阱的犧牲品。
                            </p>
                            <p class="text-slate-300">
                                <span class="bg-blue-900 text-blue-400 px-2 py-1 rounded text-xs mr-2 font-bold">年線強勢過濾</span>
                                執行「海龜突破」追高策略時，股票必須站在其本身的 200 日均線（年線）之上。確保我們只在牛市中買入真正的「強勢股」，而不是假突破的死魚股。
                            </p>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <div id="tab-manual" class="tab-content hidden card p-8 leading-relaxed">
            <h2 class="text-2xl font-bold text-blue-400 mb-4 border-b border-slate-700 pb-2">📖 系統說明書 (User Manual)</h2>
            
            <h3 class="text-lg font-bold text-white mt-6 mb-2">1. 大盤狀態自動判斷 (200MA)</h3>
            <p class="text-slate-300 mb-4">系統每天會自動讀取香港恆生指數 (HSI) 的走勢，並與其 200日移動平均線 (200MA) 進行對比。當指數在 200MA 之上，代表市場處於<strong class="text-green-400">「牛市」</strong>；若在之下，則視為<strong class="text-red-400">「熊市或震盪市」</strong>。大盤環境不同，系統啟動的抓股引擎也會不同。</p>

            <h3 class="text-lg font-bold text-white mt-6 mb-2">2. 雙重交易引擎與防暴雷機制 (Anti-Value-Trap)</h3>
            <ul class="list-disc pl-6 text-slate-300 mb-4 space-y-2">
                <li><strong class="text-green-400">牛市啟動：海龜突破策略 (順勢)</strong><br>
                當市場大好時，不猜頂，跟隨趨勢。系統會尋找突破過去 20 日最高價的股票。<strong>[V5.5 升級]</strong> 為了過濾假突破，現在個股必須站在其自身的「年線 (200MA)」之上，系統才會發放買入訊號，確保只買真強勢股。</li>
                <li><strong class="text-red-400">熊市啟動：RSI 抄底策略 (逆勢)</strong><br>
                當市場恐慌時，系統會找出同時跌破「布林通道下軌」且「RSI低於30」的股票。<strong>[V5.5 升級]</strong> 為了防止買到「死亡螺旋」的地雷股，現在系統要求進場價格不能低於過去一年最高價的 40%。腰斬再腰斬的股票會被強制過濾。</li>
            </ul>

            <h3 class="text-lg font-bold text-white mt-6 mb-2">3. 基本面護城河指標 (重要！)</h3>
            <p class="text-slate-300 mb-4">選出技術面股票後，系統會自動抓取最新的財報數據，讓您判斷這檔股票是否「值得買入」：</p>
            <div class="bg-slate-800 p-4 rounded-lg mb-4 text-sm text-slate-300 grid grid-cols-1 md:grid-cols-2 gap-4">
                <div>
                    <strong class="text-white block mb-1">💰 股息殖利率 (Dividend Yield)</strong>
                    高於 6% 代表即使股價不漲，光領利息也有不錯的回報，防禦力極高。
                </div>
                <div>
                    <strong class="text-white block mb-1">📈 本益比 (P/E)</strong>
                    代表回本年數。數值越低通常代表股價越便宜（但要注意是否為衰退型企業）。
                </div>
                <div>
                    <strong class="text-white block mb-1">📊 股價淨值比 (P/B)</strong>
                    低於 1 代表股價低於公司清算價值，通常存在低估空間。
                </div>
                <div>
                    <strong class="text-white block mb-1">🏆 股東權益報酬率 (ROE)</strong>
                    代表公司幫股東賺錢的效率。一般大於 10%~15% 視為優質的好公司。
                </div>
            </div>

            <h3 class="text-lg font-bold text-white mt-6 mb-2">⚠️ 風險提示</h3>
            <p class="text-slate-400 text-sm">本系統提供的止損(SL)與止盈(TP)價位僅供參考，系統產生的訊號不構成投資建議。投資必定有風險，入場前請務必衡量自身的資金狀況並嚴格執行止損。</p>
        </div>

        <div id="tab-log" class="tab-content hidden card p-8">
            <h2 class="text-2xl font-bold text-blue-400 mb-4 border-b border-slate-700 pb-2">📈 歷史追蹤與結算面板</h2>
            <p class="text-slate-400 mb-6 text-sm">此面板每天會自動比對推薦標的的最新價格。若觸及目標會自動移入 <b>Win (達標)</b> 或 <b>Loss (止損)</b>，並記錄下結算的日期，方便覆盤與檢驗系統勝率。</p>
            
            <div class="overflow-x-auto">
                <table class="w-full text-left border-collapse whitespace-nowrap">
                    <thead>
                        <tr class="bg-slate-800 text-slate-300 text-sm border-b border-slate-700">
                            <th class="p-3">觸發日期 (進場)</th>
                            <th class="p-3">代號與名稱</th>
                            <th class="p-3">策略類型</th>
                            <th class="p-3">觸發時財務數據快照</th>
                            <th class="p-3 text-right">進場價</th>
                            <th class="p-3 text-right">止損 (SL) / 止盈 (TP)</th>
                            <th class="p-3 text-right">今日最新價</th>
                            <th class="p-3 text-center">當前狀態與日期</th>
                        </tr>
                    </thead>
                    <tbody class="text-sm text-slate-300">
"""

if not tracking_history:
    html_content += """
                        <tr><td colspan="8" class="p-4 text-center text-slate-500">目前尚無追蹤紀錄。</td></tr>
    """
else:
    for row in reversed(tracking_history):
        type_color = "text-green-400" if "順勢" in row['type'] else "text-red-400"
        
        exit_date_str = ""
        if row.get('exit_date', '-') != '-':
            exit_date_str = f"<br><span class='text-[10px] text-slate-400'>於 {row['exit_date']} 結算</span>"

        status = row.get('status', 'Active')
        if status == 'Win':
            status_badge = f'<span class="px-2 py-1 bg-green-900/50 text-green-400 border border-green-700 rounded text-xs font-bold">🟢 達標 (Win)</span>{exit_date_str}'
        elif status == 'Loss':
            status_badge = f'<span class="px-2 py-1 bg-red-900/50 text-red-400 border border-red-700 rounded text-xs font-bold">🔴 止損 (Loss)</span>{exit_date_str}'
        else:
            status_badge = '<span class="px-2 py-1 bg-yellow-900/50 text-yellow-400 border border-yellow-700 rounded text-xs font-bold animate-pulse">⏳ 持倉 (Active)</span>'

        fin_data = row.get('financial_data', '無紀錄')
        stock_name = row.get('stock_name', '')
        
        curr_price = row.get('current_price', row['price'])
        curr_p_color = "text-green-400 font-bold" if curr_price > row['price'] else "text-red-400 font-bold" if curr_price < row['price'] else "text-slate-300"

        html_content += f"""
                        <tr class="border-b border-slate-800 hover:bg-slate-700/50 transition">
                            <td class="p-3">{row['date']}</td>
                            <td class="p-3 font-bold text-white">{row['ticker']}<br><span class="text-xs text-slate-400 font-normal">{stock_name}</span></td>
                            <td class="p-3 {type_color}">{row['type']}</td>
                            <td class="p-3 text-xs text-slate-400 leading-relaxed">{fin_data}</td>
                            <td class="p-3 text-right">${row['price']}</td>
                            <td class="p-3 text-xs text-right">
                                <span class="text-red-400">SL: ${row.get('sl', '-')}</span><br>
                                <span class="text-green-400">TP: ${row.get('tp', '-')}</span>
                            </td>
                            <td class="p-3 text-right {curr_p_color}">${curr_price}</td>
                            <td class="p-3 text-center leading-tight">{status_badge}</td>
                        </tr>
        """

html_content += f"""
                    </tbody>
                </table>
            </div>
        </div>

    </div>

    <script>
        function switchTab(tabId) {{
            document.querySelectorAll('.tab-content').forEach(el => {{
                el.classList.remove('block');
                el.classList.add('hidden');
            }});
            
            document.querySelectorAll('.tab-btn').forEach(btn => {{
                btn.classList.remove('tab-active');
                btn.classList.add('tab-inactive');
            }});

            document.getElementById(tabId).classList.remove('hidden');
            document.getElementById(tabId).classList.add('block');
            document.getElementById('btn-' + tabId.split('-')[1]).classList.remove('tab-inactive');
            document.getElementById('btn-' + tabId.split('-')[1]).classList.add('tab-active');
        }}

        const signalsData = {json.dumps(signals)};
        let myChart = null;

        function loadChart(ticker) {{
            const sig = signalsData.find(s => s.ticker === ticker);
            if (!sig) return;

            document.getElementById('chart_placeholder').classList.add('hidden');
            const canvas = document.getElementById('myChart');
            canvas.classList.remove('hidden');

            document.getElementById('chart_title').innerText = ticker + " " + sig.stock_name + " (近100日走勢)";
            const tvLink = document.getElementById('tv_link');
            tvLink.href = `https://www.tradingview.com/chart/?symbol=${{sig.tv_ticker}}`;
            tvLink.classList.remove('hidden');

            const ctx = canvas.getContext('2d');
            if (myChart) myChart.destroy();

            myChart = new Chart(ctx, {{
                type: 'line',
                data: {{
                    labels: sig.chart_dates,
                    datasets: [
                        {{
                            label: '收盤價',
                            data: sig.chart_prices,
                            borderColor: '#3b82f6',
                            borderWidth: 2,
                            tension: 0.1,
                            pointRadius: 0,
                            pointHitRadius: 10
                        }},
                        {{
                            label: '20日均線',
                            data: sig.chart_sma20,
                            borderColor: '#f59e0b',
                            borderWidth: 1.5,
                            borderDash: [5, 5],
                            tension: 0.1,
                            pointRadius: 0
                        }},
                        {{
                            label: '布林下軌',
                            data: sig.chart_lbb,
                            borderColor: '#ef4444',
                            borderWidth: 1.5,
                            borderDash: [2, 2],
                            tension: 0.1,
                            pointRadius: 0
                        }}
                    ]
                }},
                options: {{
                    responsive: true,
                    maintainAspectRatio: false,
                    interaction: {{ mode: 'index', intersect: false }},
                    plugins: {{
                        legend: {{ labels: {{ color: '#e2e8f0' }} }},
                        tooltip: {{ mode: 'index', intersect: false }}
                    }},
                    scales: {{
                        x: {{ ticks: {{ color: '#94a3b8', maxTicksLimit: 10 }}, grid: {{ color: '#334155' }} }},
                        y: {{ ticks: {{ color: '#94a3b8' }}, grid: {{ color: '#334155' }} }}
                    }}
                }}
            }});
        }}

        if (signalsData.length > 0) {{
            setTimeout(() => loadChart(signalsData[0].ticker), 100);
        }}
    </script>
</body>
</html>
"""

filename = "index.html"
with open(filename, 'w', encoding='utf-8') as f:
    f.write(html_content)

print(f"🎉 成功！已加入 V5.5 基本面防護機制，生成完美儀表板：{filename}")
try:
    from google.colab import files
    files.download(filename)
except:
    pass
