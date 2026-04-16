# Required Fields per Message Type

## Standard Header (ALL messages)

| Tag | Name | Required | Notes |
|-----|------|----------|-------|
| 8 | BeginString | Y | FIX.4.2 / FIX.4.4 / FIXT.1.1 |
| 9 | BodyLength | Y | Auto-calculated |
| 35 | MsgType | Y | Message type identifier |
| 49 | SenderCompID | Y | Sender identification |
| 56 | TargetCompID | Y | Target identification |
| 34 | MsgSeqNum | Y | Message sequence number |
| 52 | SendingTime | Y | UTC timestamp |

## Standard Trailer (ALL messages)

| Tag | Name | Required |
|-----|------|----------|
| 10 | CheckSum | Y |

---

## Logon (35=A)

| Tag | Name | Required | Notes |
|-----|------|----------|-------|
| 98 | EncryptMethod | Y | 0=None typically |
| 108 | HeartBtInt | Y | Seconds |
| 141 | ResetSeqNumFlag | N | Y to reset sequences to 1 |
| 553 | Username | N | If authentication required |
| 554 | Password | N | If authentication required |

## Heartbeat (35=0)

| Tag | Name | Required | Notes |
|-----|------|----------|-------|
| 112 | TestReqID | C | Required if responding to TestRequest |

## TestRequest (35=1)

| Tag | Name | Required |
|-----|------|----------|
| 112 | TestReqID | Y |

## ResendRequest (35=2)

| Tag | Name | Required | Notes |
|-----|------|----------|-------|
| 7 | BeginSeqNo | Y | First sequence to resend |
| 16 | EndSeqNo | Y | Last sequence (0=infinity) |

## Reject (35=3)

| Tag | Name | Required | Notes |
|-----|------|----------|-------|
| 45 | RefSeqNum | Y | SeqNum of rejected message |
| 371 | RefTagID | N | Tag that caused rejection |
| 372 | RefMsgType | N | MsgType of rejected message |
| 373 | SessionRejectReason | N | Reason code |
| 58 | Text | N | Human-readable explanation |

## SequenceReset (35=4)

| Tag | Name | Required | Notes |
|-----|------|----------|-------|
| 36 | NewSeqNo | Y | New sequence number |
| 123 | GapFillFlag | N | Y=GapFill, N/absent=Reset |

## Logout (35=5)

| Tag | Name | Required |
|-----|------|----------|
| 58 | Text | N |

---

## NewOrderSingle (35=D)

| Tag | Name | Required | Notes |
|-----|------|----------|-------|
| 11 | ClOrdID | Y | Unique client order ID |
| 1 | Account | N | Trading account |
| 21 | HandlInst | Y (4.2) / N (4.4+) | 1=Auto, 2=Manual, 3=Auto-NoIntervention |
| 55 | Symbol | Y | Instrument symbol |
| 54 | Side | Y | 1=Buy, 2=Sell |
| 60 | TransactTime | Y | Order creation time |
| 38 | OrderQty | Y* | *Or CashOrderQty |
| 40 | OrdType | Y | 1=Market, 2=Limit, etc. |
| 44 | Price | C | Required for Limit orders |
| 99 | StopPx | C | Required for Stop orders |
| 59 | TimeInForce | N | Default=Day |
| 15 | Currency | N | Currency code |
| 18 | ExecInst | N | Execution instructions |
| 432 | ExpireDate | C | Required if TIF=GTD |

## ExecutionReport (35=8)

| Tag | Name | Required | Notes |
|-----|------|----------|-------|
| 37 | OrderID | Y | Sell-side assigned ID |
| 11 | ClOrdID | Y | From order |
| 41 | OrigClOrdID | C | If cancel/replace response |
| 17 | ExecID | Y | Unique execution ID |
| 20 | ExecTransType | Y (4.2 only) | 0=New, 1=Cancel, 2=Correct, 3=Status |
| 150 | ExecType | Y | Execution type |
| 39 | OrdStatus | Y | Current order status |
| 55 | Symbol | Y | Instrument |
| 54 | Side | Y | Side |
| 38 | OrderQty | Y | Order quantity |
| 44 | Price | C | If Limit order |
| 151 | LeavesQty | Y | Remaining quantity |
| 14 | CumQty | Y | Total filled |
| 6 | AvgPx | Y | Average fill price |
| 32 | LastQty | C | Qty of last fill (if fill) |
| 31 | LastPx | C | Price of last fill (if fill) |
| 60 | TransactTime | N | Timestamp |
| 103 | OrdRejReason | C | If rejected |
| 58 | Text | N | Human-readable text |

## OrderCancelRequest (35=F)

| Tag | Name | Required | Notes |
|-----|------|----------|-------|
| 11 | ClOrdID | Y | New unique ID for cancel |
| 41 | OrigClOrdID | Y | ClOrdID of order to cancel |
| 37 | OrderID | N | Exchange order ID (if known) |
| 55 | Symbol | Y | Must match original |
| 54 | Side | Y | Must match original |
| 38 | OrderQty | Y (4.2) / N (4.4+) | Original order qty |
| 60 | TransactTime | Y | Cancel request time |

## OrderCancelReplaceRequest (35=G)

| Tag | Name | Required | Notes |
|-----|------|----------|-------|
| 11 | ClOrdID | Y | New unique ID for replace |
| 41 | OrigClOrdID | Y | ClOrdID of order to replace |
| 37 | OrderID | N | Exchange order ID |
| 55 | Symbol | Y | Must match original |
| 54 | Side | Y | Must match original |
| 38 | OrderQty | Y | New quantity |
| 40 | OrdType | Y | Order type |
| 44 | Price | C | New price (if Limit) |
| 60 | TransactTime | Y | Request time |
| 21 | HandlInst | Y (4.2) / N (4.4+) | Handling instruction |

## OrderCancelReject (35=9)

| Tag | Name | Required | Notes |
|-----|------|----------|-------|
| 37 | OrderID | Y | Exchange order ID |
| 11 | ClOrdID | Y | From cancel/replace request |
| 41 | OrigClOrdID | Y | From cancel/replace request |
| 39 | OrdStatus | Y | Current order status |
| 434 | CxlRejResponseTo | Y | 1=Cancel, 2=CancelReplace |
| 102 | CxlRejReason | N | Reason code |
| 58 | Text | N | Human-readable |

## BusinessMessageReject (35=j)

| Tag | Name | Required | Notes |
|-----|------|----------|-------|
| 45 | RefSeqNum | Y | SeqNum of rejected message |
| 372 | RefMsgType | Y | MsgType of rejected message |
| 380 | BusinessRejectReason | Y | Reason code |
| 58 | Text | N | Human-readable |
| 379 | BusinessRejectRefID | N | Reference ID from rejected msg |

## MarketDataRequest (35=V)

| Tag | Name | Required | Notes |
|-----|------|----------|-------|
| 262 | MDReqID | Y | Unique request ID |
| 263 | SubscriptionRequestType | Y | 0=Snapshot, 1=Subscribe, 2=Unsubscribe |
| 264 | MarketDepth | Y | 0=FullBook, 1=Top, N=levels |
| 267 | NoMDEntryTypes | Y | Repeating group count |
| →269 | MDEntryType | Y | 0=Bid, 1=Offer, 2=Trade |
| 146 | NoRelatedSym | Y | Repeating group count |
| →55 | Symbol | Y | Instrument |

## MarketDataRequestReject (35=Y)

| Tag | Name | Required | Notes |
|-----|------|----------|-------|
| 262 | MDReqID | Y | From request |
| 281 | MDReqRejReason | N | Reason code |
| 58 | Text | N | Human-readable |
