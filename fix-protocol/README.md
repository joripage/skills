# FIX Protocol Skill

Expert guidance for FIX (Financial Information eXchange) protocol implementation, debugging, and validation. Covers FIX 4.2 / 4.4 / 5.0 SP2.

## Installation

Copy the `fix-protocol/` folder to your Claude skills directory:
```bash
cp -r fix-protocol ~/.claude/skills/
```

## Usage

```bash
/fix-protocol "your question or task about FIX protocol"
```

## Examples

```bash
# Ask about message flows
/fix-protocol "What reject should I send for an invalid NewOrderSingle?"
/fix-protocol "When should I use 35=j vs 35=3?"
/fix-protocol "Explain the cancel/replace flow for 35=G"

# Implementation help
/fix-protocol "Review my ExecutionReport builder for correctness"
/fix-protocol "Help implement ResendRequest handler"
/fix-protocol "Validate my order state machine transitions"

# Debugging
/fix-protocol "Why is my sequence number gap fill not working?"
/fix-protocol "Order stuck in PendingCancel state"
```

## What's Included

### SKILL.md (Main)
- **Message Flow Matrix** — request → success/reject response mapping for all message types
- **Critical Rules** — 35=3 vs 35=j decision logic, reject routing per MsgType
- **Implementation Checklists** — session layer, order management, market data, security
- **Tag Quick Reference** — most-used FIX tags with descriptions

### References (loaded on-demand)

| File | Contents |
|------|----------|
| `references/order-state-machine.md` | OrdStatus/ExecType transitions, state diagram, quantity rules, ClOrdID chain |
| `references/session-management.md` | Logon/Logout flows, heartbeat/TestRequest, sequence numbers, ResendRequest, GapFill |
| `references/error-codes.md` | SessionRejectReason, BusinessRejectReason, OrdRejReason, CxlRejReason, all enum values |
| `references/required-fields.md` | Required tags per message type (D, 8, F, G, 9, j, V, Y, A, etc.) |
| `references/fix-version-differences.md` | FIX 4.2 vs 4.4 vs 5.0 differences, ExecTransType removal, OnixS engine notes |

## Key Rules at a Glance

| Incoming Message | Reject Via | Never Use |
|------------------|------------|-----------|
| 35=D (NewOrderSingle) | 35=8 (ExecutionReport, OrdStatus=8) | 35=j |
| 35=F (OrderCancelRequest) | 35=9 (OrderCancelReject, CxlRejResponseTo=1) | 35=j |
| 35=G (OrderCancelReplaceRequest) | 35=9 (OrderCancelReject, CxlRejResponseTo=2) | 35=j |
| 35=V (MarketDataRequest) | 35=Y (MarketDataRequestReject) | 35=j |
| Unknown MsgType | 35=j (BusinessMessageReject) | 35=8/9 |
| Malformed message | 35=3 (Session Reject) | 35=j |

## Auto-Activation

The skill auto-activates when Claude detects FIX-related work:
- FIX message handlers or parsers
- FIX gateway / OMS code
- FIX session management
- FIX tag references in code
