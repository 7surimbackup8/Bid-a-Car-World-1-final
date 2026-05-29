# 🎮 BID A CAR (WORLD 1) - GAME ARCHITECTURE SCHEMA

**Game Type:** Roblox Bid Battle Simulator with RNG Garages & Progression  
**Status:** Development from scratch  
**Date Created:** May 19, 2026

---

## 📋 TABLE OF CONTENTS

1. [Game Overview](#game-overview)
2. [Game Flow & Player Journey](#game-flow--player-journey)
3. [Core Systems Architecture](#core-systems-architecture)
4. [Manager Breakdown](#manager-breakdown)
5. [DataStore Structure](#datastore-structure)
6. [UI/UX Wireframes](#uiux-wireframes)
7. [Folder Structure](#folder-structure)
8. [Script Breakdown](#script-breakdown)
9. [Bid Mechanics Deep Dive](#bid-mechanics-deep-dive)
10. [Item System](#item-system)
11. [Progression & Rebirths](#progression--rebirths)

---

## 🎯 GAME OVERVIEW

### What is Bid A Car?
A Roblox game where players engage in **bid battles** to win RNG garages containing:
- **Random Rarity Cars** (Common → Rare → Epic → Legendary → SPEC)
- **Decorations** (20$ value, 4-50 per garage based on tier)
- **Lockers** (time-locked containers with dice & potions)
- **NPCs** (from Dice Shop, provide income boosts on conveyors)

### Core Loop
```
Player → Select Bid Tier → Enter Bid Battle → Win/Lose → Collect Rewards → 
Place on Plot → Generate Income → Rebirth → Unlock World 2
```

### Start State
- **Starting Money:** $700
- **Starting Conveyors:** 3 (6 spots total on plot)
- **First Task:** Mini-tutorial → First bid (BEGINNER $200)

---

## 🎬 GAME FLOW & PLAYER JOURNEY

```
┌────────────────────────────────────────────────────────────────┐
│                    GAME START (New Player)                       │
└─────────────────────────────┬───────────────────────────────────┘
                              │
                              ▼
                    ┌──────────────────┐
                    │  Mini Tutorial   │
                    │   (1 minute)     │
                    │  Mandatory       │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │  Main Lobby      │
                    │ $700 in pocket   │
                    └────────┬─────────┘
                             │
          ┌──────────────────┼──────────────────┐
          ▼                  ▼                  ▼
    ┌──────────┐      ┌──────────┐      ┌──────────┐
    │ Garage   │      │ Events   │      │ Shop     │
    │ (BID)    │      │ (Future) │      │ (Dice)   │
    └────┬─────┘      └──────────┘      └──────────┘
         │
         ▼
┌────────────────────────────���────────────────────┐
│  TIER SELECTION (Pop-up Window - Scrollable)     │
│  Horizontal single row (left ↔ right)            │
│                                                 │
│  [BEGINNER] [ADVANCED] [EXPERT] [CHOSEN] [...]  │
│     $200      $500      $1200    $2500          │
└────────────┬─────────────────────────────────────┘
             │
             ▼ (Teleport to RNG Garage)
┌──────────────────────────────────────────┐
│     RNG GARAGE (Bid Battle Arena)         │
│     PHYSICALLY ON MAP (First-person POV)  │
│                                          │
│  Player ↔ Bot ↔ Bot ↔ Bot                │
│  (Player on left, Bots in a line)        │
│                                          │
│         [Car at back with decos]         │
│  (Colored if owned, Black if locked)    │
│                                          │
│         Current Bid: $500                │
│      ┌─────────────────────────┐        │
│      │ Player: $120 ✓          │        │
│      │ Bot1: $80               │        │
│      │ Bot2: $100              │        │
│      └─────────────────────────┘        │
│                                          │
│     Countdown: 2 → 1 → 0                │
│                                          │
│        [RED BID BUTTON]                 │
│     [Inventory] [Settings]              │
└────────────┬─────────────────────────────┘
             │
      ┌──────┴──────┐
      ▼             ▼
   WIN           LOSS
    │              │
    ▼              ▼
┌─────────┐   ┌──────────────┐
│ Selection│   │ Money Refund │
│ 0-2 Deco │   │ (if no bid)  │
│ Auto-sell│   │ OR Lose All  │
│ Lockers  │   │ (if bidded)  │
└────┬─────┘   └──────────────┘
     │
     ▼ (Teleport back to Garage)
┌─────────────────────────────┐
│ Inventory Updated           │
│ Items, Cars, Lockers        │
│ Income Ready to Collect     │
└────────────┬────────────────┘
             │
             ▼
    ┌────────────────────┐
    │  Place on Plot     │
    │ Or Trade/Sell      │
    │ Or Collect Income  │
    └────────┬───────────┘
             │
             ▼
    ┌────────────────────┐
    │  Rebirth Check?    │
    │ ($2000/$5000/etc)  │
    └────────┬───────────┘
             │
      ┌──────┴──────────┐
      ▼                 ▼
   YES (Rebirth)    NO (Continue)
    │                 │
    ▼                 ▼
 LOOP             LOOP
```

---

## 🏗️ CORE SYSTEMS ARCHITECTURE

### System Dependencies Graph
```
DataStoreManager (Foundation - Saves Everything)
      │
      ├─→ PlayerDataManager (Player State)
      │
      ├─→ BidEngine (Bid Battle Logic)
      │   ├─→ NPCBidController (Bot AI)
      │   └─→ RNGGarageGenerator (Garage Creation)
      │
      ├─→ InventoryManager (Items, Cars, Lockers, Decorations)
      │   └─→ ItemDatabase (All item definitions)
      │
      ├─→ PlotManager (Conveyor System, Placement)
      │   └─→ IncomeGenerator (Passive Money Generation)
      │
      ├─→ RebirthManager (Progression, Unlocks)
      │
      ├─→ ShopManager (Dice Shop)
      │   └─→ DiceRNG (NPC Generation)
      │
      ├─→ TradeManager (Player-to-Player Trading)
      │
      ├─→ UIManager (All UI Rendering)
      │   ├─→ BidUI
      │   ├─→ InventoryUI
      │   ├─→ TierSelectionUI
      │   └─→ PlotUI
      │
      └─→ TeleportManager (Location Transitions)
```

---

## 👨‍💼 MANAGER BREAKDOWN

### 1. **DataStoreManager**
**Purpose:** Auto-save all player data every 2 minutes

**Saves:**
```lua
{
  playerId = "player_id",
  timestamp = os.time(),
  money = 700,
  rebirthCount = 0,
  inventory = { ... },
  plot = { ... },
  stats = { ... }
}
```

**Key Functions:**
- `Save()` - Called every 2 min
- `Load()` - On player join
- `UpdateField(key, value)` - Real-time updates

---

### 2. **PlayerDataManager**
**Purpose:** Handle all player state in-memory

**Stores:**
```lua
players[playerId] = {
  money = 700, ( or how much he got at that time ) 
  rebirths = {
    count = 0,
    timestamp = 0
  },
  inventory = {
    items = {},      -- Decorations, Potions
    cars = {},       -- Owned cars
    lockers = {},    -- Time-locked containers
    dice = {}        -- From shop
  },
  plot = {
    conveyors = {
      { car = nil, npc = nil },
      { car = nil, npc = nil },
      ...
    }
  },
  luckBoosts = {
    active = {},     -- Currently running
    expired = {}
  },
  stats = {
    totalBidsWon = 0,
    totalBidsLost = 0,
    totalMoneySpent = 0,
    totalMoneyEarned = 0
  }
}
```

**Key Functions:**
- `GetPlayer(playerId)`
- `UpdateMoney(playerId, amount)`
- `AddToInventory(playerId, itemType, item)`
- `RemoveFromInventory(playerId, itemType, itemId)`

---

### 3. **BidEngine**
**Purpose:** Control entire bid battle flow

**States:**
- `WAITING` - Waiting for players
- `BIDDING` - Active bidding phase
- `SETTLING` - Determining winner
- `COMPLETED` - Bid finished

**Flow:**
```
1. Player selects tier → Teleport to RNG Garage
2. RNG Garage generated (4-7 deco + 1 car etc etc -> check GARAGE TIERS)
3. Set starting bid price (tier-based)
4. Start bidding phase (2 sec player, 1 sec bots)
4.5. Bid raise with only 10% of the live bid value (current bid 300 bid 30$)
5. Countdown: 2 → 1 → 0
6. If no bids: auto-end, refund entry
7. If bids made: Calculate winner (highest bid)
8. Winner selection phase (0-2 decorations)
9. Loser handling (money lost)
10. Teleport back to garage
```

**Key Functions:**
- `StartBid(playerId, tierPrice, garageType)`
- `PlayerBid(playerId, amount)`
- `BotBid(amount)` - Called by NPCBidController
- `CalculateWinner()`
- `SettleWin(playerId)`
- `SettleLoss(playerId, bidAmount)`
- `SelectDecorations(playerId, decorationIds[])`

---

### 4. **NPCBidController**
**Purpose:** AI brain for all bot bidders (shared across all bots)

**Core Logic:**
- All bots use **SINGLE SHARED AI** (not individual)
- Bids range: **35-65% of garage value**
- Each bid: random amount within that range
- Timing: 1 second between bids (not simultaneous)
- Stop condition: Random stop between 35-65%

**Algorithm:**
```
garageValue = CarPrice + (DecoCount * 20) + LocalkerBonus
minBid = garageValue * 0.35
maxBid = garageValue * 0.65
randomStopPoint = Random(minBid, maxBid)

while currentBidAmount < randomStopPoint:
    currentBidAmount += Random(botMinIncrement, botMaxIncrement)
    wait(1 second)
    Bid(currentBidAmount)
    
    if currentBidAmount >= randomStopPoint:
        Stop()
```

**Key Functions:**
- `CalculateGarageValue(garage)`
- `DetermineBidRange(garageValue)`
- `GenerateNextBid(currentBid)`
- `ShouldContinueBidding(currentBid, stopPoint)`

---

### 5. **RNGGarageGenerator**
**Purpose:** Create random garages based on tier

**Tier Specifications:**

| Tier | Entry | Car Rarity | Deco Count | Locker Rate |
|------|-------|-----------|-----------|------------|
| BEGINNER | $200 | Common-Rare | 4-7 | 1/10 |
| ADVANCED | $500 | Uncommon-Epic | 7-13 | 1/4 |
| EXPERT | $1200 | Rare-Legendary + 3% SPEC | 13-21 | 1/2 |
| CHOSEN | $2500 | Rare-Legendary + 10% SPEC | 21-50 | 1/1 (always) |
| ------ | $5000 | Epic-Legendary +25% SPEC | 50-80 | 2x 1/1 (always double)  ( UNLOCKED AFTER DOING A CHOSEN BID ) |

**CAR RARITIES**

BEGINNER ( common 1/4 | uncommon 1/5 | rare 1/8 )
ADVANCED ( uncommon 1/8 | rare 1/6 | epic 1/10 )
EXPERT ( rare 1/10 | epic 1/6 | legendary 1/12 - 3% SPEC INSTEAD OF LEGENDARY )
CHOSEN ( rare 1/36 | epic 1/24 | legendary 1/8 - 10% SPEC INSTEAD OF LEGENDARY )
------ ( epic 1/24 | legendary 1/8 - 25% SPEC INSTEAD OF LEGENDARY )

**Generation:**
```
1. Roll car rarity based on tier
2. Roll car model from that rarity pool
3. Roll decoration count (min-max based on tier)
4. Generate decoration models (all same value $20)
5. Determine if locker drops (based on rate)
6. If locker: Roll locker rarity (33% equal chance)
7. Populate locker with contents (see Item System)
8. Spawn on map in RNG Garage location
```

**Key Functions:**
- `GenerateGarage(tierType)`
- `RollCarRarity(tier)`
- `SelectCarModel(rarity)`
- `GenerateDecorations(count)`
- `RollLocker(tier)`
- `PopulateLockerContents(lockerRarity)`

---

### 6. **InventoryManager**
**Purpose:** Manage all inventory items (Items, Cars, Lockers, Index)
UI Bid Battles Style cyan-purple

**Structure:**
```lua
inventory[playerId] = {
  -- Items Tab
  items = {
    { id = "deco_001", name = "Gold Vase", value = 20, rarity = "common" },
    { id = "deco_002", name = "Plant", value = 20, rarity = "common" },
    { id = "potion_luck_001", name = "Luck Boost", duration = 3600, type = "silver" },
    { id = "dice_basic_001", name = "Basic Dice", type = "basic" },
    { id = "locker_001", name = "Silver Locker", rarity = "silver", openedAt = 0, unopened = true }
  },
  
  -- Cars Tab
  cars = {
    { id = "car_x1", name = "Car X1", rarity = "common", income = 50, owned = true },
    { id = "car_x5", name = "Car X5", rarity = "uncommon", income = 100, owned = false },
    { id = "car_epic_01", name = "Epic Car", rarity = "epic", income = 500, owned = true }
  },
  
  -- Locker Tab
  lockers = {
    { id = "locker_silver_001", rarity = "silver", timeToOpen = 3600, unopened = true },
    { id = "locker_gold_001", rarity = "gold", timeToOpen = 14400, unopened = false, contents = {...} }
  },
  
  -- Index Tab (View Only - All Cars)
  index = {
    -- Shows all cars in game with rarity, income, and lock status
  }
}
```

**UI Tab Structure:**

1. **Items Tab** - Grid layout
   - Decorations (4 per row, scrollable)
   - Potions (4 per row, scrollable)
   - Dice (4 per row, scrollable)
   - Lockers (separate section)

2. **Cars Tab** - Modern Grid with Rarity Sections
   - Each car shows: **Image + Name + Income**
   - If owned: Colored car image + Income value visible
   - If not owned: Black car image + Lock icon + "???" income
   - Sorted by rarity

3. **Locker Tab** - List with time remaining
   - Silver (1 hour) - [OPEN] button if ready
   - Gold (4 hours) - Timer or [OPEN]
   - Black (8 hours) - Timer or [OPEN]
   
   **When opened:**
   - Modal popup showing rewards
   - Auto-collect into inventory
   - Timer resets

4. **Index Tab** - View only
   - All cars in game
   - Owned cars: Colored + Income value
   - Locked cars: Black + Lock icon + "???" income
   - Sorted by rarity (Common → Uncommon → Rare → Epic → Legendary → SPEC)

**Key Functions:**
- `GetInventory(playerId)`
- `AddItem(playerId, itemType, item)`
- `RemoveItem(playerId, itemType, itemId)`
- `ListItems(playerId)`
- `GetCars(playerId, ownedOnly = true)`
- `OpenLocker(playerId, lockerId)`
- `UsePotion(playerId, potionId)`
- `ConsumeItem(playerId, itemType, itemId, quantity)`

---

### 7. **PlotManager**
**Purpose:** Manage player plot, conveyors, and placements
PLOT IS ON THE MAP PHISICALLY, i dont need UI for this, backend for place car and npc + decorations on the plot Building phase

**Plot Structure:**
```lua
plot[playerId] = {
  conveyors = {
    {
      id = "conveyor_1",
      car = { id = "car_x1", income = 50, boosted = false },
      npc = { id = "npc_booster_1", boostType = "golden_dice", boostPercent = 50 },
      income_accumulated = 1250,
      lastCollected = os.time()
    },
    { id = "conveyor_2", car = nil, npc = nil, income_accumulated = 0, lastCollected = 0 },
    ...
  },
  totalConveyors = 3,  -- Unlocks at: 3 (free), +1 at Rebirth1, +1 at Rebirth3, +1 at Robux19
  unlockedCount = 3
}
```

**Conveyor System:**
- Max 6 conveyors total
- Start with 3 free
- +1 at Rebirth 1 ($2000)
- +1 at Rebirth 3 ($10000)
- +1 at Robux 19 (premium)
- **1 car + 1 NPC per conveyor**
- Income: BASE (car) + BOOST (NPC %)

**Income Generation:**
```
Base Income = Car's $/min
With NPC = Car Income * (1 + NPC Boost%)

Example:
Car X1 = $50/min
NPC Booster (Golden Dice) = 50% boost
Total = $50 * 1.5 = $75/min

Offline MAX = 8 hours
$75/min * 60 = $4,500/hour
$4,500 * 8 = $36,000 max offline
```

**Collection Mechanic:**
- Player walks to car on plot
- Press `E` → "Collect" prompt
- Shows total accumulated since last collect
- Updates in real-time
- Money added to player wallet
- Counter resets

**Key Functions:**
- `PlaceCar(playerId, carId, conveyorId)`
- `PlaceNPC(playerId, npcId, conveyorId)`
- `RemoveCar(playerId, conveyorId)`
- `RemoveNPC(playerId, conveyorId)`
- `CollectIncome(playerId, conveyorId)`
- `CalculateOfflineIncome(playerId)`
- `UnlockConveyor(playerId)` - Rebirth system calls this
- `GetPlotStatus(playerId)`

---

### 8. **IncomeGenerator**
**Purpose:** Passive money generation system (offline included)

**Logic:**
```
When player joins/loads:
1. Check last logout time
2. Calculate time offline (max 8 hours per conveyor)
3. For each conveyor with a car:
   - Calculate: minutes_offline * income_per_minute (capped at 8 hours)
   - Add to accumulated income
4. When player presses E on car:
   - Show accumulated amount
   - Add to wallet
   - Reset counter
   - Reset lastCollected timestamp
```

**Calculation:**
```lua
function CalculateAccumulatedIncome(conveyor, timeSinceLastCollect)
    maxOfflineTime = 8 * 60 * 60  -- 8 hours in seconds
    actualTime = min(timeSinceLastCollect, maxOfflineTime)
    
    baseIncome = conveyor.car.income
    npcBoost = conveyor.npc ? conveyor.npc.boostPercent : 0
    totalIncomePerSecond = (baseIncome / 60) * (1 + npcBoost/100)
    
    accumulated = totalIncomePerSecond * actualTime
    return floor(accumulated)
end
```

**Key Functions:**
- `CalculateOfflineIncome(playerId, conveyorId)`
- `AddAccumulatedIncome(playerId, conveyorId, amount)`
- `CollectAll(playerId)` - Collect from all conveyors at once
- `UpdateIncomePerSecond(playerId, conveyorId)`

---

### 9. **RebirthManager**
**Purpose:** Handle progression milestones and unlocks

**Rebirth Levels:**

| Level | Cost | Effect | Unlocks |
|-------|------|--------|---------|
| 0 (Start) | - | - | - |
| 1 | $2,000 | Reset money to $0 | +1 Conveyor (4 total), +2 Luck Boosts |
| 2 | $5,000 | Reset money to $0 | Trade System (Player ↔ Player) |
| 3 | $10,000 | Reset money to $0 | +1 Conveyor (5 total), Access World 2 |

**Mechanics:**
```
1. Player has $2000+ -> click on rebirth button -> Rebirth UI
2. Player confirms rebirth
3. Money set to $0 (lose all)
4. Rebirth count +1
5. Unlock new features
6. Keep all items/cars/lockers
7. Keep NPCs on conveyors
8. Reset money only
9. Update DataStore
```

**Key Functions:**
- `CanRebirth(playerId, level)`
- `ExecuteRebirth(playerId, level)`
- `UnlockFeature(playerId, feature)`
- `GetRebirthStatus(playerId)`
- `GetNextRebirthCost(playerId)`

---

### 10. **ShopManager**
**Purpose:** Dice shop where players buy dice for NPC generation

**Shop Interface:**
```
┌────────────────────────────────┐
│         DICE SHOP              │
├────────────────────────────────┤
│                                │
│ [BASIC DICE] [GOLDEN DICE]     │
│ $150         $300              │
│ [BUY]        [BUY]             │   modern smooth UI style
│                                │
│ [DIAMOND DICE] [NA-SPEC DICE]  │
│ $1100        $2500             │
│ [BUY]        [BUY]             │
│                                │
└────────────────────────────────┘
```

**Dice Types & Contents:**

| Dice | Price | NPC Rarity | Boost Range | Availability |
|------|-------|------------|-------------|--------------|
| Basic | $150 | Common-Rare | 10-30%+ | Always |
| Golden | $300 | Uncommon-Epic | 20-60% | Always |
| Diamond | $1100 | Rare-Legendary | 50-100% | Always |
| NA-SPEC | $2500 | Rare-SPEC | 50-150% | Rebirth 1+ |

Basic (1/2 common | 1/6 uncommon | 1/10 rare )
Golden ( 1/4 uncommon | 1/6 rare | 1/10 epic )
Diamond ( 1/10 rare | 1/6 epic | 1/12 legendary )
NA-SPEC ( 1/21 rare | 1/16 epic | 1/4 legendary | 1/8 SPEEC )
-----------------RNG---------------------

**Flow:**
0. Player clicks Shop button and get teleported to Merchant where he interact with the NPC ( E ) 
1. UI opens and player buys dice ($X deducted)
2. Dice goes to Items → Inventory
3. Player opens dice → RNG rolls NPC
4. NPC goes to Items → Inventory
5. Player places NPC on conveyor

**Key Functions:**
- `BuyDice(playerId, diceType)`
- `OpenDice(playerId, diceId)` - RNG NPC generation
- `GetDiceShop()` - Returns all available dice
- `CanAffordDice(playerId, diceType)`

---

### 11. **DiceRNG**
**Purpose:** Generate random NPCs from dice

**NPC Generation:**
```lua
function GenerateNPC(diceType)
    rarities = GetRaritiesForDice(diceType)
    boostPercent = RandomRange(MinBoost[diceType], MaxBoost[diceType])
    npcName = GenerateRandomName()
    
    return {
        id = generateUUID(),
        name = npcName,
        type = diceType,
        boostPercent = boostPercent,
        rarity = SelectRarity(rarities),
        createdAt = os.time()
    }
end
```

**Key Functions:**
- `RollNPC(diceType)`
- `GetNPCStats(npcId)`
- `GenerateBoostPercent(diceType)`

---

### 12. **TradeManager** (Rebirth 2+)
**Purpose:** Player-to-player trading

**Trade System:**
- Modern UI (Pet Simulator style)
- 1v1 trades
- Only cars & NPCs tradeable
- No cash trading
- Confirm from both sides required

**Flow:**
```
1. Player A opens trade
2. Adds car/NPC to offer
3. Sends to Player B
4. Player B sees incoming trade
5. Player B adds car/NPC to counter
6. Both confirm [ACCEPT] or [DECLINE]
7. If both accept: Swap items
8. Update DataStore
```

**Key Functions:**
- `InitiateTrade(playerAId, playerBId)`
- `AddToTrade(playerId, tradeId, itemType, itemId)`
- `RemoveFromTrade(playerId, tradeId, itemType, itemId)`
- `AcceptTrade(playerId, tradeId)`
- `DeclineTrade(playerId, tradeId)`
- `CompleteTrade(tradeId)`
- `GetPendingTrades(playerId)`

---

### 13. **UIManager**
**Purpose:** Central hub for all UI rendering

**UI Screens:**

1. **Main Lobby Screen** - Physical world (not a UI pop-up)
   - Top-right buttons: [Events] [Garage] [Shop]
   - Bottom-right: Wallet display ($700)
   - Bottom-center: [Inventory] [Settings] buttons
   - Settings icon on left side
   - Merchant NPC visible with "Dice Shop - Open Shop" prompt
   - Garage teleporter visible

2. **Tier Selection UI** - Pop-up Window (Scrollable Horizontal)
   - Single horizontal row of bid tiers
   - Scroll left/right to view more tiers
   - Visible tiers: [BEGINNER $200] [ADVANCED $500] [EXPERT $1200] [CHOSEN $2500] [...5th tier]
   - Click on tier to select and confirm

3. **Bid Battle UI** - Physical World (First-person POV)
   - Live arena with players and bots physically positioned
   - Player on left, bots in line facing the car
   - Car at back with decorations displayed
   - Bid status panel showing current bid and all player bids
   - Countdown timer (2 → 1 → 0)
   - Large RED [BID] button at bottom-center
   - [Inventory] and [Settings] buttons accessible

4. **Inventory UI** - Pop-up Window
   - **Tabs at top:** [Items] [Cars] [Lockers] [Index]
   - **Cars Tab shows:**
     - Car image (no background, colored if owned, black if locked)
     - Car name
     - Income value
     - Grid layout, scrollable
   - **Items Tab shows:**
     - Decorations, Potions, Dice in grid layout
   - **Lockers Tab shows:**
     - List of lockers with timer/open status
   - **Index Tab shows:**
     - All cars in game, filtered by rarity
     - Colored if owned, black if locked

5. **RebirthUI** - Pop-up confirmation with benefits display

6. **TradeUI** - Pet Simulator style modern smooth (Rebirth 2+)

7. **DiceShopUI** - Pop-up window on Merchant NPC interaction (E key)

8. **ShopGamePassUI** - General shop grid (COMING SOON)

**Key Functions:**
- `ShowMainLobby(playerId)`
- `ShowTierSelection(playerId)`
- `ShowBidUI(playerId, garageInfo)`
- `ShowInventory(playerId, tab = "cars")`
- `ShowRebirthUI(playerId)`
- `ShowTradeUI(playerAId, playerBId)`
- `HideUI(playerId, uiName)`
- `UpdateUIElement(uiName, element, value)`

---

### 14. **TeleportManager**
**Purpose:** Handle all location transitions

**Teleport Points:**

| From | To | Trigger |
|------|-----|---------|
| Lobby | RNG Garage | BID tier selection |
| RNG Garage | Lobby | Exit/Lose bid |
| Lobby | Merchant | SHOP button click |
| Merchant | Plot | GARAGE button click |
| Plot | RNG Garage | BID button click |

**Key Functions:**
- `Teleport(playerId, destination, args = {})`
- `TeleportToRNGGarage(playerId, tierType)`
- `TeleportToLobby(playerId)`
- `TeleportToMerchant(playerId)`
- `GetTeleportPosition(location)`

---

## 💾 DATASTORE STRUCTURE

### Main Save File Schema

```json
{
  "playerId": "player_id_12345",
  "timestamp": 1715113200,
  "gameVersion": "1.0.0",
  
  "account": {
    "username": "PlayerName",
    "joinDate": 1715000000,
    "lastLogin": 1715113200,
    "totalPlaytime": 3600
  },
  
  "wallet": {
    "money": 1500,
    "lastUpdated": 1715113200
  },
  
  "progression": {
    "rebirthCount": 0,
    "rebirthTimestamps": [],
    "currentWorld": 1,
    "tutorialCompleted": true
  },
  
  "inventory": {
    "items": [
      {
        "id": "deco_vase_001",
        "type": "decoration",
        "name": "Gold Vase",
        "quantity": 1,
        "value": 20,
        "acquiredAt": 1715112000
      },
      {
        "id": "potion_luck_001",
        "type": "potion",
        "name": "Luck Boost",
        "rarity": "silver",
        "durationSeconds": 3600,
        "expiresAt": 1715116600
      },
      {
        "id": "dice_basic_001",
        "type": "dice",
        "diceType": "basic",
        "unopened": true,
        "acquiredAt": 1715110000
      },
      {
        "id": "locker_silver_001",
        "type": "locker",
        "rarity": "silver",
        "openedAt": 0,
        "unopened": true,
        "acquiredAt": 1715109000
      }
    ],
    
    "cars": [
      {
        "id": "car_x1_001",
        "model": "x1",
        "rarity": "common",
        "income": 50,
        "acquiredAt": 1715107000,
        "onConveyorId": "conveyor_1"
      },
      {
        "id": "car_x5_001",
        "model": "x5",
        "rarity": "uncommon",
        "income": 100,
        "acquiredAt": 1715105000,
        "onConveyorId": null
      }
    ]
  },
  
  "plot": {
    "totalConveyors": 3,
    "unlockedCount": 3,
    "conveyors": [
      {
        "id": "conveyor_1",
        "carId": "car_x1_001",
        "npcId": "npc_golden_001",
        "incomeAccumulated": 1250,
        "lastCollected": 1715112000
      },
      {
        "id": "conveyor_2",
        "carId": null,
        "npcId": null,
        "incomeAccumulated": 0,
        "lastCollected": 0
      },
      {
        "id": "conveyor_3",
        "carId": null,
        "npcId": null,
        "incomeAccumulated": 0,
        "lastCollected": 0
      }
    ]
  },
  
  "luckBoosts": {
    "active": [
      {
        "id": "boost_001",
        "type": "silver",
        "expiresAt": 1715116600,
        "percentIncrease": 20
      }
    ],
    "expired": []
  },
  
  "stats": {
    "totalBidsParticipated": 42,
    "totalBidsWon": 28,
    "totalBidsLost": 14,
    "totalMoneySpent": 8400,
    "totalMoneyEarned": 12500,
    "totalIncomeCollected": 5200,
    "longestWinStreak": 5,
    "favoriteCarModel": "x1",
    "favoriteTier": "BEGINNER"
  }
}
```

---

## 🎨 UI/UX WIREFRAMES

### 1. MAIN LOBBY (Physical World + UI Elements)
```
┌─────────────────────────────────────────────────────────────────┐
│ GAME WORLD VIEW (Physical Merchant Area)                         │
│                                                                  │
│  [⚙️] (Settings)              [Events] [Garage] [Shop]           │
│  (Left Side)          (Top-Right Buttons - Purple/Blue/Green)   │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Merchant NPC visible with "Dice Shop - Open Shop"      │   │
│  │  Stall with dice display                                │   │
│  │  Garage teleporter visible in world                     │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│                                                                  │
│  (Player Free Roam Area)                                        │
│                                                                  │
│                                  $700 (Wallet)                  │
│                                  [Inventory] [Settings]         │
│                           (Bottom-Center Buttons)               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 2. TIER SELECTION (Pop-up Window - Horizontal Scrollable)
```
┌──────────────────────────────────────────────────────────────┐
│  SELECT YOUR BID TIER                          [X]            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ◄ [BEGINNER] [ADVANCED] [EXPERT] [CHOSEN] [???] ►         │
│      $200      $500      $1200    $2500                     │
│                                                              │
│              [SCROLL HORIZONTALLY FOR MORE]                 │
│                                                              │
│                          [SELECT]  [CANCEL]                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### 3. BID INTERFACE (Physical World - First-person POV)
```
┌────────────────────────────────────────────────────────────┐
│ GARAGE AUCTION ARENA                                       │
│                                                            │
│  Barbara        Jack        Roblofía        Mashallah  │
│  (Player)       (Bot 1)     (Bot 2)        (Bot 3)     │
│  ┌────┐         ┌────┐      ┌────┐        ┌────┐      │
│  │ 😊 │         │ 🤖 │      │ 🤖 │       │ 🤖 │      │
│  └────┘         └────┘      └────┘        └────┘      │
│      ↓              ↓           ↓              ↓        │
│  ┌─────────────────────────────────────────────────┐   │
│  │ Current Bid: $500                              │   │
│  │ Player Bid: $120 ✓                             │   │
│  │ Bot 1: $80                                     │   │
│  │ Bot 2: $100                                    │   │
│  │ Bot 3: $150                                    │   │
│  └─────────────────────────────────────────────────┘   │
│                                                        │
│          ═══════════════════════════════════          │
│                                                        │
│               [YELLOW CAR IN BACK]                    │
│        ✦ Decoration  ✦ Decoration                    │
│       ✦ Decoration ✦ Decoration                      │
│                                                        │
│              Countdown: 2                              │
│                                                        │
│  ┌───────────────────────────────────────────────┐   │
│  │              [RED BID BUTTON]                  │   │
│  └───────────────────────────────────────────────┘   │
│                                                        │
│  [Inventory]    [Settings]    Wallet: $1250         │
│                                                        │
└────────────────────────────────────────────────────────┘
```

### 4. INVENTORY UI (Pop-up Window)
```
┌────────────────────────────────────────────────────────┐
│ INVENTORY                              [X]              │
├────────────────────────────────────────────────────────┤
│ [Items]  [CARS]  [Lockers]  [Index]                   │
├────────────────────────────────────────────────────────┤
│                                                        │
│  Common Cars:                                         │
│  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐             │
│  │ 🚗   │  │ 🚗   │  │ 🔒   │  │ 🔒   │             │
│  │ Car  │  │ Car  │  │ ???  │  │ ???  │             │
│  │ X1   │  │ X2   │  │ ???  │  │ ???  │             │
│  │$50   │  │$75   │  │$50   │  │$75   │             │
│  └──────┘  └──────┘  └──────┘  └──────┘             │
│                                                        │
│  Uncommon Cars:                                       │
│  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐             │
│  │ 🚗   │  │ 🚗   │  │ 🚗   │  │ 🔒   │             │
│  │ Car  │  │ Car  │  │ Car  │  │ ???  │             │
│  │ X5   │  │ X6   │  │ X7   │  │ ???  │             │
│  │$150  │  │$200  │  │$250  │  │$150  │             │
│  └──────┘  └──────┘  └──────┘  └──────┘             │
│                                                        │
│  [SCROLL DOWN FOR MORE]                              │
│                                                        │
└────────────────────────────────────────────────────────┘
```

### 5. INVENTORY - ITEMS TAB
```
┌────────────────────────────────────────────────────────┐
│ INVENTORY                              [X]              │
├────────────────────────────────────────────────────────┤
│ [ITEMS]  [Cars]  [Lockers]  [Index]                   │
├────────────────────────────────────────────────────────┤
│                                                        │
│  DECORATIONS:                                         │
│  ┌───┐ ┌───┐ ┌───┐ ┌───┐                             │
│  │ 🏺│ │🌱 │ │💡 │ │🖼 │                             │
│  │Vase│ │Plant│ │Lamp│ │Painting│                    │
│  │$20 │ │$20 │ │$20 │ │$20 │                        │
│  └───┘ └───┘ └───┘ └───┘                             │
│                                                        │
│  POTIONS:                                             │
│  ┌──────────────┐                                     │
│  │ ✨ Luck Boost│                                     │
│  │ Silver x2    │ [USE]                               │
│  └──────────────┘                                     │
│                                                        │
│  DICE:                                                │
│  ┌──────────────┐                                     │
│  │ 🎲 Basic     │                                     │
│  │ Dice x1      │ [OPEN]                              │
│  └──────────────┘                                     │
│                                                        │
│  LOCKERS:                                             │
│  ┌──────────────┐                                     │
│  │ 🔒 Silver    │                                     │
│  │ Opens: 0h 15m│ [OPEN-LOCKED]                      │
│  └──────────────┘                                     │
│                                                        │
└────────────────────────────────────────────────────────┘
```

### 6. INVENTORY - LOCKERS TAB
```
┌────────────────────────────────────────────────────────┐
│ INVENTORY                              [X]              │
├────────────────────────────────────────────────────────┤
│ [Items]  [Cars]  [LOCKERS]  [Index]                   │
├──────────��─────────────────────────────────────────────┤
│                                                        │
│  SILVER LOCKERS (1 hour):                             │
│  ┌──────────────────────────────┐                     │
│  │ 🔒 Silver Locker #1          │                     │
│  │ Opens in: 0h 15m 30s         │                     │
│  │                   [OPEN-LOCKED] │                     │
│  └──────────────────────────────┘                     │
│                                                        │
│  GOLD LOCKERS (4 hours):                              │
│  ┌──────────────────────────────┐                     │
│  │ 🔓 Gold Locker #1            │                     │
│  │ Ready to open!               │                     │
│  │                   [OPEN-READY]   │                     │
│  └──────────────────────────────┘                     │
│                                                        │
│  BLACK LOCKERS (8 hours):                             │
│  ┌──────────────────────────────┐                     │
│  │ 🔒 Black Locker #1           │                     │
│  │ Opens in: 3h 45m             │                     │
│  │                   [OPEN-LOCKED] │                     │
│  └──────────────────────────────┘                     │
│                                                        │
└────────────────────────────────────────────────────────┘
```

### 7. INVENTORY - INDEX TAB
```
┌────────────────────────────────────────────────────────┐
│ INVENTORY                              [X]              │
├────────────────────────────────────────────────────────┤
│ [Items]  [Cars]  [Lockers]  [INDEX]                   │
├─────────────���──────────────────────────────────────────┤
│                                                        │
│  COMMON CARS:                                         │
│  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐             │
│  │ 🚗   │  │ 🚗   │  │ 🚗   │  │ 🚗   │             │
│  │ Car X1│ │ Car X2│ │ Car X3│ │ Car X4│            │
│  │ $50   │  │ $75  │  │ $100  │  │ $125  │           │
│  └──────┘  └──────┘  └──────┘  └──────┘             │
│                                                        │
│  UNCOMMON CARS:                                       │
│  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐             │
│  │ 🚗   │  │ 🚗   │  │ 🚗   │  │ 🚗   │             │
│  │ Car X5│ │ Car X6│ │ Car X7│ │ Car X8│            │
│  │ $150  │  │ $200 │  │ $250  │  │ $300  │           │
│  └──────┘  └──────┘  └──────┘  └──────┘             │
│                                                        │
│  [SCROLL DOWN FOR RARE/EPIC/LEGENDARY/SPEC]          │
│                                                        │
└────────────────────────────────────────────────────────┘
```

---

## 📁 FOLDER STRUCTURE

```
Workspace/
├── Camera
├── Terrain
├── Conveyor Central
│   └── Conveyors (x6 physical tents on map)
│       ├── Conveyor 1 (Part with script)
│       ├── Conveyor 2
│       ├── ...
│       └── Conveyor 6
│
├── Events (Teleport button to events)
├── FencesG (Map decoration)
├── Folder (Generic folder)
├── Decorations (Item models)
│   ├── Vase
│   ├── Plant
│   ├── Lamp
│   └── ...
│
├── Map (Main world)
│   ├── Spawn Area
│   ├── Lobby
│   ├── Garages
│   └── Merchant Area
│
├── Bid a Car (World 1) (Main container)
├── Lockers (Physical locker objects on map x3)
│   ├── Locker 1 (Part with interact script)
│   ├── Locker 2
│   └── Locker 3
│
├── GarageRNG (RNG Garage template for bids)
│   ├── Ground
│   ├── Display (Car & decoration spawn points)
│   └── Exit (Teleport back button)
│
├── Garages (Storage for garage variants)
│   ├── Garage200 (BEGINNER template)
│   ├── Garage500 (ADVANCED template)
│   ├── Garage1200 (EXPERT template)
│   └── Garage2500 (CHOSEN template)
│
├── Merchant (NPC shop area)
│   ├── NPC_Merchant (Character model)
│   └── ShopArea
│
├── NPCS (AI Bidders)
│   ├── BotBidder1 (Humanoid, no name above head initially)
│   ├── BotBidder2
│   ├── BotBidder3
│   ├── BotBidder4
│   ├── BotBidder5
│   ├── BotBidder6
│   └── (More as needed)
│
├── ShopNpc (Shop NPC display area)
│
├── SpawnLocation (Player spawn)
│
├── Players (Cloned on join)
│   └── [PlayerName]
│       └── PlayerGui (UI elements)
│
├── StarterGui
│   ├── ScreenGui (Main container)
│   ├── TierSelectionGui
│   ├── BidGui
│   ├── InventoryGui
│   ├── RebirthGui
│   ├── TradeGui
│   ├── PlotGui
│   └── ShopGui
│
├── StarterPack (Tools given at spawn)
│
├── StarterPlayer (Character templates)
│
├── Lighting
├── MaterialService
├── NetworkClient
├── ReplicatedFirst
├── ReplicatedStorage
│   ├── Modules
│   │   ├── DataStoreManager
│   │   ├── PlayerDataManager
│   │   ├── BidEngine
│   │   ├── NPCBidController
│   │   ├── RNGGarageGenerator
│   │   ├── InventoryManager
│   │   ├── PlotManager
│   │   ├── IncomeGenerator
│   │   ├── RebirthManager
│   │   ├── ShopManager
│   │   ├── DiceRNG
│   │   ├── TradeManager
│   │   ├── UIManager
│   │   ├── TeleportManager
│   │   ├── ItemDatabase
│   │   └── Config
│   │
│   ├── Assets
│   │   ├── CarModels
│   │   │   ├── Common (x1-x4)
│   │   │   ├── Uncommon (x5-x8)
│   │   │   ├── Rare
│   │   │   ├── Epic
│   │   │   ├── Legendary
│   │   │   └── SPEC
│   │   │
│   │   ├── Decorations
│   │   │   ├── Vase
│   │   │   ├── Plant
│   │   │   └── ...
│   │   │
│   │   └── UI Images
│   │       ├── CarImages (no background)
│   │       ├── Icons
│   │       └── Buttons
│   │
│   └── Events
│       ├── OnPlayerJoin
│       ├── OnPlayerLeave
│       ├── OnBidStart
│       └── OnMoneyChanged
│
├── ServerScriptService
│   ├── Main (Initializes everything)
│   └── PlayerJoinHandler
│
├── ServerStorage
│   ├── CarDatabase (All car models)
│   ├── DecorationDatabase
│   ├── NPCDatabase
│   └── Templates
│
├── SoundService
├── TextChatService
└── Teams
```

---

## ✅ IMPLEMENTATION CHECKLIST

### Phase 1: Core Infrastructure
- [ ] DataStoreManager (Save/Load)
- [ ] PlayerDataManager (In-memory state)
- [ ] UIManager (Basic screens)
- [ ] TeleportManager (Movement)

### Phase 2: Bid System
- [ ] BidEngine (State machine)
- [ ] NPCBidController (AI)
- [ ] RNGGarageGenerator (Garage creation)
- [ ] BidUI (Modern interface)

### Phase 3: Inventory & Items
- [ ] InventoryManager (Item tracking)
- [ ] ItemDatabase (All item definitions)
- [ ] InventoryUI (4 tabs)
- [ ] Locker system (Time-locked containers)

### Phase 4: Plot & Income
- [ ] PlotManager (Conveyor system)
- [ ] IncomeGenerator (Passive money)
- [ ] Physical plot visualization

### Phase 5: Progression
- [ ] RebirthManager (Milestone system)
- [ ] ShopManager (Dice shop)
- [ ] DiceRNG (NPC generation)

### Phase 6: Trading
- [ ] TradeManager (P2P trading)
- [ ] TradeUI (Modern interface)

### Phase 7: Polish
- [ ] UI animations
- [ ] Sound effects
- [ ] Tutorial system
- [ ] Error handling

---

**Schema Version:** 1.1  
**Last Updated:** May 29, 2026  
**Status:** Ready for development

**Recent Changes:**
- Updated Main Lobby to be physical world with UI buttons (not a pop-up)
- Changed Tier Selection to horizontal scrollable pop-up with single row
- Updated Bid Interface to match Bid Battles POV (first-person with players/bots physical positioning)
- Updated Inventory UI layout to show car images, names, and income only
- Added Index tab properly to inventory structure
- Clarified all UI elements match Bid Battles game design
