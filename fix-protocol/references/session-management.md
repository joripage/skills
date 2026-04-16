# Session Management

## Logon Flow (35=A)

```
Initiator                           Acceptor
    │                                   │
    │──── Logon (35=A) ───────────────►│
    │     MsgSeqNum=1                   │
    │     HeartBtInt=30                 │
    │     EncryptMethod=0              │
    │     [Username=554]               │
    │     [Password=925]               │
    │                                   │
    │◄─── Logon (35=A) ────────────────│
    │     MsgSeqNum=1                   │
    │     HeartBtInt=30                 │
    │                                   │
    │     SESSION ESTABLISHED           │
```

### Logon Rules
- Initiator sends first; acceptor responds with Logon to confirm
- HeartBtInt (108) must be agreed — acceptor may echo back or use its own
- EncryptMethod (98) typically 0 (None)
- ResetSeqNumFlag (141=Y) to reset both sides to 1
- If acceptor rejects: sends Logout (35=5) with Text reason, then disconnects
- Validate SenderCompID/TargetCompID match expected values
- Never log Password (554) or NewPassword (925)

## Heartbeat & TestRequest Flow

```
 HeartBtInt = 30 seconds

 Normal:
 ├── Any message resets the heartbeat timer
 ├── If no outbound message for 30s → send Heartbeat (35=0)
 └── If no inbound message for 30s + reasonable delay → send TestRequest (35=1)

 TestRequest Flow:
 Sender                              Receiver
    │                                   │
    │  (no message for HeartBtInt+delay)│
    │                                   │
    │──── TestRequest (35=1) ─────────►│
    │     TestReqID (112) = "TEST001"   │
    │                                   │
    │◄─── Heartbeat (35=0) ────────────│
    │     TestReqID (112) = "TEST001"   │  ← MUST echo back same TestReqID
    │                                   │

 Timeout:
 ├── If no Heartbeat response within HeartBtInt seconds after TestRequest
 └── → Disconnect (consider session lost)
```

### Heartbeat Rules
- Heartbeat timer resets on ANY outbound message (not just Heartbeats)
- Inbound timer resets on ANY inbound message
- Grace period for TestRequest: typically HeartBtInt * 1.2 or + a few seconds
- TestReqID (112) in response MUST match the request — otherwise treat as unsolicited heartbeat

## Sequence Number Management

### Rules
1. MsgSeqNum (34) starts at 1 for each session (or continues from last session)
2. Each side maintains independent inbound/outbound sequence counters
3. Outbound: increment by 1 for each message sent
4. Inbound: validate matches expected sequence number

### Gap Detection

```
Expected SeqNum = 10, Received SeqNum = 15

→ Gap detected (messages 10-14 missing)
→ Queue message 15 (process later)
→ Send ResendRequest (35=2):
    BeginSeqNo (7) = 10
    EndSeqNo (16) = 14  (or 0 for "all remaining")
→ Wait for resent messages or GapFill
```

### Too-Low Sequence Number

```
Expected SeqNum = 10, Received SeqNum = 5

Case 1: PossDupFlag (43) = Y
→ Process normally (it's a legitimate resend)
→ Check OrigSendingTime (122) < SendingTime (52)

Case 2: PossDupFlag (43) = N or absent
→ CRITICAL ERROR — sequence integrity broken
→ Send Logout with error text
→ Disconnect immediately
```

## ResendRequest Flow (35=2)

```
Side A                              Side B
    │                                   │
    │  (detects gap: expected 5, got 8) │
    │                                   │
    │──── ResendRequest (35=2) ───────►│
    │     BeginSeqNo (7) = 5            │
    │     EndSeqNo (16) = 7             │
    │                                   │
    │◄─── SequenceReset-GapFill ───────│ (for admin messages 5-6)
    │     MsgSeqNum=5                   │
    │     NewSeqNo (36) = 7             │
    │     GapFillFlag (123) = Y         │
    │     PossDupFlag (43) = Y          │
    │                                   │
    │◄─── Original Message (resent) ───│ (message 7, app-level)
    │     MsgSeqNum=7                   │
    │     PossDupFlag (43) = Y          │
    │     OrigSendingTime (122) = orig  │
    │                                   │
    │     (now process queued msg 8)    │
```

### ResendRequest Rules
- Admin messages (Logon, Logout, Heartbeat, TestRequest, ResendRequest, SequenceReset, Reject) → send GapFill, do NOT resend
- Application messages → resend with PossDupFlag=Y and OrigSendingTime
- EndSeqNo=0 means "infinity" (all messages from BeginSeqNo onward)
- Consecutive admin messages → combine into single GapFill
- GapFill NewSeqNo must be > MsgSeqNum (moves expected forward)

## SequenceReset (35=4)

### GapFill Mode (GapFillFlag=Y)
- Used during resend to skip admin messages
- NewSeqNo (36) = next expected sequence number after gap
- MsgSeqNum follows normal sequence
- Subject to sequence number validation

### Reset Mode (GapFillFlag=N or absent)
- Emergency reset — use sparingly
- NewSeqNo (36) = new sequence number to expect
- MUST NOT be processed through normal sequence validation
- Only forward resets allowed (NewSeqNo > current expected)

## Logout Flow (35=5)

```
Normal Logout:
Side A                              Side B
    │                                   │
    │──── Logout (35=5) ──────────────►│
    │     Text = "End of trading day"   │
    │                                   │
    │◄─── Logout (35=5) ───────────────│
    │     Text = "Confirmed"            │
    │                                   │
    │     DISCONNECT                    │

Error Logout (initiated by acceptor):
    │◄─── Logout (35=5) ───────────────│
    │     Text = "MsgSeqNum too low"    │
    │                                   │
    │──── Logout (35=5) ──────────────►│
    │                                   │
    │     DISCONNECT                    │
```

### Logout Rules
- Always respond to Logout with Logout before disconnecting
- Wait reasonable time for Logout response before forcing disconnect
- Persist final sequence numbers for session recovery
- If Logout contains error → log the Text (58) for debugging

## SendingTime Validation

```
Rule: |SendingTime - ReceivingTime| ≤ MaxTimeDelta (default 120 seconds)

If violated:
→ Send Reject (35=3) with:
    SessionRejectReason (373) = 10 (SendingTime accuracy problem)
    RefSeqNum (45) = MsgSeqNum of offending message
→ Send Logout (35=5)
→ Disconnect
```

## CompID Validation

```
Incoming message MUST have:
    SenderCompID (49) = our expected TargetCompID
    TargetCompID (56) = our SenderCompID

If mismatch:
→ Send Reject (35=3) with:
    SessionRejectReason (373) = 9 (CompID problem)
→ Send Logout (35=5) with error text
→ Disconnect
```

## Session State Summary

```
┌────────────┐  Logon sent   ┌────────────┐  Logon received  ┌────────────┐
│Disconnected│──────────────►│  LogonSent │─────────────────►│  Active    │
└────────────┘               └────────────┘                  └─────┬──────┘
      ▲                                                            │
      │                      ┌────────────┐  Logout confirmed │
      └──────────────────────│LogoutSent  │◄───────────────────┘
           disconnected      └────────────┘  Logout sent
```
