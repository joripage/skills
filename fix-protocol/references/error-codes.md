# FIX Error Codes & Enum Values

## SessionRejectReason (Tag 373) — used in Reject (35=3)

| Value | Name | Description |
|-------|------|-------------|
| 0 | InvalidTagNumber | Tag not defined for this message type |
| 1 | RequiredTagMissing | Required tag missing |
| 2 | TagNotDefinedForMsgType | Tag not valid for this message type |
| 3 | UndefinedTag | Tag not recognized |
| 4 | TagSpecifiedWithoutValue | Tag present but empty value |
| 5 | ValueOutOfRange | Value not in valid range for tag |
| 6 | IncorrectDataFormatForValue | Wrong data type (e.g., letters in int field) |
| 7 | DecryptionProblem | Decryption issue |
| 8 | SignatureProblem | Signature issue |
| 9 | CompIDProblem | SenderCompID/TargetCompID mismatch |
| 10 | SendingTimeAccuracyProblem | SendingTime too far from current time |
| 11 | InvalidMsgType | MsgType not valid |
| 12 | XMLValidationError | XML body validation failed |
| 13 | TagAppearsMoreThanOnce | Duplicate tag in message |
| 14 | TagSpecifiedOutOfRequiredOrder | Tag in wrong position |
| 15 | RepeatingGroupFieldsOutOfOrder | Group fields misordered |
| 16 | IncorrectNumInGroupCountForRepeatingGroup | Group count doesn't match entries |
| 17 | NonDataValueIncludesFieldDelimiter | SOH in non-data field |
| 99 | Other | Other reason (use Text tag 58) |

## BusinessRejectReason (Tag 380) — used in BusinessMessageReject (35=j)

| Value | Name | Description |
|-------|------|-------------|
| 0 | Other | Other reason |
| 1 | UnknownID | Unknown identifier |
| 2 | UnknownSecurity | Unknown security/instrument |
| 3 | UnsupportedMessageType | MsgType recognized but not supported |
| 4 | ApplicationNotAvailable | Application temporarily unavailable |
| 5 | ConditionallyRequiredFieldMissing | Business-required field missing |
| 6 | NotAuthorized | Not authorized for this message type |
| 7 | DeliverToFirmNotAvailable | DeliverTo firm unreachable |
| 18 | InvalidPriceIncrement | Price doesn't match tick size |

## OrdRejReason (Tag 103) — used in ExecutionReport (35=8) for order rejects

| Value | Name | Description |
|-------|------|-------------|
| 0 | BrokerExchangeOption | Broker/exchange option |
| 1 | UnknownSymbol | Symbol not found |
| 2 | ExchangeClosed | Exchange not open |
| 3 | OrderExceedsLimit | Order exceeds position/credit limit |
| 4 | TooLateToEnter | Past cutoff time |
| 5 | UnknownOrder | Order not found (for status requests) |
| 6 | DuplicateOrder | Duplicate ClOrdID |
| 7 | DuplicateVerballyRcvd | Duplicate of verbal order |
| 8 | StaleOrder | Order too old |
| 9 | TradeAlongRequired | Trade along required |
| 10 | InvalidInvestorID | Invalid investor/account |
| 11 | UnsupportedOrderCharacteristic | Unsupported order feature |
| 13 | IncorrectQuantity | Invalid quantity |
| 14 | IncorrectAllocatedQuantity | Allocation qty error |
| 15 | UnknownAccount | Account not found |
| 18 | InvalidPriceIncrement | Tick size violation |
| 99 | Other | Other (see Text) |

## CxlRejReason (Tag 102) — used in OrderCancelReject (35=9)

| Value | Name | Description |
|-------|------|-------------|
| 0 | TooLateToCancel | Order already filled or about to fill |
| 1 | UnknownOrder | Order not found (OrigClOrdID mismatch) |
| 2 | BrokerExchangeOption | Broker/exchange prevents cancel |
| 3 | AlreadyPendingCancel | Cancel already in progress |
| 4 | UnableToProcessMassCancel | Mass cancel rejected |
| 5 | OrigOrdModTimeMismatch | Stale cancel request |
| 6 | DuplicateClOrdID | ClOrdID already used |
| 99 | Other | Other (see Text) |

## CxlRejResponseTo (Tag 434)

| Value | Name |
|-------|------|
| 1 | OrderCancelRequest (35=F) |
| 2 | OrderCancelReplaceRequest (35=G) |

## MDReqRejReason (Tag 281) — used in MarketDataRequestReject (35=Y)

| Value | Name | Description |
|-------|------|-------------|
| 0 | UnknownSymbol | Symbol not found |
| 1 | DuplicateMDReqID | MDReqID already in use |
| 2 | InsufficientBandwidth | Too many subscriptions |
| 3 | InsufficientPermissions | Not authorized |
| 4 | UnsupportedSubscriptionRequestType | Invalid SubscriptionRequestType |
| 5 | UnsupportedMarketDepth | MarketDepth not available |
| 6 | UnsupportedMDUpdateType | MDUpdateType not supported |
| 7 | UnsupportedAggregatedBook | AggregatedBook not supported |
| 8 | UnsupportedMDEntryType | MDEntryType not supported |
| C | UnsupportedOpenCloseSettlFlag | OpenCloseSettl not supported |
| D | UnsupportedScope | Scope not supported |

## OrdStatus (Tag 39) Values

| Value | Name |
|-------|------|
| 0 | New |
| 1 | PartiallyFilled |
| 2 | Filled |
| 3 | DoneForDay |
| 4 | Cancelled |
| 5 | Replaced (FIX 4.2 only) |
| 6 | PendingCancel |
| 7 | Stopped |
| 8 | Rejected |
| 9 | Suspended |
| A | PendingNew |
| B | Calculated |
| C | Expired |
| D | AcceptedForBidding |
| E | PendingReplace |

## OrdType (Tag 40) Values

| Value | Name | Price Required? |
|-------|------|-----------------|
| 1 | Market | No |
| 2 | Limit | Yes (tag 44) |
| 3 | Stop | Yes (StopPx tag 99) |
| 4 | StopLimit | Yes (44 + 99) |
| 5 | MarketOnClose | No |
| 6 | WithOrWithout | Optional |
| 7 | LimitOrBetter | Yes |
| 8 | LimitWithOrWithout | Yes |
| 9 | OnBasis | No |
| P | Pegged | No (PegOffsetValue) |

## Side (Tag 54) Values

| Value | Name |
|-------|------|
| 1 | Buy |
| 2 | Sell |
| 3 | BuyMinus |
| 4 | SellPlus |
| 5 | SellShort |
| 6 | SellShortExempt |
| 7 | Undisclosed |
| 8 | Cross |
| 9 | CrossShort |

## TimeInForce (Tag 59) Values

| Value | Name | Notes |
|-------|------|-------|
| 0 | Day | Valid for trading day only (DEFAULT) |
| 1 | GoodTillCancel (GTC) | Valid until explicitly cancelled |
| 2 | AtTheOpening (OPG) | Execute at open or cancel |
| 3 | ImmediateOrCancel (IOC) | Fill what you can, cancel rest |
| 4 | FillOrKill (FOK) | Fill entirely or cancel entirely |
| 5 | GoodTillCrossing (GTX) | Valid until crossing session |
| 6 | GoodTillDate (GTD) | Requires ExpireDate (432) or ExpireTime (126) |
| 7 | AtTheClose | Execute at close |

## ExecType (Tag 150) — see order-state-machine.md for full details

## MassCancelRejectReason (Tag 532)

| Value | Name |
|-------|------|
| 0 | MassCancelNotSupported |
| 1 | InvalidOrUnknownSecurity |
| 2 | InvalidOrUnknownUnderlying |
| 3 | InvalidOrUnknownProduct |
| 4 | InvalidOrUnknownCFICode |
| 5 | InvalidOrUnknownSecurityType |
| 6 | InvalidOrUnknownTradingSession |
| 99 | Other |

## SecurityRequestResult (Tag 560)

| Value | Name |
|-------|------|
| 0 | ValidRequest |
| 1 | InvalidOrUnsupportedRequest |
| 2 | NoInstrumentsFound |
| 3 | NotAuthorized |
| 4 | InstrumentDataTemporarilyUnavailable |
| 5 | RequestForInstrumentDataNotSupported |
