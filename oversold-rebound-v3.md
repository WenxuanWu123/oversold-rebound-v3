# Oversold Rebound V3

**A dual-tier mean-reversion signal system for the SSE Composite Index (上证综指), built for swing-trading the 510210 SSE Composite ETF.**

Written in the TongDaXin / Tonghuashun (通达信/同花顺) formula language — the DSL used by most Chinese retail trading platforms. Works on TDX desktop, TDX mobile, THS desktop, and most broker-supplied TDX-kernel clients.

---

## Why this exists

This indicator was born from a real trading need: buying panic dips in the SSE Composite Index via the 510210 ETF, and selling the rebound. It went through three iterations:

**V1 — "Cheap is enough" (got schooled by the market).**
The first version scored five dimensions (RSI, bias ratio, Bollinger position, drawdown, volume) into a 0–100 composite, triggering a buy at ≥60. On 2026-07-16 it printed 61; the next day the index crashed -3.5% intraday. Lesson learned the expensive way: *in a downtrend, oversold can get more oversold.* Cheap without evidence of stabilization is catching a falling knife.

**V2 — "Confirmation first" (too strict).**
V2 reweighted the system: cheapness is worth only 55 points, **evidence of seller exhaustion is worth 45**. A panic-capitulation pattern (intraday drop ≥2.5% + close recovering ≥50% of the day's range + volume spike) carries the largest single weight (30 pts), and every buy requires confirmation (same-day capitulation, or a close above the warning day's high within 3 days). V2 went 4-for-4 over three years — but fired barely once a year. Great defense, no offense.

**V3 — Dual tiers (current).**
The single high bar was split into two tiers:
- **Tier A (strong):** the original V2 logic — rare but high-conviction (3 signals in 3 years)
- **Tier B (standard):** a lower bar (score ≥40) with three lightweight confirmations (10 signals in 3 years)

Result over 3 years: **13 trades, 77% win rate, +1.56% avg per trade, 2.1 profit factor, max single loss −2.4%, cumulative +20.2%** vs +18.8% for the index itself. Roughly 4 signals a year — the frequency V2 was missing, paid for with three small, controlled losses.

---

## How it works

A three-stage pipeline: **score the cheapness → confirm the turn → enforce the exit.**

### Stage 1 — Composite score (0–100)

| Component | Weight | What it measures |
|---|---|---|
| RSI(6) | 0–15 | Short-term panic thermometer; sub-20 is genuinely stretched |
| BIAS(20) | 0–15 | % deviation from the 20-day MA — how far the rubber band is pulled |
| Bollinger %B | 0–10 | Breaks below the lower band = statistically abnormal |
| 60-day drawdown | 0–15 | Total depth of the current correction |
| **Panic pattern** | **0–30** | Intraday plunge ≥2.5% + reclaim ≥50% of range + volume ≥1.2× — capitulation with buyers stepping in; the single biggest weight |
| Lower-shadow reclaim | 0–15 | How much of the day's range the close recovered |

### Stage 2 — Dual-tier entry

- **Tier A (`BuyA`, normal size):** ① panic day with score ≥65 (buy near the close), or ② score ≥55 warning, then a close above the warning day's high within 3 sessions.
- **Tier B (`BuyB`, starter size):** score ≥40 warning, then any one of three lightweight confirmations within 3 sessions: ① close above yesterday's high, ② RSI(6) crossing up through 30, ③ a green candle reclaiming ≥50% of its range.
- Duplicate signals within 3 sessions are suppressed.

### Stage 3 — Exit rules (first one hit wins, one sell per trade)

1. RSI(6) > 70 — rebound overheated
2. Close crosses above MA20 — mean reversion target reached
3. Close < lowest low of the 4 days up to entry × 0.99 — thesis wrong, cut
4. 10 trading days held — time stop; oversold bounces have a shelf life

---

## Backtest (SSE Composite daily, 2023-07 → 2026-07)

| Metric | Value |
|---|---|
| Trades | 13 (3 Tier-A, 10 Tier-B) |
| Win rate | 77% |
| Avg per trade | +1.56% |
| Profit factor (avg win / avg loss) | 2.1 |
| Best | +5.4% (2024-09-18) |
| Worst | −2.4% (2023-12-11, time stop) |
| Cumulative (non-compounded) | +20.2% vs index +18.8% |

Notable catches: the 2024-02-05 generational bottom (+3.7%), the 2025-04 tariff-crash rebound (+3.3%), the 2026-06 rebound (+2.2%).

**Caveats:** 13 trades is a small sample. The window covers the 2023H2–2024 bear and the post-2024-09 recovery, but not a 2018-style year-long grind or a 2020-COVID-style crash. In sustained bear markets expect more small losing trades — the stop-loss discipline matters more than the signal.

---

## Installation

**TDX mobile (recommended — fully phone-based):**
1. Open the SSE Composite Index (`999999`) daily chart
2. Indicators → custom indicator → create new
3. Name `CDV3_MAIN`, type "main-chart overlay", paste Formula 1
4. Name `CDV3_SUB`, type "sub-chart", paste Formula 2
5. Apply them to the main and sub panes

**THS desktop:** Tools → Formula Manager (Ctrl+F) → New → Technical Indicator → paste, test, save. Tick "save to cloud" to sync to mobile THS (mobile THS cannot edit formulas directly).

**TDX desktop / broker TDX-kernel clients:** 功能 → 公式系统 → 公式管理器 → New Technical Indicator.

Form fields: type = Technical Indicator; password protection = off; parameter table = empty. Index code: `999999` on TDX, `1A0001` on THS.

---

## Daily usage (30 seconds after close)

1. **Check the last bar's color in the sub-chart:**
   - Blue (<40): nothing to do
   - Magenta (40–55): Tier-B warning — start watching
   - Light red (55–65): Tier-A warning — watch closely
   - Red (≥65): strong-signal zone
2. **Check for text markers on the main chart:**
   - `WarnB` (blue) / `WarnA` (magenta): warning only — do nothing yet
   - `BuyB` (light red): standard entry — starter position
   - `BuyA` (red): strong entry — normal position
   - `Sell` (green): exit, no questions asked

Workflow: `BuyA`/`BuyB` appears → buy 510210 → check daily → exit on `Sell` → wait for the next one. Expect ~4 trades a year; long stretches of blue bars doing nothing are a feature, not a bug.

**Discipline notes:**
- `BuyB` is a left-side probe: half size or less. `BuyA` is the strong signal: normal size.
- Hard rule: a close below the 4-day low into entry = out, no exceptions.
- A warning that produces no entry within 3 sessions expires — don't chase.
- The indicator handles *timing*; position sizing and stops are on you.

**Tunable parameters (in the source):**
- `40 / 55 / 65` — Tier-B / Tier-A / strong thresholds
- `-2.5 / 0.5 / 1.2` — panic pattern (intraday drop % / range reclaim / volume ratio)
- `3` — confirmation window & duplicate suppression (days)
- `70 / 10` — RSI overheat exit / max holding days
- `0.99` — stop-loss buffer

---

## Formula 1 — `CDV3_MAIN` (main-chart signals)

```
MA20:MA(C,20),COLORMAGENTA;
LC:=REF(C,1);
RSI6:=SMA(MAX(C-LC,0),6,1)/SMA(ABS(C-LC),6,1)*100;
B20:=MA(C,20);
BIAS20:=(C-B20)/B20*100;
S20:=STD(C,20);
PCTB:=(C-(B20-2*S20))/(4*S20);
DD60:=(C/HHV(H,60)-1)*100;
VRT:=V/MA(V,20);
IDROP:=(L/LC-1)*100;
RECLAIM:=IF(H>L,(C-L)/(H-L),1);
SRSI:=IF(RSI6<20,15,IF(RSI6<25,12,IF(RSI6<30,8,IF(RSI6<35,4,0))));
SBIAS:=IF(BIAS20<-5,15,IF(BIAS20<-4,12,IF(BIAS20<-3,8,IF(BIAS20<-2,4,0))));
SPCT:=IF(PCTB<-0.15,10,IF(PCTB<0,8,IF(PCTB<0.1,4,0)));
SDD:=IF(DD60<-12,15,IF(DD60<-8,12,IF(DD60<-5,6,0)));
PANIC:=IDROP<=-2.5 AND RECLAIM>=0.5 AND VRT>=1.2;
SVOL:=IF(PANIC,30,IF(RECLAIM>=0.5 AND VRT>=1.2,10,IF(VRT<0.9,5,0)));
SHDW:=IF(RECLAIM>=0.7,15,IF(RECLAIM>=0.5,10,IF(RECLAIM>=0.3,4,0)));
SCORE:=SRSI+SBIAS+SPCT+SDD+SVOL+SHDW;
YJA:=SCORE>=55;
YJB:=SCORE>=40;
NYA:=BARSLAST(YJA);
NYB:=BARSLAST(YJB);
PBSIG:=PANIC AND SCORE>=65;
ACONF:=NYA>=1 AND NYA<=3 AND C>REF(H,NYA);
BCONF:=NYB>=1 AND NYB<=3 AND (C>REF(H,1) OR CROSS(RSI6,30) OR (C>O AND RECLAIM>=0.5));
BSIG0:=PBSIG OR ACONF OR BCONF;
BSIG:=BSIG0 AND COUNT(REF(BSIG0,1),3)=0;
ABUY:=PBSIG OR ACONF;
NBS:=BARSLAST(BSIG);
BLOW:=REF(LLV(L,4),NBS);
SS0:=(NBS>=1 AND NBS<=10 AND RSI6>70) OR (NBS>=1 AND NBS<=10 AND CROSS(C,MA20)) OR (NBS>=1 AND NBS<=10 AND C<BLOW*0.99) OR (NBS=10);
SSIG:=SS0 AND NBS>=1 AND COUNT(REF(SS0,1),NBS)=0;
DRAWTEXT(BSIG AND ABUY,L*0.97,'BuyA'),COLORRED;
DRAWTEXT(BSIG AND NOT(ABUY),L*0.97,'BuyB'),COLORLIRED;
DRAWTEXT(SSIG,H*1.03,'Sell'),COLORGREEN;
DRAWTEXT(YJA AND NOT(BSIG),L*0.99,'WarnA'),COLORMAGENTA;
DRAWTEXT(YJB AND NOT(YJA) AND NOT(BSIG),L*0.99,'WarnB'),COLORBLUE;
```

## Formula 2 — `CDV3_SUB` (sub-chart score)

```
LC:=REF(C,1);
RSI6:=SMA(MAX(C-LC,0),6,1)/SMA(ABS(C-LC),6,1)*100;
B20:=MA(C,20);
BIAS20:=(C-B20)/B20*100;
S20:=STD(C,20);
PCTB:=(C-(B20-2*S20))/(4*S20);
DD60:=(C/HHV(H,60)-1)*100;
VRT:=V/MA(V,20);
IDROP:=(L/LC-1)*100;
RECLAIM:=IF(H>L,(C-L)/(H-L),1);
SRSI:=IF(RSI6<20,15,IF(RSI6<25,12,IF(RSI6<30,8,IF(RSI6<35,4,0))));
SBIAS:=IF(BIAS20<-5,15,IF(BIAS20<-4,12,IF(BIAS20<-3,8,IF(BIAS20<-2,4,0))));
SPCT:=IF(PCTB<-0.15,10,IF(PCTB<0,8,IF(PCTB<0.1,4,0)));
SDD:=IF(DD60<-12,15,IF(DD60<-8,12,IF(DD60<-5,6,0)));
PANIC:=IDROP<=-2.5 AND RECLAIM>=0.5 AND VRT>=1.2;
SVOL:=IF(PANIC,30,IF(RECLAIM>=0.5 AND VRT>=1.2,10,IF(VRT<0.9,5,0)));
SHDW:=IF(RECLAIM>=0.7,15,IF(RECLAIM>=0.5,10,IF(RECLAIM>=0.3,4,0)));
SCORE:=SRSI+SBIAS+SPCT+SDD+SVOL+SHDW;
Score:SCORE,NODRAW;
STICKLINE(SCORE>=65,0,SCORE,4,0),COLORRED;
STICKLINE(SCORE>=55 AND SCORE<65,0,SCORE,4,0),COLORLIRED;
STICKLINE(SCORE>=40 AND SCORE<55,0,SCORE,4,0),COLORMAGENTA;
STICKLINE(SCORE<40,0,SCORE,4,0),COLORBLUE;
WarnB:40,COLORMAGENTA,POINTDOT;
WarnA:55,COLORLIRED,POINTDOT;
StrongSig:65,COLORRED,POINTDOT;
YJA:=SCORE>=55;
YJB:=SCORE>=40;
NYA:=BARSLAST(YJA);
NYB:=BARSLAST(YJB);
PBSIG:=PANIC AND SCORE>=65;
ACONF:=NYA>=1 AND NYA<=3 AND C>REF(H,NYA);
BCONF:=NYB>=1 AND NYB<=3 AND (C>REF(H,1) OR CROSS(RSI6,30) OR (C>O AND RECLAIM>=0.5));
BSIG0:=PBSIG OR ACONF OR BCONF;
BSIG:=BSIG0 AND COUNT(REF(BSIG0,1),3)=0;
ABUY:=PBSIG OR ACONF;
DRAWTEXT(BSIG AND ABUY,SCORE+8,'BuyA'),COLORRED;
DRAWTEXT(BSIG AND NOT(ABUY),SCORE+8,'BuyB'),COLORLIRED;
```

> The Chinese-label version of the signals (`买A`/`买B`/`卖`/`警A`/`警B`) is functionally identical — only the display text differs.

---

## Disclaimer

This is a technical-analysis tool, not investment advice. Backtested performance does not guarantee future results. Honor your stop-losses and size positions at your own risk.
