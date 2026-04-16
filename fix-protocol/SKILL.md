---
name: fix-protocol
description: "FIX (Financial Information eXchange) protocol expert. Use when working with FIX messages, gateway implementation, order management, session management, or debugging FIX flows. Covers FIX 4.2/4.4/5.0."
argument-hint: "[question or task about FIX protocol]"
metadata:
  author: joripage 
  version: "1.0.0"
---

# FIX Protocol Expert

Expert guidance for FIX (Financial Information eXchange) protocol implementation, debugging, and validation.

**Reference:** [OnixS FIX Engine](https://www.onixs.biz) | FIX 4.2 / 4.4 / 5.0 SP2

## When to Activate

- Implementing/modifying FIX message handlers
- Debugging FIX session or order issues
- Validating FIX message flows and state transitions
- Building FIX gateways, OMS, or matching engines
- Reviewing FIX-related code for correctness

## Quick Reference: Message Flow Matrix

### Order Entry Flows

| Request (Initiator) | Tag 35 | Success Response | Reject Response | Reject Tag 35 |
|---------------------|--------|------------------|-----------------|----------------|
| NewOrderSingle | D | ExecutionReport (New) | ExecutionReport (Rejected) | 8 |
| OrderCancelRequest | F | ExecutionReport (Cancelled) | OrderCancelReject | 9 |
| OrderCancelReplaceRequest | G | ExecutionReport (Replaced) | OrderCancelReject | 9 |
| OrderStatusRequest | H | ExecutionReport | BusinessMessageReject | j |
| MassOrderCancelRequest | q | OrderMassCancelReport | OrderMassCancelReport (rejected) | r |

### Market Data Flows

| Request | Tag 35 | Success Response | Reject Response | Reject Tag 35 |
|---------|--------|------------------|-----------------|----------------|
| MarketDataRequest | V | MarketDataSnapshotFullRefresh (W) / MarketDataIncrementalRefresh (X) | MarketDataRequestReject | Y |
| SecurityListRequest | x | SecurityList | SecurityList (rejected) | y |
| SecurityDefinitionRequest | c | SecurityDefinition | SecurityDefinition (rejected) | d |

### Quote Flows

| Request | Tag 35 | Success Response | Reject Response | Reject Tag 35 |
|---------|--------|------------------|-----------------|----------------|
| QuoteRequest | R | Quote (S) | QuoteRequestReject | AG |
| QuoteCancel | Z | — | BusinessMessageReject | j |

### Post-Trade Flows

| Request | Tag 35 | Success Response | Reject Response | Reject Tag 35 |
|---------|--------|------------------|-----------------|----------------|
| AllocationInstruction | J | AllocationInstructionAck | AllocationInstructionAck (rejected) | P |
| TradeCaptureReport | AE | TradeCaptureReportAck | TradeCaptureReportAck (rejected) | AR |

### Session-Level Flows

| Request | Tag 35 | Response | Notes |
|---------|--------|----------|-------|
| Logon | A | Logon (A) | Mutual exchange; initiator sends first |
| Logout | 5 | Logout (5) | Graceful disconnect |
| Heartbeat | 0 | — | No response needed (unless TestReqID present) |
| TestRequest | 1 | Heartbeat (0) | Response MUST include TestReqID (112) from request |
| ResendRequest | 2 | SequenceReset-GapFill (4) or replayed messages | Resend range [BeginSeqNo..EndSeqNo] |
| SequenceReset | 4 | — | GapFill or Reset mode |
| Reject | 3 | — | Session-level reject, no response |

---

## CRITICAL RULES

### Rule 1: Session Reject (35=3) vs Business Reject (35=j)

```
35=3 (Reject) — SESSION-level problems
├── Malformed message (bad syntax, missing required tags)
├── Invalid tag number
├── Tag without value
├── Incorrect data format
├── Tag appears outside valid context
├── Required tag missing (header/trailer)
├── Undefined tag
├── CompID mismatch
└── SendingTime accuracy problem

35=j (BusinessMessageReject) — APPLICATION-level problems
├── Unknown/unsupported MsgType
├── Application not available
├── Conditionally required field missing (business logic)
├── Not authorized for message type
├── Deliverto firm not available
└── Message received for unsupported function
```

**Decision rule:** If the message is structurally valid but cannot be processed for business reasons → use 35=j. If the message structure itself is broken → use 35=3.

**EXCEPTION:** Order-related messages (D, F, G) have their OWN reject mechanisms (35=8, 35=9). Do NOT use 35=j for order rejects — use ExecutionReport or OrderCancelReject.

### Rule 2: Reject Response by Message Type

```
Incoming 35=D (NewOrderSingle)
├── Session error → 35=3 (Reject)
├── Business error (unknown symbol, invalid account, etc.)
│   → 35=8 (ExecutionReport) with:
│     ├── OrdStatus (39) = 8 (Rejected)
│     ├── ExecType (150) = 8 (Rejected)
│     ├── OrdRejReason (103) = reason code
│     └── Text (58) = human-readable reason
└── NEVER use 35=j for D rejects

Incoming 35=F (OrderCancelRequest)
├── Session error → 35=3
├── Cancel rejected (order not found, too late, etc.)
│   → 35=9 (OrderCancelReject) with:
│     ├── CxlRejResponseTo (434) = 1 (Order Cancel Request)
│     ├── CxlRejReason (102) = reason code
│     ├── OrdStatus (39) = current order status
│     └── ClOrdID (11) + OrigClOrdID (41)
└── NEVER use 35=j for F rejects

Incoming 35=G (OrderCancelReplaceRequest)
├── Session error → 35=3
├── Replace rejected (order not found, invalid new values, etc.)
│   → 35=9 (OrderCancelReject) with:
│     ├── CxlRejResponseTo (434) = 2 (Order Cancel/Replace Request)
│     ├── CxlRejReason (102) = reason code
│     └── OrdStatus (39) = current order status
└── NEVER use 35=j for G rejects

Incoming 35=V (MarketDataRequest)
├── Session error → 35=3
├── Business error → 35=Y (MarketDataRequestReject) with:
│     ├── MDReqID (262) = from request
│     └── MDReqRejReason (281) = reason code
└── NEVER use 35=j for V rejects

Incoming 35=H (OrderStatusRequest)
├── Session error → 35=3
├── Order found → 35=8 (ExecutionReport) with current state
└── Order not found → 35=j (BusinessMessageReject)

Any unknown/unsupported MsgType
└── 35=j (BusinessMessageReject) with:
      ├── RefMsgType (372) = the unknown MsgType
      └── BusinessRejectReason (380) = 3 (Unsupported Message Type)
```

### Rule 3: Order State Machine

See `references/order-state-machine.md` for full state diagram.

### Rule 4: Sequence Number Management

See `references/session-management.md` for sequence number rules.

### Rule 5: Required Fields Validation

See `references/required-fields.md` for per-message-type required fields.

### Rule 6: Common Error Codes

See `references/error-codes.md` for OrdRejReason, CxlRejReason, BusinessRejectReason, SessionRejectReason values.

---

## Implementation Checklist

### Session Layer Checklist

- [ ] Logon handshake with correct HeartBtInt (108) negotiation
- [ ] Heartbeat monitoring — send TestRequest (35=1) after HeartBtInt+grace period with no message
- [ ] Disconnect if no Heartbeat response within HeartBtInt after TestRequest
- [ ] Sequence number persistence across sessions
- [ ] ResendRequest handling for gaps (35=2, BeginSeqNo/EndSeqNo)
- [ ] SequenceReset-GapFill (35=4, GapFillFlag=Y) for admin messages during resend
- [ ] SequenceReset-Reset (35=4, GapFillFlag=N) for emergency reset
- [ ] Reject (35=3) for malformed messages with correct RefSeqNum (45)
- [ ] MsgSeqNum (34) validation — reject if too low (except PossDupFlag), request resend if gap
- [ ] PossDupFlag (43) and PossResend (97) handling
- [ ] SendingTime (52) accuracy validation (±120 seconds default)
- [ ] CompID validation (SenderCompID↔TargetCompID swap check)
- [ ] Graceful Logout (35=5) with Text explanation
- [ ] Handle unexpected disconnect — persist sequence numbers

### Order Management Checklist

- [ ] ClOrdID (11) uniqueness validation per session
- [ ] OrigClOrdID (41) chain for cancel/replace → links back to previous ClOrdID
- [ ] OrderID (37) assigned by sell-side, returned in all ExecutionReports
- [ ] OrdStatus (39) state machine enforced — no invalid transitions
- [ ] ExecType (150) matches the action taken
- [ ] CumQty (14) + LeavesQty (151) + LastQty (32) consistency:
  - CumQty = sum of all fills
  - LeavesQty = OrderQty - CumQty (0 when fully filled or cancelled)
  - CumQty + LeavesQty ≤ OrderQty
- [ ] Price validation per OrdType (40): Market orders ignore price, Limit orders require it
- [ ] Side (54) validation: 1=Buy, 2=Sell, 5=SellShort, etc.
- [ ] TimeInForce (59) handling: DAY, GTC, IOC, FOK, GTD with ExpireDate/Time
- [ ] Reject with OrdRejReason (103) for invalid orders
- [ ] Handle cancel of partially filled order — LeavesQty=0, CumQty=partial amount

### Market Data Checklist

- [ ] MDReqID (262) tracking for subscribe/unsubscribe
- [ ] Handle subscription vs snapshot (SubscriptionRequestType 263: 0=Snapshot, 1=Subscribe, 2=Unsubscribe)
- [ ] MarketDataRequestReject (35=Y) with MDReqRejReason (281) for failures
- [ ] MDEntryType (269) validation: 0=Bid, 1=Offer, 2=Trade, etc.
- [ ] Handle full refresh (35=W) vs incremental (35=X)

### Security & Validation Checklist

- [ ] Never log passwords (tag 554) or raw credentials
- [ ] Validate all enum fields against allowed values
- [ ] Protect against message injection (validate CheckSum tag 10)
- [ ] Rate limiting on order submission
- [ ] Duplicate ClOrdID detection and rejection
- [ ] Fat finger checks (price/qty limits)
- [ ] Account (1) and Symbol (55) validation against allowed lists

---

## Tag Reference (Most Used)

| Tag | Name | Description |
|-----|------|-------------|
| 8 | BeginString | FIX version (FIX.4.2, FIX.4.4, FIXT.1.1) |
| 9 | BodyLength | Message body length |
| 10 | CheckSum | 3-char checksum |
| 11 | ClOrdID | Client order ID (unique per session) |
| 14 | CumQty | Cumulative filled quantity |
| 15 | Currency | Currency code |
| 17 | ExecID | Unique execution ID |
| 20 | ExecTransType | FIX 4.2 only: 0=New, 1=Cancel, 2=Correct, 3=Status |
| 32 | LastQty | Quantity of last fill |
| 34 | MsgSeqNum | Message sequence number |
| 35 | MsgType | Message type identifier |
| 37 | OrderID | Sell-side assigned order ID |
| 38 | OrderQty | Order quantity |
| 39 | OrdStatus | Current order status |
| 40 | OrdType | Order type (1=Market, 2=Limit, etc.) |
| 41 | OrigClOrdID | Original ClOrdID for cancel/replace |
| 43 | PossDupFlag | Possible duplicate (Y/N) |
| 44 | Price | Order price |
| 45 | RefSeqNum | Referenced sequence number (in Reject) |
| 49 | SenderCompID | Sender identifier |
| 52 | SendingTime | Message send time |
| 54 | Side | 1=Buy, 2=Sell, 5=SellShort |
| 55 | Symbol | Instrument symbol |
| 56 | TargetCompID | Target identifier |
| 58 | Text | Free-form text |
| 59 | TimeInForce | 0=Day, 1=GTC, 3=IOC, 4=FOK, 6=GTD |
| 103 | OrdRejReason | Order reject reason code |
| 108 | HeartBtInt | Heartbeat interval (seconds) |
| 150 | ExecType | Execution report type |
| 151 | LeavesQty | Remaining open quantity |
| 371 | RefTagID | Tag that caused reject |
| 372 | RefMsgType | MsgType that caused reject |
| 373 | SessionRejectReason | Reason for session reject (35=3) |
| 380 | BusinessRejectReason | Reason for business reject (35=j) |
| 434 | CxlRejResponseTo | 1=Cancel, 2=Cancel/Replace |

---

## Usage Examples

**Ask questions:**
```
/ck:fix-protocol "What reject should I send for an invalid NewOrderSingle?"
/ck:fix-protocol "Explain the cancel/replace flow for 35=G"
/ck:fix-protocol "When should I use 35=j vs 35=3?"
```

**Implementation help:**
```
/ck:fix-protocol "Review my ExecutionReport builder for correctness"
/ck:fix-protocol "Help implement ResendRequest handler"
/ck:fix-protocol "Validate my order state machine transitions"
```

## Process

1. **Identify** the FIX message type(s) involved
2. **Consult** the flow matrix above and reference files
3. **Validate** against the checklist for the specific flow
4. **Verify** field requirements and enum values from references
5. **Check** version differences if targeting specific FIX version

## References

Load as needed — keep main context clean:
- `references/order-state-machine.md` — OrdStatus/ExecType transitions
- `references/session-management.md` — Logon, heartbeat, sequence numbers, resend
- `references/required-fields.md` — Required tags per message type
- `references/error-codes.md` — All reject reason codes and enum values
- `references/fix-version-differences.md` — FIX 4.2 vs 4.4 vs 5.0 differences
