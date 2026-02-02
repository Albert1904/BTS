//+------------------------------------------------------------------+
//| Frame.mq5                                                        |
//| Frame EA v1.0 - Timeframing base chart + Cube event detection    |
//| Timeframing provides TFU/TFD/ABS markers with live tick updates  |
//| Cube provides multi-set HV/LV/HR/LR event detection             |
//+------------------------------------------------------------------+
#property strict
#property version "1.00"
#include <Trade\Trade.mqh>

//-------------------- INPUTS (SETUP) -------------------------------
input string HEADER_SETUP               = "===== SETUP =====";

// Timeframing base chart settings
input ENUM_TIMEFRAMES TimeframingTF     = PERIOD_M15;  // Timeframing analysis timeframe

input color  TFUpColor                  = clrLime;     // Timeframing Up color
input color  TFDownColor                = clrTomato;   // Timeframing Down color
input color  AbsorptionColor            = clrDodgerBlue; // Absorption color
input int    TimeframingMarkerWidth     = 2;           // Timeframing marker width

// Cube event detection settings
input bool   EnforceMarketHours         = true;
input string Session1Start              = "07:00";
input string Session1End                = "12:00";
input string Session2Start              = "12:00";
input string Session2End                = "22:00";
input bool   SkipWeekends               = true;

// Shared settings
input bool   ClearMarkerFileOnInit      = true;        // Clear all markers on init
input int    HistoricalDrawDays         = 1;           // Days of history to draw
input bool   VerboseDiagnostics         = false;

//-------------------- INPUTS (SET 1 - Timeframing Ranges) ----------
input string HEADER_SET1                = "===== SET 1 =====";
input ENUM_TIMEFRAMES SlotTF_Set1       = PERIOD_M5;
input int    RotationsDepth_Set1        = 2;
input int    MarkerSize_Set1            = 1;
input string AlertSound_Set1            = "tick.wav";
input bool   CollapseTable_Set1         = false;
input bool   DrawMarkers_Set1           = true;
input bool   PlaySound_Set1             = true;

input bool Enable_HV_HR_Set1            = true;
input bool Enable_LV_LR_Set1            = true;
input bool Enable_HV_Set1               = true;
input bool Enable_LV_Set1               = true;
input bool Enable_HR_Set1               = true;
input bool Enable_LR_Set1               = true;

input double ThresholdPct_HV_HR_Set1    = 25.0;
input double ThresholdPct_LV_LR_Set1    = 25.0;
input double ThresholdPct_HV_Set1       = 25.0;
input double ThresholdPct_LV_Set1       = 25.0;
input double ThresholdPct_HR_Set1       = 25.0;
input double ThresholdPct_LR_Set1       = 25.0;

input color Color_HV_HR_Set1            = clrDeepSkyBlue;
input color Color_LV_LR_Set1            = clrMediumPurple;
input color Color_HV_Set1               = clrMediumSeaGreen;
input color Color_LV_Set1               = clrLightYellow;
input color Color_HR_Set1               = clrSteelBlue;
input color Color_LR_Set1               = clrSandyBrown;

//-------------------- INPUTS (SET 2) -------------------------------
input string HEADER_SET2                = "===== SET 2 =====";
input ENUM_TIMEFRAMES SlotTF_Set2       = PERIOD_M5;
input int    RotationsDepth_Set2        = 2;
input int    MarkerSize_Set2            = 1;
input string AlertSound_Set2            = "tick.wav";
input bool   CollapseTable_Set2         = false;
input bool   DrawMarkers_Set2           = true;
input bool   PlaySound_Set2             = true;

input bool Enable_HV_HR_Set2            = true;
input bool Enable_LV_LR_Set2            = true;
input bool Enable_HV_Set2               = true;
input bool Enable_LV_Set2               = true;
input bool Enable_HR_Set2               = true;
input bool Enable_LR_Set2               = true;

input double ThresholdPct_HV_HR_Set2    = 25.0;
input double ThresholdPct_LV_LR_Set2    = 25.0;
input double ThresholdPct_HV_Set2       = 25.0;
input double ThresholdPct_LV_Set2       = 25.0;
input double ThresholdPct_HR_Set2       = 25.0;
input double ThresholdPct_LR_Set2       = 25.0;

input color Color_HV_HR_Set2            = clrDeepSkyBlue;
input color Color_LV_LR_Set2            = clrMediumPurple;
input color Color_HV_Set2               = clrMediumSeaGreen;
input color Color_LV_Set2               = clrLightYellow;
input color Color_HR_Set2               = clrSteelBlue;
input color Color_LR_Set2               = clrSandyBrown;

//-------------------- INPUTS (SET 3) -------------------------------
input string HEADER_SET3                = "===== SET 3 =====";
input ENUM_TIMEFRAMES SlotTF_Set3       = PERIOD_M5;
input int    RotationsDepth_Set3        = 2;
input int    MarkerSize_Set3            = 1;
input string AlertSound_Set3            = "tick.wav";
input bool   CollapseTable_Set3         = false;
input bool   DrawMarkers_Set3           = true;
input bool   PlaySound_Set3             = true;

input bool Enable_HV_HR_Set3            = true;
input bool Enable_LV_LR_Set3            = true;
input bool Enable_HV_Set3               = true;
input bool Enable_LV_Set3               = true;
input bool Enable_HR_Set3               = true;
input bool Enable_LR_Set3               = true;

input double ThresholdPct_HV_HR_Set3    = 25.0;
input double ThresholdPct_LV_LR_Set3    = 25.0;
input double ThresholdPct_HV_Set3       = 25.0;
input double ThresholdPct_LV_Set3       = 25.0;
input double ThresholdPct_HR_Set3       = 25.0;
input double ThresholdPct_LR_Set3       = 25.0;

input color Color_HV_HR_Set3            = clrDeepSkyBlue;
input color Color_LV_LR_Set3            = clrMediumPurple;
input color Color_HV_Set3               = clrMediumSeaGreen;
input color Color_LV_Set3               = clrLightYellow;
input color Color_HR_Set3               = clrSteelBlue;
input color Color_LR_Set3               = clrSandyBrown;

//-------------------- EXECUTION ------------------------------------
input string HEADER_EXECUTION           = "===== EXECUTION =====";
input string Hotkey_Buy                 = "A";
input string Hotkey_Sell                = "S";
input string Hotkey_Close               = "D";
input double TradeLotSize               = 0.10;

//-------------------- TYPES / CONSTANTS ----------------------------
struct SoundRequest { string file; int priority; int setNum; };
struct ComparisonResult { string state; double pct; };
struct TypedList { datetime t[]; };

// Timeframing range tracking
struct TimeframingRange {
   datetime startTime;
   datetime endTime;
   int direction;      // +1 = Up, -1 = Down, 0 = Absorption
   double totalVolume;
   double totalRange;  // high - low
   double highestHigh;
   double lowestLow;
};

enum CubeEventId { CE_NONE=0, CE_HV_HR=1, CE_LV_LR=2, CE_HV=3, CE_LV=4, CE_HR=5, CE_LR=6 };

#define SETS 3
#define EVENTS 7   // 0..6

//-------------------- GLOBALS (Timeframing) ------------------------
string   TF_PREFIX = "TFMKR_";
datetime g_lastProcessedClosedBarTime = 0;
string   LIVE_NAME = "TFMKR_LIVE_0";

// Live candle settings (hardcoded)
bool     DrawLiveCandle = true;
int      LiveLookbackBars = 80;

// Live bar tracking from ticks
datetime g_liveBarOpenTime = 0;
double   g_liveLow = 0.0;
double   g_liveHigh = 0.0;
bool     g_liveInitialized = false;

// Timeframing range tracking (F0=most recent closed, F1=previous, etc.)
TimeframingRange g_TFRanges[];
int g_MaxTFRanges = 10;  // Keep last 10 ranges for SET 3 analysis
bool g_NewTimeframingRangeDetected = false;

// Track current SET 3 event for one-time recoloring after timeframing scan
datetime g_CurrentSet3RangeStart = 0;
datetime g_CurrentSet3RangeEnd = 0;
color g_CurrentSet3Color = clrNONE;
bool g_NeedSet3Recolor = false;

// Track timestamps of candles that have been recolored for SET 1 events
// Using timestamps instead of object names allows protection regardless of tag (TFU/TFD/ABS)
datetime g_Set1ProtectedTimestamps[];

//-------------------- GLOBALS (Cube) -------------------------------
SoundRequest g_SoundQueue[];
datetime     g_NextSoundTime=0;

int          g_S1StartMin=0,g_S1EndMin=0,g_S2StartMin=0,g_S2EndMin=0;
bool         g_SessionsValid=false;

double       g_v[6], g_r[6]; bool g_ok[6];

datetime     g_PreloadCursor[SETS];
string       g_SlotText[SETS];
string       g_LastComment="";

datetime     g_LastSoundRotation[SETS];
int          g_Sec[SETS];

int          g_TimerIntervalSeconds=1;
int          g_TypedEventsMax=128;

TypedList    g_Typed[SETS][EVENTS];

// per-set config from inputs
ENUM_TIMEFRAMES g_TF[SETS];
int            g_Depth[SETS], g_MarkerSize[SETS];
bool           g_Collapse[SETS], g_Draw[SETS], g_Sound[SETS];
string         g_SoundFile[SETS];
bool           g_Enable[SETS][EVENTS];
double         g_Thresh[SETS][EVENTS];
color          g_Color[SETS][EVENTS];

//-------------------- SMALL HELPERS --------------------------------
int SlotSeconds(ENUM_TIMEFRAMES tf){ int s=PeriodSeconds(tf); return (s>0?s:0); }

int BarsForDays(ENUM_TIMEFRAMES tf, int days){
   int sec = PeriodSeconds(tf);
   if(sec <= 0) return 0;
   long bars = (long)days * 86400 / sec + 10;
   if(bars < 10) bars = 10;
   if(bars > 200000) bars = 200000;
   return (int)bars;
}

bool IsUpStep(const double prevHigh, const double prevLow, const double currHigh, const double currLow){
   return (currHigh > prevHigh && currLow > prevLow);
}

bool IsDownStep(const double prevHigh, const double prevLow, const double currHigh, const double currLow){
   return (currHigh < prevHigh && currLow < prevLow);
}

string Trim(const string s){
   int n=StringLen(s); if(n==0) return "";
   int i=0; while(i<n && StringGetCharacter(s,i)<=32) i++;
   int j=n-1; while(j>=i && StringGetCharacter(s,j)<=32) j--;
   return (j<i?"":StringSubstr(s,i,j-i+1));
}
string SafeShort(string s){
   StringReplace(s," ","-"); StringReplace(s,",",""); StringReplace(s,"%","p");
   StringReplace(s,":","-"); StringReplace(s,".","-"); StringReplace(s,"/","-");
   StringReplace(s,"\\","-"); StringReplace(s,"_","-");
   return s;
}
bool ParseHourMin(const string s,int &h,int &m){
   int c=StringFind(s,":"); if(c<0) return false;
   long hh=StringToInteger(Trim(StringSubstr(s,0,c)));
   long mm=StringToInteger(Trim(StringSubstr(s,c+1)));
   if(hh<0||hh>23||mm<0||mm>59) return false;
   h=(int)hh; m=(int)mm; return true;
}
bool MinutesInSession(int mins,int startMin,int endMin){
   if(startMin==endMin) return false;
   if(startMin<endMin)  return (mins>=startMin && mins<endMin);
   return (mins>=startMin || mins<endMin);
}
bool IsWithinSessions(datetime t){
   if(!g_SessionsValid) return true;
   MqlDateTime dt; TimeToStruct(t,dt);
   int mins=dt.hour*60+dt.min;
   return MinutesInSession(mins,g_S1StartMin,g_S1EndMin) || MinutesInSession(mins,g_S2StartMin,g_S2EndMin);
}
bool IsTradingAllowed(datetime t){
   if(SkipWeekends){ MqlDateTime dt; TimeToStruct(t,dt); if(dt.day_of_week==0||dt.day_of_week==6) return false; }
   if(EnforceMarketHours && g_SessionsValid) return IsWithinSessions(t);
   return true;
}
string TFLabel(ENUM_TIMEFRAMES tf){
   switch(tf){ case PERIOD_M1:return "M1"; case PERIOD_M5:return "M5"; case PERIOD_M15:return "M15"; case PERIOD_M30:return "M30"; case PERIOD_H1:return "H1"; case PERIOD_H4:return "H4"; default:return "TF"; }
}
string SetLabel(int set){ return (set==0?"SET 1":set==1?"SET 2":"SET 3"); }
string SetPrefix(int set){ return (set==0?"Set1_":set==1?"Set2_":"Set3_"); }
int    SetSoundTag(int set){ return (set==0?11:set==1?12:13); }

string EventName(int id){
   switch(id){
      case CE_HV_HR:return "HV_HR";
      case CE_LV_LR:return "LV_LR";
      case CE_HV:return "HV";
      case CE_LV:return "LV";
      case CE_HR:return "HR";
      case CE_LR:return "LR";
      default:return "NONE";
   }
}
inline bool IsH(const string s){ return s=="H"; }
inline bool IsL(const string s){ return s=="L"; }
inline bool CheckThresh(const string st,double pct,double th){ return ((st=="H"||st=="L") && MathAbs(pct)>=th); }

int EventPriority(const int eventId){
   switch(eventId){
      case CE_HV_HR:  return 90;
      case CE_LV_LR:  return 80;
      case CE_HV:     return 70;
      case CE_LV:     return 60;
      case CE_HR:     return 50;
      case CE_LR:     return 40;
      default:        return 0;
   }
}

bool EnsureRatesLoadedAround(const string sym,ENUM_TIMEFRAMES tf,datetime around,int bars=20){
   if(bars<=0) return true;
   MqlRates r[]; datetime from=around-(datetime)(bars*(long)SlotSeconds(tf));
   return (CopyRates(sym,tf,from,bars*2,r)>0);
}
bool GetExactBarHighLow(const string sym,ENUM_TIMEFRAMES tf,datetime barOpen,double &hi,double &lo){
   EnsureRatesLoadedAround(sym,tf,barOpen,20);
   int idx=iBarShift(sym,tf,barOpen,true);
   if(idx<0 || iTime(sym,tf,idx)!=barOpen) return false;
   hi=iHigh(sym,tf,idx); lo=iLow(sym,tf,idx);
   return (hi>0.0 && lo>0.0);
}

//-------------------- TIMEFRAMING MARKERS --------------------------
void DeleteTimeframingMarkers(){
   int total = ObjectsTotal(0, 0, -1);
   for(int i=total-1; i>=0; --i){
      string name = ObjectName(0, i, 0, -1);
      if(StringFind(name, TF_PREFIX) == 0)
         ObjectDelete(0, name);
   }
}

// Check if a timestamp is protected by SET 1 event
bool IsTimestampProtected(datetime candleTime){
   for(int i=0; i<ArraySize(g_Set1ProtectedTimestamps); i++){
      if(g_Set1ProtectedTimestamps[i] == candleTime) return true;
   }
   return false;
}

void DrawCandleMarker(const string tag, datetime candleTime,
                      double candleLow, double candleHigh,
                      color clr, int width){
   // This function ONLY operates on timeframing markers (TFMKR_ prefix)
   // It NEVER touches or modifies SET 2/SET 3 event markers (Set2_/Set3_ prefix)
   double lo = MathMin(candleLow, candleHigh);
   double hi = MathMax(candleLow, candleHigh);

   string name = TF_PREFIX + tag + "_" + IntegerToString((long)candleTime);
   
   // Check if this candle timestamp is protected by a SET 1 event
   bool isProtected = IsTimestampProtected(candleTime);

   // If protected, don't create or modify this marker at all
   if(isProtected) return;

   if(ObjectFind(0, name) < 0){
      if(!ObjectCreate(0, name, OBJ_TREND, 0, candleTime, lo, candleTime, hi))
         return;

      ObjectSetInteger(0, name, OBJPROP_RAY_RIGHT, false);
      ObjectSetInteger(0, name, OBJPROP_RAY_LEFT,  false);
      ObjectSetInteger(0, name, OBJPROP_SELECTABLE,false);
      ObjectSetInteger(0, name, OBJPROP_HIDDEN,    true);  // Background object (behind SET 2/3 markers)
   }
   else{
      ObjectMove(0, name, 0, candleTime, lo);
      ObjectMove(0, name, 1, candleTime, hi);
   }

   ObjectSetInteger(0, name, OBJPROP_COLOR, clr);
   ObjectSetInteger(0, name, OBJPROP_WIDTH, width);
}

void DeleteCandleMarker(const string tag, datetime candleTime){
   // Don't delete protected markers
   if(IsTimestampProtected(candleTime)) return;
   
   string name = TF_PREFIX + tag + "_" + IntegerToString((long)candleTime);
   if(ObjectFind(0, name) >= 0)
      ObjectDelete(0, name);
}

void DrawAbsorption(datetime t, double lo, double hi) { DrawCandleMarker("ABS", t, lo, hi, AbsorptionColor, TimeframingMarkerWidth); }
void DrawTFUp(datetime t, double lo, double hi)      { DrawCandleMarker("TFU", t, lo, hi, TFUpColor, TimeframingMarkerWidth); }
void DrawTFDown(datetime t, double lo, double hi)    { DrawCandleMarker("TFD", t, lo, hi, TFDownColor, TimeframingMarkerWidth); }

void DeleteAbsorption(datetime t) { DeleteCandleMarker("ABS", t); }
void DeleteTFUp(datetime t)       { DeleteCandleMarker("TFU", t); }
void DeleteTFDown(datetime t)     { DeleteCandleMarker("TFD", t); }

void DeleteLiveMarker(){
   ObjectDelete(0, LIVE_NAME);
}

void RecolorTimeframingRange(datetime startTime, datetime endTime, color newColor){
   int total = ObjectsTotal(0, 0, -1);
   int recolored = 0;
   
   for(int i=0; i<total; ++i){
      string name = ObjectName(0, i, 0, -1);
      if(StringFind(name, TF_PREFIX) != 0) continue;
      if(StringFind(name, "LIVE") >= 0) continue; // Skip live marker
      
      // Get the actual bar time from the object name (it's encoded as timestamp)
      // Format: TFMKR_TAG_timestamp
      int lastUnderscore = StringFind(name, "_", StringLen(TF_PREFIX));
      if(lastUnderscore >= 0){
         string timeStr = StringSubstr(name, lastUnderscore + 1);
         datetime objTime = (datetime)StringToInteger(timeStr);
         
         if(objTime >= startTime && objTime <= endTime){
            ObjectSetInteger(0, name, OBJPROP_COLOR, newColor);
            
            // Add timestamp to protection array if not already there
            if(!IsTimestampProtected(objTime)){
               int n = ArraySize(g_Set1ProtectedTimestamps);
               ArrayResize(g_Set1ProtectedTimestamps, n + 1);
               g_Set1ProtectedTimestamps[n] = objTime;
            }
            
            recolored++;
         }
      }
   }
}

void DrawOrUpdateLiveMarker(datetime t, double lo, double hi, color clr){
   if(ObjectFind(0, LIVE_NAME) < 0){
      if(!ObjectCreate(0, LIVE_NAME, OBJ_TREND, 0, t, lo, t, hi))
         return;

      ObjectSetInteger(0, LIVE_NAME, OBJPROP_RAY_RIGHT, false);
      ObjectSetInteger(0, LIVE_NAME, OBJPROP_RAY_LEFT,  false);
      ObjectSetInteger(0, LIVE_NAME, OBJPROP_HIDDEN, false);
      ObjectSetInteger(0, LIVE_NAME, OBJPROP_BACK,   false);
      ObjectSetInteger(0, LIVE_NAME, OBJPROP_ZORDER, 999);
      ObjectSetInteger(0, LIVE_NAME, OBJPROP_SELECTABLE, false);
   }
   else{
      ObjectMove(0, LIVE_NAME, 0, t, lo);
      ObjectMove(0, LIVE_NAME, 1, t, hi);
   }

   ObjectSetInteger(0, LIVE_NAME, OBJPROP_COLOR, clr);
   ObjectSetInteger(0, LIVE_NAME, OBJPROP_WIDTH, TimeframingMarkerWidth);
}

int GetLiveState(){
   int need = MathMax(LiveLookbackBars, 12);

   MqlRates r[];
   ArraySetAsSeries(r, true);

   int copied = CopyRates(_Symbol, TimeframingTF, 0, need, r);
   if(copied < 6) return 0;

   bool bar0UpStep   = IsUpStep(r[1].high, r[1].low, g_liveHigh, g_liveLow);
   bool bar0DownStep = IsDownStep(r[1].high, r[1].low, g_liveHigh, g_liveLow);

   int oldest = copied - 1;
   int state = 0;

   for(int idx = oldest; idx >= 1; --idx){
      int prev = idx + 1;
      if(prev > oldest) continue;

      bool upStep   = IsUpStep(r[prev].high, r[prev].low, r[idx].high, r[idx].low);
      bool downStep = IsDownStep(r[prev].high, r[prev].low, r[idx].high, r[idx].low);

      if(state == 0){
         int s = idx + 2, m = idx + 1, e = idx;
         if(s <= oldest){
            bool up1 = IsUpStep(r[s].high, r[s].low, r[m].high, r[m].low);
            bool up2 = IsUpStep(r[m].high, r[m].low, r[e].high, r[e].low);

            bool dn1 = IsDownStep(r[s].high, r[s].low, r[m].high, r[m].low);
            bool dn2 = IsDownStep(r[m].high, r[m].low, r[e].high, r[e].low);

            if(up1 && up2) state = +1;
            else if(dn1 && dn2) state = -1;
         }
         continue;
      }

      if(state == +1 && !upStep) state = 0;
      if(state == -1 && !downStep) state = 0;
   }

   if(state == +1 && bar0UpStep)   return +1;
   if(state == -1 && bar0DownStep) return -1;
   return 0;
}

void UpdateLiveFromTick(){
   if(!DrawLiveCandle)
      return;

   datetime barOpen = iTime(_Symbol, TimeframingTF, 0);
   if(barOpen == 0)
      return;

   MqlTick tk;
   if(!SymbolInfoTick(_Symbol, tk))
      return;

   double price = tk.last;
   if(price <= 0.0)
      price = (tk.bid + tk.ask) * 0.5;

   if(!g_liveInitialized || barOpen != g_liveBarOpenTime){
      g_liveBarOpenTime = barOpen;
      g_liveLow = price;
      g_liveHigh = price;
      g_liveInitialized = true;
   }
   else{
      if(price < g_liveLow)  g_liveLow = price;
      if(price > g_liveHigh) g_liveHigh = price;
   }

   int liveState = GetLiveState();
   color liveClr = AbsorptionColor;
   if(liveState == +1) liveClr = TFUpColor;
   else if(liveState == -1) liveClr = TFDownColor;

   DrawOrUpdateLiveMarker(g_liveBarOpenTime, g_liveLow, g_liveHigh, liveClr);
}

void ScanAndDrawTimeframing(){
   int barsToLoad = BarsForDays(TimeframingTF, HistoricalDrawDays);
   if(barsToLoad < 10){
      if(VerboseDiagnostics) Print("ScanAndDrawTimeframing: barsToLoad too small: ", barsToLoad);
      return;
   }

   MqlRates rates[];
   ArraySetAsSeries(rates, true);

   int copied = CopyRates(_Symbol, TimeframingTF, 0, barsToLoad, rates);
   if(copied < 6){
      if(VerboseDiagnostics) Print("ScanAndDrawTimeframing: Failed to copy rates. Requested: ", barsToLoad, " Copied: ", copied);
      return;
   }

   int oldest = copied - 1;
   if(oldest <= 3) return;

   // Remember old range count to detect new ranges
   int oldRangeCount = ArraySize(g_TFRanges);
   
   // Clear and prepare timeframing ranges array
   ArrayResize(g_TFRanges, 0);
   
   int state = 0;
   datetime rangeStart = 0;
   double rangeHighest = 0.0;
   double rangeLowest = DBL_MAX;
   double rangeVolume = 0.0;

   for(int idx = oldest; idx >= 1; --idx){
      int prev = idx + 1;
      if(prev > oldest) continue;

      double prevHigh = rates[prev].high;
      double prevLow  = rates[prev].low;
      double currHigh = rates[idx].high;
      double currLow  = rates[idx].low;

      bool upStep   = IsUpStep(prevHigh, prevLow, currHigh, currLow);
      bool downStep = IsDownStep(prevHigh, prevLow, currHigh, currLow);

      bool inRange = false;
      if(state == +1) inRange = upStep;
      else if(state == -1) inRange = downStep;

      if(inRange){
         if(state == +1) DrawTFUp(rates[idx].time, currLow, currHigh);
         else            DrawTFDown(rates[idx].time, currLow, currHigh);
         
         // Accumulate range statistics
         if(currHigh > rangeHighest) rangeHighest = currHigh;
         if(currLow < rangeLowest) rangeLowest = currLow;
         rangeVolume += (double)rates[idx].tick_volume;
      }
      else{
         DrawAbsorption(rates[idx].time, currLow, currHigh);
         
         // If we were in a range, save it before transitioning to absorption
         if(state != 0 && rangeStart > 0){
            TimeframingRange range;
            range.startTime = rangeStart;
            range.endTime = rates[prev].time;
            range.direction = state;
            range.totalVolume = rangeVolume;
            range.highestHigh = rangeHighest;
            range.lowestLow = rangeLowest;
            range.totalRange = rangeHighest - rangeLowest;
            
            int n = ArraySize(g_TFRanges);
            ArrayResize(g_TFRanges, n + 1);
            g_TFRanges[n] = range;
            
            // Reset range tracking
            rangeStart = 0;
            rangeHighest = 0.0;
            rangeLowest = DBL_MAX;
            rangeVolume = 0.0;
         }
      }

      if(state == 0){
         int s = idx + 2, m = idx + 1, e = idx;
         if(s <= oldest){
            bool up1 = IsUpStep(rates[s].high, rates[s].low, rates[m].high, rates[m].low);
            bool up2 = IsUpStep(rates[m].high, rates[m].low, rates[e].high, rates[e].low);

            bool dn1 = IsDownStep(rates[s].high, rates[s].low, rates[m].high, rates[m].low);
            bool dn2 = IsDownStep(rates[m].high, rates[m].low, rates[e].high, rates[e].low);

            if(up1 && up2){
               state = +1;

               DrawTFUp(rates[s].time, rates[s].low, rates[s].high);
               DrawTFUp(rates[m].time, rates[m].low, rates[m].high);
               DrawTFUp(rates[e].time, rates[e].low, rates[e].high);
               
               // Initialize range tracking
               rangeStart = rates[s].time;
               rangeHighest = MathMax(rates[s].high, MathMax(rates[m].high, rates[e].high));
               rangeLowest = MathMin(rates[s].low, MathMin(rates[m].low, rates[e].low));
               rangeVolume = (double)(rates[s].tick_volume + rates[m].tick_volume + rates[e].tick_volume);
            }
            else if(dn1 && dn2){
               state = -1;

               DrawTFDown(rates[s].time, rates[s].low, rates[s].high);
               DrawTFDown(rates[m].time, rates[m].low, rates[m].high);
               DrawTFDown(rates[e].time, rates[e].low, rates[e].high);
               
               // Initialize range tracking
               rangeStart = rates[s].time;
               rangeHighest = MathMax(rates[s].high, MathMax(rates[m].high, rates[e].high));
               rangeLowest = MathMin(rates[s].low, MathMin(rates[m].low, rates[e].low));
               rangeVolume = (double)(rates[s].tick_volume + rates[m].tick_volume + rates[e].tick_volume);
            }
         }
         continue;
      }

      if(state == +1 && !upStep){
         // Save the up range before transitioning
         if(rangeStart > 0){
            TimeframingRange range;
            range.startTime = rangeStart;
            range.endTime = rates[prev].time;
            range.direction = +1;
            range.totalVolume = rangeVolume;
            range.highestHigh = rangeHighest;
            range.lowestLow = rangeLowest;
            range.totalRange = rangeHighest - rangeLowest;
            
            int n = ArraySize(g_TFRanges);
            ArrayResize(g_TFRanges, n + 1);
            g_TFRanges[n] = range;
         }
         state = 0;
         rangeStart = 0;
         rangeHighest = 0.0;
         rangeLowest = DBL_MAX;
         rangeVolume = 0.0;
      }
      if(state == -1 && !downStep){
         // Save the down range before transitioning
         if(rangeStart > 0){
            TimeframingRange range;
            range.startTime = rangeStart;
            range.endTime = rates[prev].time;
            range.direction = -1;
            range.totalVolume = rangeVolume;
            range.highestHigh = rangeHighest;
            range.lowestLow = rangeLowest;
            range.totalRange = rangeHighest - rangeLowest;
            
            int n = ArraySize(g_TFRanges);
            ArrayResize(g_TFRanges, n + 1);
            g_TFRanges[n] = range;
         }
         state = 0;
         rangeStart = 0;
         rangeHighest = 0.0;
         rangeLowest = DBL_MAX;
         rangeVolume = 0.0;
      }
   }
   
   // Save the final range if still in one
   if(state != 0 && rangeStart > 0){
      TimeframingRange range;
      range.startTime = rangeStart;
      range.endTime = rates[1].time;
      range.direction = state;
      range.totalVolume = rangeVolume;
      range.highestHigh = rangeHighest;
      range.lowestLow = rangeLowest;
      range.totalRange = rangeHighest - rangeLowest;
      
      int n = ArraySize(g_TFRanges);
      ArrayResize(g_TFRanges, n + 1);
      g_TFRanges[n] = range;
   }
   
   // Keep only the most recent ranges (reverse order: index 0 = most recent)
   int total = ArraySize(g_TFRanges);
   if(total > g_MaxTFRanges){
      TimeframingRange temp[];
      ArrayResize(temp, g_MaxTFRanges);
      for(int i=0; i<g_MaxTFRanges; i++){
         temp[i] = g_TFRanges[total - g_MaxTFRanges + i];
      }
      ArrayCopy(g_TFRanges, temp);
   }
   
   // Set flag if we have a new range (or initial load)
   int newRangeCount = ArraySize(g_TFRanges);
   if(newRangeCount != oldRangeCount || oldRangeCount == 0){
      g_NewTimeframingRangeDetected = true;
   }
   
   if(VerboseDiagnostics) Print("ScanAndDrawTimeframing: Detected ", newRangeCount, " ranges. Old: ", oldRangeCount, " New detected: ", g_NewTimeframingRangeDetected);
}

//-------------------- CUBE CONFIG LOAD -----------------------------
void LoadSetConfig(){
   // NOTE: Input-to-SET mapping corrected to fix user input routing issues.
   // After debugging, the correct mapping is:
   // - SET 1 uses Set1 inputs
   // - SET 2 uses Set3 inputs (not Set2!)
   // - SET 3 uses Set2 inputs (not Set3!)
   // This specific mapping ensures user changes appear in the correct SET printouts.
   
   // SET 1: Map from Set1 inputs
   g_TF[0]=SlotTF_Set1; g_Depth[0]=RotationsDepth_Set1; g_MarkerSize[0]=MarkerSize_Set1;
   g_Collapse[0]=CollapseTable_Set1; g_Draw[0]=DrawMarkers_Set1; g_Sound[0]=PlaySound_Set1; g_SoundFile[0]=AlertSound_Set1;

   g_Enable[0][CE_HV_HR]=Enable_HV_HR_Set1; g_Enable[0][CE_LV_LR]=Enable_LV_LR_Set1; g_Enable[0][CE_HV]=Enable_HV_Set1;
   g_Enable[0][CE_LV]=Enable_LV_Set1; g_Enable[0][CE_HR]=Enable_HR_Set1; g_Enable[0][CE_LR]=Enable_LR_Set1;

   g_Thresh[0][CE_HV_HR]=ThresholdPct_HV_HR_Set1; g_Thresh[0][CE_LV_LR]=ThresholdPct_LV_LR_Set1; g_Thresh[0][CE_HV]=ThresholdPct_HV_Set1;
   g_Thresh[0][CE_LV]=ThresholdPct_LV_Set1; g_Thresh[0][CE_HR]=ThresholdPct_HR_Set1; g_Thresh[0][CE_LR]=ThresholdPct_LR_Set1;

   g_Color[0][CE_HV_HR]=Color_HV_HR_Set1; g_Color[0][CE_LV_LR]=Color_LV_LR_Set1; g_Color[0][CE_HV]=Color_HV_Set1;
   g_Color[0][CE_LV]=Color_LV_Set1; g_Color[0][CE_HR]=Color_HR_Set1; g_Color[0][CE_LR]=Color_LR_Set1;

   // SET 2: Map from Set3 inputs  
   g_TF[1]=SlotTF_Set3; g_Depth[1]=RotationsDepth_Set3; g_MarkerSize[1]=MarkerSize_Set3;
   g_Collapse[1]=CollapseTable_Set3; g_Draw[1]=DrawMarkers_Set3; g_Sound[1]=PlaySound_Set3; g_SoundFile[1]=AlertSound_Set3;

   g_Enable[1][CE_HV_HR]=Enable_HV_HR_Set3; g_Enable[1][CE_LV_LR]=Enable_LV_LR_Set3; g_Enable[1][CE_HV]=Enable_HV_Set3;
   g_Enable[1][CE_LV]=Enable_LV_Set3; g_Enable[1][CE_HR]=Enable_HR_Set3; g_Enable[1][CE_LR]=Enable_LR_Set3;

   g_Thresh[1][CE_HV_HR]=ThresholdPct_HV_HR_Set3; g_Thresh[1][CE_LV_LR]=ThresholdPct_LV_LR_Set3; g_Thresh[1][CE_HV]=ThresholdPct_HV_Set3;
   g_Thresh[1][CE_LV]=ThresholdPct_LV_Set3; g_Thresh[1][CE_HR]=ThresholdPct_HR_Set3; g_Thresh[1][CE_LR]=ThresholdPct_LR_Set3;

   g_Color[1][CE_HV_HR]=Color_HV_HR_Set3; g_Color[1][CE_LV_LR]=Color_LV_LR_Set3; g_Color[1][CE_HV]=Color_HV_Set3;
   g_Color[1][CE_LV]=Color_LV_Set3; g_Color[1][CE_HR]=Color_HR_Set3; g_Color[1][CE_LR]=Color_LR_Set3;

   // SET 3: Map from Set2 inputs
   g_TF[2]=SlotTF_Set2; g_Depth[2]=RotationsDepth_Set2; g_MarkerSize[2]=MarkerSize_Set2;
   g_Collapse[2]=CollapseTable_Set2; g_Draw[2]=DrawMarkers_Set2; g_Sound[2]=PlaySound_Set2; g_SoundFile[2]=AlertSound_Set2;

   g_Enable[2][CE_HV_HR]=Enable_HV_HR_Set2; g_Enable[2][CE_LV_LR]=Enable_LV_LR_Set2; g_Enable[2][CE_HV]=Enable_HV_Set2;
   g_Enable[2][CE_LV]=Enable_LV_Set2; g_Enable[2][CE_HR]=Enable_HR_Set2; g_Enable[2][CE_LR]=Enable_LR_Set2;

   g_Thresh[2][CE_HV_HR]=ThresholdPct_HV_HR_Set2; g_Thresh[2][CE_LV_LR]=ThresholdPct_LV_LR_Set2; g_Thresh[2][CE_HV]=ThresholdPct_HV_Set2;
   g_Thresh[2][CE_LV]=ThresholdPct_LV_Set2; g_Thresh[2][CE_HR]=ThresholdPct_HR_Set2; g_Thresh[2][CE_LR]=ThresholdPct_LR_Set2;

   g_Color[2][CE_HV_HR]=Color_HV_HR_Set2; g_Color[2][CE_LV_LR]=Color_LV_LR_Set2; g_Color[2][CE_HV]=Color_HV_Set2;
   g_Color[2][CE_LV]=Color_LV_Set2; g_Color[2][CE_HR]=Color_HR_Set2; g_Color[2][CE_LR]=Color_LR_Set2;
}

string EnabledSummary(int set){
   string s="";
   for(int id=1; id<EVENTS; id++) if(g_Enable[set][id]) s+=EventName(id)+" ";
   s=Trim(s); return (StringLen(s)?s:"NONE");
}

//-------------------- TYPED EVENTS ---------------------------------
void ClearTypedEvents_All(){ for(int s=0;s<SETS;s++) for(int e=0;e<EVENTS;e++) ArrayResize(g_Typed[s][e].t,0); }
void PushTypedEvent(const int set,const int eventId,const datetime t){
   if(set<0||set>=SETS||eventId<=0||eventId>=EVENTS||t<=0) return;
   int n=ArraySize(g_Typed[set][eventId].t);
   if(n>0 && g_Typed[set][eventId].t[0]==t) return;
   ArrayResize(g_Typed[set][eventId].t,n+1);
   for(int i=n;i>0;i--) g_Typed[set][eventId].t[i]=g_Typed[set][eventId].t[i-1];
   g_Typed[set][eventId].t[0]=t;
   if(ArraySize(g_Typed[set][eventId].t)>g_TypedEventsMax) ArrayResize(g_Typed[set][eventId].t,g_TypedEventsMax);
}

//-------------------- SLOT REPORT + DETECTION -----------------------
bool BuildSlotReport_FromTimeframingRanges(const int set,const datetime slotEnd,const int depth,ComparisonResult &cv,ComparisonResult &cr,int maxRangeIndex=-1){
   // SET 3 uses timeframing ranges (F0, F1, F2...) instead of time slots
   // maxRangeIndex allows historical processing without array swapping
   int numRanges = ArraySize(g_TFRanges);
   if(numRanges < 1) return false;
   
   // If maxRangeIndex specified, use it as the effective array size
   if(maxRangeIndex > 0 && maxRangeIndex < numRanges) numRanges = maxRangeIndex;
   
   int regs = depth + 1;
   if(regs > 6) regs = 6;
   if(regs > numRanges) regs = numRanges;
   
   for(int i=0; i<6; i++){ g_v[i]=0; g_r[i]=0; g_ok[i]=false; }
   
   // F0 = most recent closed range, F1 = previous, etc.
   // g_TFRanges is already stored with most recent at the end
   for(int i=0; i<regs; i++){
      int idx = numRanges - 1 - i; // Access from most recent backwards
      if(idx < 0) break;
      
      g_v[i] = g_TFRanges[idx].totalVolume;
      g_r[i] = g_TFRanges[idx].totalRange;
      g_ok[i] = (g_v[i] > 0);
   }
   
   if(!g_ok[0]) return false;
   
   // Compare F0 against F1..FN for VOLUME
   double vBase = g_v[0], vMx = -DBL_MAX, vMn = DBL_MAX;
   for(int i=1; i<regs; i++){
      if(g_v[i] > vMx) vMx = g_v[i];
      if(g_v[i] < vMn) vMn = g_v[i];
   }
   if(vBase > vMx && vMx > 0.0){ cv.state="H"; cv.pct=(vBase-vMx)/vMx*100.0; }
   else if(vBase < vMn && vMn > 0.0){ cv.state="L"; cv.pct=((vMn-vBase)/vMn*100.0)*-1.0; }
   else { cv.state="B"; cv.pct=0.0; }
   
   // Compare F0 against F1..FN for RANGE
   double rBase = g_r[0], rMx = -DBL_MAX, rMn = DBL_MAX;
   for(int i=1; i<regs; i++){
      if(g_r[i] > rMx) rMx = g_r[i];
      if(g_r[i] < rMn) rMn = g_r[i];
   }
   if(rBase > rMx && rMx > 0.0){ cr.state="H"; cr.pct=(rBase-rMx)/rMx*100.0; }
   else if(rBase < rMn && rMn > 0.0){ cr.state="L"; cr.pct=((rMn-rBase)/rMn*100.0)*-1.0; }
   else { cr.state="B"; cr.pct=0.0; }
   
   return true;
}

bool BuildSlotReport_Generic(const int set,const datetime slotEnd,const int depth,ComparisonResult &cv,ComparisonResult &cr){
   ENUM_TIMEFRAMES tf=g_TF[set];
   int sec=SlotSeconds(tf); if(sec<=0) return false;

   int regs=depth+1; if(regs>6) regs=6;
   for(int i=0;i<regs;i++){ g_v[i]=0; g_r[i]=0; g_ok[i]=false; }

   for(int i=0;i<regs;i++){
      datetime regionStart=(slotEnd-(datetime)(i*(long)sec))-(datetime)sec;
      int idx=iBarShift(_Symbol,tf,regionStart,false);
      if(idx<0) idx=iBarShift(_Symbol,tf,regionStart+1,true);
      if(idx<0) return false;

      long vol=(long)iVolume(_Symbol,tf,idx);
      double hi=iHigh(_Symbol,tf,idx), lo=iLow(_Symbol,tf,idx);
      g_v[i]=(double)vol;
      g_r[i]=(vol>0 && hi>0.0 && lo>0.0)?(hi-lo):0.0;
      g_ok[i]=(vol>0);
   }

   double base=g_v[0], mx=-DBL_MAX, mn=DBL_MAX;
   for(int i=1;i<regs;i++){ if(g_v[i]>mx) mx=g_v[i]; if(g_v[i]<mn) mn=g_v[i]; }
   if(base>mx && mx>0.0){ cv.state="H"; cv.pct=(base-mx)/mx*100.0; }
   else if(base<mn && mn>0.0){ cv.state="L"; cv.pct=((mn-base)/mn*100.0)*-1.0; }
   else { cv.state="B"; cv.pct=0.0; }

   base=g_r[0]; mx=-DBL_MAX; mn=DBL_MAX;
   for(int i=1;i<regs;i++){ if(g_r[i]>mx) mx=g_r[i]; if(g_r[i]<mn) mn=g_r[i]; }
   if(base>mx && mx>0.0){ cr.state="H"; cr.pct=(base-mx)/mx*100.0; }
   else if(base<mn && mn>0.0){ cr.state="L"; cr.pct=((mn-base)/mn*100.0)*-1.0; }
   else { cr.state="B"; cr.pct=0.0; }

   return true;
}

int DetectEvent(const int set,const ComparisonResult &cv,const ComparisonResult &cr){
   if(g_Enable[set][CE_HV_HR] && IsH(cv.state) && IsH(cr.state) &&
      CheckThresh(cv.state,cv.pct,g_Thresh[set][CE_HV_HR]) && CheckThresh(cr.state,cr.pct,g_Thresh[set][CE_HV_HR])) return CE_HV_HR;

   if(g_Enable[set][CE_LV_LR] && IsL(cv.state) && IsL(cr.state) &&
      CheckThresh(cv.state,cv.pct,g_Thresh[set][CE_LV_LR]) && CheckThresh(cr.state,cr.pct,g_Thresh[set][CE_LV_LR])) return CE_LV_LR;

   if(g_Enable[set][CE_HV] && IsH(cv.state) && CheckThresh(cv.state,cv.pct,g_Thresh[set][CE_HV])) return CE_HV;
   if(g_Enable[set][CE_LV] && IsL(cv.state) && CheckThresh(cv.state,cv.pct,g_Thresh[set][CE_LV])) return CE_LV;
   if(g_Enable[set][CE_HR] && IsH(cr.state) && CheckThresh(cr.state,cr.pct,g_Thresh[set][CE_HR])) return CE_HR;
   if(g_Enable[set][CE_LR] && IsL(cr.state) && CheckThresh(cr.state,cr.pct,g_Thresh[set][CE_LR])) return CE_LR;

   return CE_NONE;
}

bool BuildSlotTextAndDetect(const int set,const datetime slotEnd,int &eventId,ComparisonResult &cv,ComparisonResult &cr){
   eventId=CE_NONE;
   
   bool success = false;
   // SET 1 uses timeframing ranges instead of time slots
   if(set == 0){
      success = BuildSlotReport_FromTimeframingRanges(set, slotEnd, g_Depth[set], cv, cr);
   }
   else{
      success = BuildSlotReport_Generic(set, slotEnd, g_Depth[set], cv, cr);
   }
   
   if(!success){
      string nodata=SetLabel(set)+": "+EnabledSummary(set)+"\nNO DATA\n";
      StringToUpper(nodata); g_SlotText[set]=nodata;
      return false;
   }

   eventId=DetectEvent(set,cv,cr);
   string ev=(eventId==CE_NONE?"":EventName(eventId));

   string out=SetLabel(set)+": "+EnabledSummary(set)+"\n";
   if(set == 0){
      // SET 1 shows timeframing range analysis
      out+="TF RANGES DE: "+IntegerToString(g_Depth[set])+"\n";
      if(!g_Collapse[set]){
         int regs=g_Depth[set]+1; if(regs>6) regs=6;
         for(int i=0;i<regs;i++) out+=StringFormat("F%d V%.0f R%."+IntegerToString(_Digits)+"f\n",i,g_v[i],g_r[i]);
      }
   }
   else{
      // SET 2 & 3 show time slot analysis
      out+="TF: "+TFLabel(g_TF[set])+" DE: "+IntegerToString(g_Depth[set])+"\n";
      if(!g_Collapse[set]){
         int regs=g_Depth[set]+1; if(regs>6) regs=6;
         for(int i=0;i<regs;i++) out+=StringFormat("R%d V%.0f R%."+IntegerToString(_Digits)+"f\n",i,g_v[i],g_r[i]);
      }
   }
   out+=StringFormat("V%s %.0f%% R%s %.0f%%\n",cv.state,MathRound(cv.pct),cr.state,MathRound(cr.pct));
   out+="EV: "+ev+"\n";
   StringToUpper(out); g_SlotText[set]=out;
   return true;
}

//-------------------- DRAWING + SOUND -------------------------------
// DrawVerticalLine: Creates SET 2 and SET 3 event markers (Set2_/Set3_ prefix)
// These markers are NEVER deleted or modified during timeframing operations
// They use a separate object namespace from timeframing markers (TFMKR_ prefix)
// Once created, they persist until EA removal or ClearMarkerFileOnInit
bool DrawVerticalLine(const string name,datetime t1,datetime t2,double low,double high,color clr,int width){
   if(ObjectFind(0,name)>=0) return false;  // Marker already exists, never overwrite
   datetime center=t1+(t2-t1)/2;
   if(!ObjectCreate(0,name,OBJ_TREND,0,center,low,center,high)) return false;
   ObjectSetInteger(0,name,OBJPROP_COLOR,clr);
   ObjectSetInteger(0,name,OBJPROP_WIDTH,width);
   ObjectSetInteger(0,name,OBJPROP_RAY,false);
   ObjectSetInteger(0,name,OBJPROP_BACK,false);  // Foreground object (visible on top)
   ObjectSetInteger(0,name,OBJPROP_ZORDER,100);  // High z-order: always on top of timeframing markers
   return true;
}

bool IsSoundAlreadyQueued(const string file,int setNum){
   for(int i=0,n=ArraySize(g_SoundQueue);i<n;i++) if(g_SoundQueue[i].file==file && g_SoundQueue[i].setNum==setNum) return true;
   return false;
}
void EnqueueUniqueSound(const string file,int prio,bool enabled,int setNum){
   if(!enabled || StringLen(Trim(file))==0 || IsSoundAlreadyQueued(file,setNum)) return;
   int n=ArraySize(g_SoundQueue); ArrayResize(g_SoundQueue,n+1);
   g_SoundQueue[n].file=file; g_SoundQueue[n].priority=prio; g_SoundQueue[n].setNum=setNum;
}
void ProcessSoundQueue(){
   int n=ArraySize(g_SoundQueue); if(n==0 || g_NextSoundTime>TimeCurrent()) return;
   string f=g_SoundQueue[0].file; if(StringLen(Trim(f))>0) PlaySound(f);
   for(int i=1;i<n;i++) g_SoundQueue[i-1]=g_SoundQueue[i];
   ArrayResize(g_SoundQueue,n-1);
   g_NextSoundTime=TimeCurrent()+2;
}

void DrawAndNotifyWinner(const int set,const int eventId,const ComparisonResult &cv,const ComparisonResult &cr,const datetime slotStart,const datetime slotEnd){
   if(eventId==CE_NONE) return;

   PushTypedEvent(set,eventId,slotEnd); // winner only
   
   // SET 1: Recolor the timeframing range markers instead of drawing new markers
   if(set == 0){
      int numRanges = ArraySize(g_TFRanges);
      if(numRanges > 0){
         // Get F0 (most recent closed range)
         TimeframingRange range = g_TFRanges[numRanges - 1];
         
         // Store current event range for one-time recoloring after next timeframing scan
         g_CurrentSet3RangeStart = range.startTime;
         g_CurrentSet3RangeEnd = range.endTime;
         g_CurrentSet3Color = g_Color[set][eventId];
         g_NeedSet3Recolor = true;
         
         // Immediate recolor for this event only
         RecolorTimeframingRange(range.startTime, range.endTime, g_Color[set][eventId]);
      }
      
      if(g_Sound[set] && g_LastSoundRotation[set]!=slotEnd){
         g_LastSoundRotation[set]=slotEnd;
         EnqueueUniqueSound(g_SoundFile[set],5,true,SetSoundTag(set));
      }
      return;
   }
   
   // SET 2 & 3: Draw vertical markers as before
   if(!g_Draw[set]) return;

   double hi=0,lo=0;
   if(!GetExactBarHighLow(_Symbol,g_TF[set],slotStart,hi,lo)) return;

   int vol=(int)MathRound(g_v[0]), rng=(int)MathRound(g_r[0]);
   int cvp=(int)MathRound(cv.pct), crp=(int)MathRound(cr.pct);
   string shortRaw=StringFormat("V%d R%d, V%s%d%% R%s%d%%",vol,rng,cv.state,cvp,cr.state,crp);
   string shortSafe=SafeShort(shortRaw);

   string tstr=TimeToString(slotEnd,TIME_DATE|TIME_SECONDS);
   StringReplace(tstr," ","_"); StringReplace(tstr,":","-");

   string obj=SetPrefix(set)+EventName(eventId)+"_"+shortSafe+"_"+tstr;
   bool created=DrawVerticalLine(obj,slotStart,slotEnd,lo,hi,g_Color[set][eventId],g_MarkerSize[set]);
   if(created) ObjectSetString(0,obj,OBJPROP_TEXT,shortRaw);

   if(created && g_Sound[set] && g_LastSoundRotation[set]!=slotEnd){
      g_LastSoundRotation[set]=slotEnd;
      EnqueueUniqueSound(g_SoundFile[set],5,true,SetSoundTag(set));
   }
}

//-------------------- PRELOAD / INIT / TIMER ------------------------
void ProcessSet1HistoricalRanges(){
   // Process all historical timeframing ranges for SET 1 event detection
   int numRanges = ArraySize(g_TFRanges);
   if(VerboseDiagnostics) Print("ProcessSet1HistoricalRanges: Starting with ", numRanges, " ranges");
   if(numRanges == 0){
      if(VerboseDiagnostics) Print("ProcessSet1HistoricalRanges: No ranges found, exiting");
      return;
   }
   
   // Safety limit to prevent runaway loops
   int maxRangesToProcess = MathMin(numRanges, 1000);
   if(numRanges > maxRangesToProcess){
      Print("ProcessSet1HistoricalRanges: WARNING - Limiting processing to ", maxRangesToProcess, " of ", numRanges, " ranges");
   }
   
   // Store detected events for batch processing
   struct RangeEvent {
      datetime startTime;
      datetime endTime;
      color clr;
      datetime slotEnd;
      int eventId;
   };
   RangeEvent events[];
   int eventsDetected = 0;
   
   // Process each range from oldest to newest (NO ARRAY SWAPPING)
   for(int rangeIdx = g_Depth[0]; rangeIdx < maxRangesToProcess; rangeIdx++){
      // Detect event for this range using maxRangeIndex parameter
      int eventId = CE_NONE;
      ComparisonResult cv, cr;
      datetime slotEnd = g_TFRanges[rangeIdx].endTime;
      
      // Pass rangeIdx+1 as maxRangeIndex to treat g_TFRanges[rangeIdx] as F0
      bool hasEvent = BuildSlotReport_FromTimeframingRanges(0, slotEnd, g_Depth[0], cv, cr, rangeIdx+1);
      if(hasEvent) eventId = DetectEvent(0, cv, cr);
      
      // If event detected, store for batch processing
      if(hasEvent && eventId != CE_NONE){
         int n = ArraySize(events);
         ArrayResize(events, n + 1);
         events[n].startTime = g_TFRanges[rangeIdx].startTime;
         events[n].endTime = g_TFRanges[rangeIdx].endTime;
         events[n].clr = g_Color[0][eventId];
         events[n].slotEnd = slotEnd;
         events[n].eventId = eventId;
         eventsDetected++;
         
         // Play sound for historical event if enabled (only most recent)
         if(g_Sound[0] && rangeIdx == numRanges-1){
            EnqueueUniqueSound(g_SoundFile[0], 5, true, SetSoundTag(0));
         }
      }
   }
   
   // Batch process all detected events with SINGLE object scan
   int totalObjects = ObjectsTotal(0, 0, -1);
   int recoloredCount = 0;
   
   // Clear and rebuild protection array for SET 1 event markers
   ArrayResize(g_Set1ProtectedTimestamps, 0);
   
   // Single pass through all objects to recolor multiple ranges
   for(int objIdx=0; objIdx<totalObjects; objIdx++){
      string objName = ObjectName(0, objIdx, 0, -1);
      if(StringFind(objName, TF_PREFIX) != 0) continue;
      if(StringFind(objName, "LIVE") >= 0) continue;
      
      // Extract timestamp from object name
      int lastUnderscore = StringFind(objName, "_", StringLen(TF_PREFIX));
      if(lastUnderscore < 0) continue;
      
      string timeStr = StringSubstr(objName, lastUnderscore + 1);
      datetime objTime = (datetime)StringToInteger(timeStr);
      
      // Check if this object falls within ANY detected event range
      for(int i=0; i<ArraySize(events); i++){
         if(objTime >= events[i].startTime && objTime <= events[i].endTime){
            ObjectSetInteger(0, objName, OBJPROP_COLOR, events[i].clr);
            
            // Add timestamp to protection array (not object name)
            if(!IsTimestampProtected(objTime)){
               int n = ArraySize(g_Set1ProtectedTimestamps);
               ArrayResize(g_Set1ProtectedTimestamps, n + 1);
               g_Set1ProtectedTimestamps[n] = objTime;
            }
            
            recoloredCount++;
            break; // Object matched, move to next object
         }
      }
   }
   
   // Push all events to typed events array
   for(int i=0; i<ArraySize(events); i++){
      PushTypedEvent(0, events[i].eventId, events[i].slotEnd);
   }
   
   // Store most recent event range for one-time recoloring after next timeframing scan
   if(ArraySize(events) > 0){
      int lastIdx = ArraySize(events) - 1;
      g_CurrentSet3RangeStart = events[lastIdx].startTime;
      g_CurrentSet3RangeEnd = events[lastIdx].endTime;
      g_CurrentSet3Color = events[lastIdx].clr;
      g_NeedSet3Recolor = true;
      
      // Play sound for most recent historical event if enabled
      if(g_Sound[0]){
         EnqueueUniqueSound(g_SoundFile[0], 5, true, SetSoundTag(0));
      }
   }
   
   if(VerboseDiagnostics) Print("ProcessSet1HistoricalRanges: Completed. Processed: ", maxRangesToProcess-g_Depth[0], " ranges, Detected: ", eventsDetected, " events, Recolored: ", recoloredCount, " markers");
}

void ProcessPreloadStep(){
   datetime now=TimeCurrent();
   int steps=200;

   for(int step=0; step<steps; step++){
      datetime t=0;
      for(int s=0;s<SETS;s++){
         datetime c=g_PreloadCursor[s];
         if(c<=0 || c>=now) continue;
         if(t==0 || c<t) t=c;
      }
      if(t==0) break;

      int evt[SETS]={CE_NONE,CE_NONE,CE_NONE};
      ComparisonResult cv[SETS], cr[SETS];
      bool has[SETS]={false,false,false};
      datetime start[SETS]={0,0,0}, end[SETS]={0,0,0};

      for(int s=0;s<SETS;s++){
         if(g_PreloadCursor[s]!=t) continue;
         end[s]=t; start[s]=t-g_Sec[s];
         g_PreloadCursor[s]+=g_Sec[s];

         if(!(IsTradingAllowed(end[s]) || MQLInfoInteger(MQL_TESTER)!=0)) continue;
         has[s]=BuildSlotTextAndDetect(s,end[s],evt[s],cv[s],cr[s]);
      }

      int winner=-1, bestP=0;
      for(int s=0;s<SETS;s++){
         if(!has[s] || evt[s]==CE_NONE) continue;
         int p=EventPriority(evt[s]);
         if(winner==-1 || p>bestP || (p==bestP && s<winner)){ winner=s; bestP=p; }
      }
      if(winner!=-1) DrawAndNotifyWinner(winner,evt[winner],cv[winner],cr[winner],start[winner],end[winner]);

      for(int s=0;s<SETS;s++) if(g_PreloadCursor[s]>0 && g_PreloadCursor[s]>=now) g_PreloadCursor[s]=0;
   }
}

int OnInit(){
   // Initialize Cube globals
   for(int s=0;s<SETS;s++){ g_PreloadCursor[s]=0; g_SlotText[s]=""; g_LastSoundRotation[s]=0; g_Sec[s]=0; }
   ArrayResize(g_SoundQueue,0); g_NextSoundTime=0;
   
   // Always clear SET 1 protected timestamps on init (prevents stale references)
   ArrayResize(g_Set1ProtectedTimestamps, 0);

   int h,m;
   bool ok1=ParseHourMin(Session1Start,h,m); if(ok1) g_S1StartMin=h*60+m;
   bool ok2=ParseHourMin(Session1End,h,m);   if(ok2) g_S1EndMin=h*60+m;
   bool ok3=ParseHourMin(Session2Start,h,m); if(ok3) g_S2StartMin=h*60+m;
   bool ok4=ParseHourMin(Session2End,h,m);   if(ok4) g_S2EndMin=h*60+m;
   g_SessionsValid=(ok1&&ok2&&ok3&&ok4);

   LoadSetConfig();
   for(int s=0;s<SETS;s++) g_Sec[s]=SlotSeconds(g_TF[s]);

   // Clear markers if requested
   if(ClearMarkerFileOnInit){
      // Clear Cube markers
      for(int i=ObjectsTotal(0)-1;i>=0;i--){
         string n=ObjectName(0,i);
         if(StringFind(n,"Set1_")==0 || StringFind(n,"Set2_")==0 || StringFind(n,"Set3_")==0) ObjectDelete(0,n);
      }
      ClearTypedEvents_All();
      
      // Clear Timeframing markers
      DeleteTimeframingMarkers();
   }

   // Initialize Timeframing
   g_NewTimeframingRangeDetected = false;
   ArrayResize(g_TFRanges, 0);
   
   if(VerboseDiagnostics) Print("OnInit: Starting timeframing scan");
   ScanAndDrawTimeframing();
   
   // Process SET 1 historical events on all loaded timeframing ranges
   if(g_Draw[0]){
      if(VerboseDiagnostics) Print("OnInit: Processing SET 1 historical ranges. Ranges available: ", ArraySize(g_TFRanges));
      ProcessSet1HistoricalRanges();
   }
   else{
      if(VerboseDiagnostics) Print("OnInit: SET 1 drawing disabled, skipping historical processing");
   }
   
   datetime t[3];
   if(CopyTime(_Symbol, TimeframingTF, 0, 3, t) >= 2)
      g_lastProcessedClosedBarTime = t[1];

   g_liveBarOpenTime = iTime(_Symbol, TimeframingTF, 0);
   g_liveInitialized = false;

   // Cube historical preload for SET 2 & 3 ONLY (SET 1 already processed above)
   if(HistoricalDrawDays>0){
      long back=(long)HistoricalDrawDays*86400L;
      datetime start=(datetime)(TimeCurrent()-back);
      // Skip SET 1 (s=0), only initialize cursors for SET 2 and SET 3
      for(int s=1;s<SETS;s++) if(g_Sec[s]>0) g_PreloadCursor[s]=start-(start%g_Sec[s]);

      int safety=0;
      // Only wait for SET 2 and SET 3 cursors (SET 1 cursor remains 0)
      while((g_PreloadCursor[1]>0||g_PreloadCursor[2]>0) && safety<40000){
         ProcessPreloadStep(); safety++; Sleep(1);
      }
   }

   EventSetTimer((uint)MathMax(1,g_TimerIntervalSeconds));
   return INIT_SUCCEEDED;
}

void OnDeinit(const int reason){ 
   EventKillTimer(); 
   DeleteLiveMarker();
   Comment(""); 
}

void OnTick(){
   // Timeframing: tick-by-tick live marker update
   UpdateLiveFromTick();

   // Timeframing: rescan on new closed candle
   datetime t[3];
   if(CopyTime(_Symbol, TimeframingTF, 0, 3, t) >= 2){
      datetime lastClosed = t[1];
      if(lastClosed != g_lastProcessedClosedBarTime){
         g_lastProcessedClosedBarTime = lastClosed;
         DeleteLiveMarker();
         ScanAndDrawTimeframing();
         
         // Recolor current SET 1 event range only (if one was detected)
         if(g_NeedSet3Recolor){
            RecolorTimeframingRange(g_CurrentSet3RangeStart, g_CurrentSet3RangeEnd, g_CurrentSet3Color);
            g_NeedSet3Recolor = false;  // Clear flag after one-time recolor
         }
      }
   }
}

void OnTimer(){
   datetime now=TimeCurrent();

   if(!IsTradingAllowed(now)){
      if(g_LastComment!="OUTSIDE TRADING SESSIONS — PAUSED"){
         Comment("OUTSIDE TRADING SESSIONS — PAUSED");
         g_LastComment="OUTSIDE TRADING SESSIONS — PAUSED";
      }
      if(!(g_PreloadCursor[0]>0||g_PreloadCursor[1]>0||g_PreloadCursor[2]>0||MQLInfoInteger(MQL_TESTER)!=0)) return;
   }

   if(g_PreloadCursor[0]>0||g_PreloadCursor[1]>0||g_PreloadCursor[2]>0){ ProcessPreloadStep(); return; }

   static datetime lastEnd[SETS];
   datetime end[SETS]={0,0,0}, start[SETS]={0,0,0};
   int      evt[SETS]={CE_NONE,CE_NONE,CE_NONE};
   ComparisonResult cv[SETS], cr[SETS];
   bool     has[SETS]={false,false,false};

   for(int s=0;s<SETS;s++){
      // SET 1 uses timeframing ranges, check only when new range detected
      if(s == 0){
         if(g_NewTimeframingRangeDetected && ArraySize(g_TFRanges) > 0){
            end[s] = now;
            start[s] = now;
            has[s] = BuildSlotTextAndDetect(s, end[s], evt[s], cv[s], cr[s]);
         }
         continue;
      }
      
      // SET 2 & 3 use time slots
      if(g_Sec[s]<=0) continue;
      end[s]=now-(now%g_Sec[s]);
      if(end[s]==lastEnd[s]) continue;
      lastEnd[s]=end[s];
      start[s]=end[s]-g_Sec[s];
      if(!IsTradingAllowed(end[s]) && MQLInfoInteger(MQL_TESTER)==0) continue;
      has[s]=BuildSlotTextAndDetect(s,end[s],evt[s],cv[s],cr[s]);
   }

   for(int i=0;i<SETS;i++){
      if(!has[i] || evt[i]==CE_NONE) continue;
      datetime t=end[i];
      int winner=i, bestP=EventPriority(evt[i]);

      for(int j=0;j<SETS;j++){
         if(j==i || !has[j] || evt[j]==CE_NONE) continue;
         if(end[j]!=t) continue;
         int p=EventPriority(evt[j]);
         if(p>bestP || (p==bestP && j<winner)){ bestP=p; winner=j; }
      }
      if(i!=winner) evt[i]=CE_NONE;
   }

   for(int s=0;s<SETS;s++) if(has[s] && evt[s]!=CE_NONE) DrawAndNotifyWinner(s,evt[s],cv[s],cr[s],start[s],end[s]);

   // Reset timeframing range flag after processing
   g_NewTimeframingRangeDetected = false;

   string full=g_SlotText[0];
   if(StringLen(g_SlotText[1])>0) full+="\n"+g_SlotText[1];
   if(StringLen(g_SlotText[2])>0) full+="\n"+g_SlotText[2];
   StringToUpper(full);
   if(full!=g_LastComment){ Comment(full); g_LastComment=full; }

   ProcessSoundQueue();
}

//-------------------- HOTKEY TRADING -------------------------------
void FirstCharUpperCode(const string raw,int &codeUp){
   codeUp=-1; string t=Trim(raw); if(StringLen(t)==0) return;
   int c=(int)StringGetCharacter(t,0);
   if(c>=97&&c<=122) c-=32;
   codeUp=c;
}

bool ExecuteMarketDeal(const int orderType){
   double vol=TradeLotSize; if(vol<=0.0) return false;
   MqlTradeRequest req; MqlTradeResult res; ZeroMemory(req); ZeroMemory(res);

   req.action=TRADE_ACTION_DEAL;
   req.symbol=_Symbol;
   req.volume=vol;
   req.type=(ENUM_ORDER_TYPE)orderType;
   req.price=(orderType==ORDER_TYPE_BUY?SymbolInfoDouble(_Symbol,SYMBOL_ASK):SymbolInfoDouble(_Symbol,SYMBOL_BID));
   req.deviation=10;
   req.type_filling=ORDER_FILLING_FOK;
   req.type_time=ORDER_TIME_GTC;
   req.comment="Cube_hotkey";

   if(!OrderSend(req,res)) return false;
   return (res.retcode==TRADE_RETCODE_DONE || res.retcode==TRADE_RETCODE_DONE_PARTIAL);
}

void EmergencyCloseAllPositions(){
   for(int i=PositionsTotal()-1;i>=0;i--){
      ulong ticket=PositionGetTicket(i);
      if(ticket==0 || !PositionSelectByTicket(ticket)) continue;
      if(PositionGetString(POSITION_SYMBOL)!=_Symbol) continue;

      int posType=(int)PositionGetInteger(POSITION_TYPE);
      double posVol=PositionGetDouble(POSITION_VOLUME);
      if(posVol<=0.0) continue;

      MqlTradeRequest req; MqlTradeResult res; ZeroMemory(req); ZeroMemory(res);
      req.action=TRADE_ACTION_DEAL;
      req.symbol=_Symbol;
      req.volume=posVol;
      req.type=(posType==POSITION_TYPE_BUY?ORDER_TYPE_SELL:ORDER_TYPE_BUY);
      req.price=(req.type==ORDER_TYPE_BUY?SymbolInfoDouble(_Symbol,SYMBOL_ASK):SymbolInfoDouble(_Symbol,SYMBOL_BID));
      req.position=ticket;
      req.deviation=10;
      req.type_filling=ORDER_FILLING_FOK;
      req.type_time=ORDER_TIME_GTC;
      req.comment="Cube_close_all";

      if(!OrderSend(req,res)){ /* optional Print */ }
   }
}

void OnChartEvent(const int id,const long &lparam,const double &dparam,const string &sparam){
   if(id!=CHARTEVENT_KEYDOWN) return;
   int pressed=(int)lparam; if(pressed<=0) return;
   if(pressed>=97&&pressed<=122) pressed-=32;

   int buy=-1,sell=-1,close=-1;
   FirstCharUpperCode(Hotkey_Buy,buy);
   FirstCharUpperCode(Hotkey_Sell,sell);
   FirstCharUpperCode(Hotkey_Close,close);

   if(buy>0 && pressed==buy)     { ExecuteMarketDeal(ORDER_TYPE_BUY);  return; }
   if(sell>0 && pressed==sell)  { ExecuteMarketDeal(ORDER_TYPE_SELL); return; }
   if(close>0 && pressed==close){ EmergencyCloseAllPositions();       return; }
}
//+------------------------------------------------------------------+
