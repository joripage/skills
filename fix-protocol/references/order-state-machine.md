# Order State Machine

## OrdStatus (Tag 39) Values

| Value | Name | Description |
|-------|------|-------------|
| 0 | New | Order accepted, working in market |
| 1 | PartiallyFilled | Part of order executed |
| 2 | Filled | Fully executed |
| 3 | DoneForDay | Order done for the day (not fully filled) |
| 4 | Cancelled | Cancelled (may have partial fills) |
| 5 | Replaced | Replaced by cancel/replace (35=G) — FIX 4.2 only |
| 6 | PendingCancel | Cancel request received, awaiting confirmation |
| 7 | Stopped | Guaranteed fill at specified price |
| 8 | Rejected | Order rejected — never entered order book |
| 9 | Suspended | Order suspended |
| A | PendingNew | Order received, awaiting acceptance |
| B | Calculated | Triggered from calculated event |
| C | Expired | TimeInForce expired |
| D | AcceptedForBidding | Accepted for bidding |
| E | PendingReplace | Replace request received, awaiting confirmation |

## ExecType (Tag 150) Values

### FIX 4.4 / 5.0
| Value | Name | When Used |
|-------|------|-----------|
| 0 | New | Order accepted |
| 3 | DoneForDay | No more fills today |
| 4 | Cancelled | Cancel confirmed |
| 5 | Replaced | Replace confirmed |
| 6 | PendingCancel | Cancel request acknowledged |
| 7 | Stopped | Price guaranteed |
| 8 | Rejected | Order rejected |
| 9 | Suspended | Order suspended |
| A | PendingNew | Order received |
| B | Calculated | Triggered |
| C | Expired | TIF expired |
| D | Restated | Order restated (unsolicited) |
| E | PendingReplace | Replace request acknowledged |
| F | Trade | Fill occurred (partial or full) |
| G | TradeCorrect | Trade correction |
| H | TradeCancel | Trade bust/cancel |
| I | OrderStatus | Response to status request |

### FIX 4.2 (different from 4.4+)
| Value | Name | Notes |
|-------|------|-------|
| 0 | New | Same |
| 1 | PartialFill | Use ExecType=F in FIX 4.4+ |
| 2 | Fill | Use ExecType=F in FIX 4.4+ |
| 4 | Cancelled | Same |
| 5 | Replace | Same |
| 8 | Rejected | Same |

**FIX 4.2 also uses ExecTransType (tag 20):** 0=New, 1=Cancel, 2=Correct, 3=Status. This tag is REMOVED in FIX 4.4+.

## State Transition Diagram

```
                    ┌──────────────┐
     35=D ────────►│  PendingNew  │
                    │  (39=A)      │
                    └──────┬───────┘
                           │ accepted
                           ▼
                    ┌──────────────┐
              ┌────│     New       │────────────────┐
              │    │  (39=0)      │                 │
              │    └──────┬───────┘                 │
              │           │ fill                    │
              │           ▼                         │
              │    ┌──────────────┐                 │
              │    │ PartialFill  │──── fill ──┐    │
              │    │  (39=1)      │             │    │
              │    └──────┬───────┘             │    │
              │           │ final fill          │    │
              │           ▼                     ▼    │
              │    ┌──────────────┐      ┌──────────┐
              │    │   Filled     │      │ Filled   │
              │    │  (39=2)      │      │ (39=2)   │
              │    └──────────────┘      └──────────┘
              │           TERMINAL              TERMINAL
              │
   35=F ──────┼───────────────────────────────────┐
              │                                    │
              ▼                                    ▼
       ┌──────────────┐                    ┌──────────────┐
       │PendingCancel │                    │PendingCancel │
       │  (39=6)      │                    │  (39=6)      │
       └──────┬───────┘                    └──────┬───────┘
              │ confirmed                         │ confirmed
              ▼                                    ▼
       ┌──────────────┐                    ┌──────────────┐
       │  Cancelled   │                    │  Cancelled   │
       │  (39=4)      │                    │  (39=4)      │
       └──────────────┘                    └──────────────┘
              TERMINAL                            TERMINAL

   35=G ──────┤
              ▼
       ┌──────────────┐
       │PendingReplace│
       │  (39=E)      │
       └──────┬───────┘
              │ confirmed
              ▼
       ┌──────────────┐
       │  New (replaced)│ ← new OrderID, new ClOrdID
       │  (39=0, 150=5)│
       └──────────────┘

   Expired (TIF):
       ┌──────────────┐
       │   Expired    │
       │  (39=C)      │ ← TERMINAL
       └──────────────┘

   Rejected (never entered book):
       ┌──────────────┐
       │  Rejected    │
       │  (39=8)      │ ← TERMINAL (from PendingNew or directly)
       └──────────────┘
```

## Valid State Transitions

| From | To | Trigger |
|------|----|---------|
| — | PendingNew (A) | NewOrderSingle received |
| — | Rejected (8) | NewOrderSingle rejected immediately |
| PendingNew (A) | New (0) | Order accepted |
| PendingNew (A) | Rejected (8) | Order rejected after pending |
| New (0) | PartiallyFilled (1) | Partial execution |
| New (0) | Filled (2) | Full execution |
| New (0) | PendingCancel (6) | Cancel request received |
| New (0) | PendingReplace (E) | Replace request received |
| New (0) | Cancelled (4) | Cancelled (immediate) |
| New (0) | Expired (C) | TIF expired |
| New (0) | Suspended (9) | Halted/suspended |
| New (0) | DoneForDay (3) | Market close |
| PartiallyFilled (1) | PartiallyFilled (1) | Another partial fill |
| PartiallyFilled (1) | Filled (2) | Final fill |
| PartiallyFilled (1) | PendingCancel (6) | Cancel request |
| PartiallyFilled (1) | PendingReplace (E) | Replace request |
| PartiallyFilled (1) | Cancelled (4) | Cancelled with partial |
| PartiallyFilled (1) | Expired (C) | TIF expired with partial |
| PendingCancel (6) | Cancelled (4) | Cancel confirmed |
| PendingCancel (6) | New (0) | Cancel rejected → back to original |
| PendingCancel (6) | PartiallyFilled (1) | Fill during pending cancel |
| PendingCancel (6) | Filled (2) | Filled before cancel |
| PendingReplace (E) | New (0) | Replace confirmed (new values) |
| PendingReplace (E) | PartiallyFilled (1) | Fill during pending replace |
| PendingReplace (E) | Filled (2) | Filled before replace |
| Suspended (9) | New (0) | Resumed |
| Suspended (9) | Cancelled (4) | Cancelled while suspended |

## Terminal States (no further transitions)

- **Filled (2)** — fully executed
- **Cancelled (4)** — cancelled (with or without partial fills)
- **Rejected (8)** — never entered order book
- **Expired (C)** — time-in-force expired
- **DoneForDay (3)** — typically terminal for the session

## Quantity Rules

```
INVARIANT: CumQty(14) + LeavesQty(151) = OrderQty(38)

On New:       CumQty=0,    LeavesQty=OrderQty, LastQty=0
On Partial:   CumQty+=fill, LeavesQty-=fill,    LastQty=fill
On Fill:      CumQty=OrderQty, LeavesQty=0,      LastQty=final_fill
On Cancel:    CumQty=partial, LeavesQty=0,        LastQty=0
On Reject:    CumQty=0,    LeavesQty=0,           LastQty=0
On Replace:   LeavesQty recalculated from new OrderQty - CumQty
```

## ClOrdID Chain for Cancel/Replace

```
NewOrderSingle:  ClOrdID = "ORD001"
                 → OrderID = "EXCH-123" (assigned by exchange)

CancelReplace:   ClOrdID = "ORD002", OrigClOrdID = "ORD001"
                 → OrderID = "EXCH-123" (same)
                 → ExecReport: ClOrdID="ORD002", OrigClOrdID="ORD001"

Cancel:          ClOrdID = "ORD003", OrigClOrdID = "ORD002"
                 → OrderID = "EXCH-123" (same)

CancelReject:    ClOrdID = "ORD003", OrigClOrdID = "ORD002"
                 → OrdStatus = current status of original order
```

**Rule:** OrigClOrdID always points to the MOST RECENT ClOrdID in the chain, NOT the original.
