# FIX Version Differences

## FIX 4.2 vs FIX 4.4 vs FIX 5.0 SP2

### Major Structural Changes

| Feature | FIX 4.2 | FIX 4.4 | FIX 5.0 SP2 |
|---------|---------|---------|-------------|
| BeginString (8) | FIX.4.2 | FIX.4.4 | FIXT.1.1 |
| Transport/App separation | No | No | Yes (FIXT.1.1 transport) |
| ExecTransType (20) | Required | Removed | Removed |
| ExecType fill values | 1=Partial, 2=Fill | F=Trade | F=Trade |
| OrdStatus=5 (Replaced) | Yes | Removed | Removed |
| HandlInst (21) in D | Required | Optional | Optional |
| Parties component | No | Yes | Yes (enhanced) |
| TradingSessionID | Simple | Enhanced | Enhanced |

### ExecType Differences (Critical)

**FIX 4.2:**
```
ExecTransType (20) + ExecType (150):
  20=0 (New) + 150=0 → New order acknowledged
  20=0 (New) + 150=1 → Partial fill
  20=0 (New) + 150=2 → Full fill
  20=0 (New) + 150=4 → Cancelled
  20=0 (New) + 150=5 → Replaced
  20=0 (New) + 150=8 → Rejected
  20=1 (Cancel) + 150=1 → Cancel partial fill (trade bust)
  20=1 (Cancel) + 150=2 → Cancel full fill (trade bust)
  20=2 (Correct) + 150=1 → Correct partial fill
  20=3 (Status) + 150=0 → Status response
```

**FIX 4.4 / 5.0:**
```
ExecType (150) only — ExecTransType (20) REMOVED:
  150=0 → New order acknowledged
  150=F → Trade (partial or full — check OrdStatus for which)
  150=4 → Cancelled
  150=5 → Replaced
  150=8 → Rejected
  150=G → Trade Correct (replaces ExecTransType=2)
  150=H → Trade Cancel (replaces ExecTransType=1)
  150=I → Order Status (replaces ExecTransType=3)
```

**Migration rule:** When migrating from FIX 4.2 to 4.4+:
- Remove ExecTransType (20) entirely
- Replace ExecType 1/2 with F
- Use OrdStatus to distinguish partial vs full fill
- Add ExecType G/H/I for corrections, busts, status

### Order Replace Handling

**FIX 4.2:**
- OrdStatus (39) = 5 (Replaced) is valid
- Replaced order gets final ExecReport with OrdStatus=5

**FIX 4.4+:**
- OrdStatus=5 is REMOVED
- Replace confirmed via ExecType=5, but OrdStatus shows the NEW state (0=New)
- Original order is implicitly closed

### NewOrderSingle (35=D) Differences

| Field | FIX 4.2 | FIX 4.4 | FIX 5.0 |
|-------|---------|---------|---------|
| HandlInst (21) | Required | Optional | Optional |
| ExecInst (18) | Multi-value string | Multi-value string | Multi-value string |
| Account (1) | String | String | String |
| Parties block | N/A | Optional (453+) | Optional (453+) |

### Identification Differences

**FIX 4.2:**
```
Identification via simple tags:
  1  = Account
  109 = ClientID
  76  = ExecBroker
```

**FIX 4.4+:**
```
Identification via Parties component (repeating group):
  453 = NoPartyIDs
  →448 = PartyID
  →447 = PartyIDSource
  →452 = PartyRole
```

### BusinessMessageReject Differences

| Feature | FIX 4.2 | FIX 4.4 | FIX 5.0 |
|---------|---------|---------|---------|
| 35=j available | Yes | Yes | Yes |
| BusinessRejectReason values | 0-7 | 0-7,18 | 0-7,18 |
| RefMsgType (372) | Required | Required | Required |

### Session Layer (FIXT.1.1 — FIX 5.0 only)

FIX 5.0 introduces transport independence via FIXT.1.1:
- BeginString = FIXT.1.1 (not FIX.5.0)
- ApplVerID (1128) specifies application version
- DefaultApplVerID (1137) in Logon message
- Session messages follow FIXT.1.1 rules
- Application messages follow FIX 5.0 rules
- Allows different app versions on same session

### Message Type Additions by Version

**Added in FIX 4.4 (not in 4.2):**
- 35=AC (Position Maintenance Request)
- 35=AD (Position Maintenance Report)
- 35=AM (Request for Positions)
- 35=AN (Request for Positions Ack)
- 35=AO (Position Report)
- 35=q (Order Mass Cancel Request)
- 35=r (Order Mass Cancel Report)
- Cross Orders (35=s, t, u)
- Trade Capture (35=AE, AR)

**Added in FIX 5.0:**
- 35=BE (User Request)
- 35=BF (User Response)
- 35=BJ (User Notification)
- Party details/actions
- Enhanced allocation messages

### OnixS FIX Engine Notes

OnixS FIX Engine (onixs.biz) specific considerations:
- Supports FIX 4.0 through FIX 5.0 SP2
- Handles session-level protocol automatically (heartbeat, sequence reset, resend)
- Dictionary-based validation — custom fields via FIX dictionary XML
- Session storage options: file-based, memory-based
- Thread-safety model varies by language binding (C++, .NET, Java)
- Custom message types require dictionary extension
- Warm standby / failover support in enterprise editions
- Performance tuning: zero-copy parsing, pre-allocated buffers
