# Security Specification: IIUC Matrimony

## Data Invariants
1. A user can only create one profile, and it must match their Auth UID.
2. Users can only send interests if they have a completed profile.
3. Messaging is only allowed if an interest has been 'accepted' between the two participants.
4. `isVerified` can only be set by an admin (or a system process).
5. Users can only read their own private PII (email, guardian contact).

## The "Dirty Dozen" Payloads (Testing Denial)

1. **Identity Spoofing**: Create a profile with `uid` different from `request.auth.uid`.
2. **Elevated Privilege**: Create/Update profile with `isVerified: true` as a regular user.
3. **Ghost Field**: Add `admin: true` to a profile document.
4. **Orphaned Interest**: Send an interest where `senderId` is not the current user.
5. **Illegal State**: Directly create an interest with `status: 'accepted'`.
6. **Bypassing Consent**: Send a message to a conversation where no accepted interest exists.
7. **Resource Poisoning**: Send a message with 1MB of junk text.
8. **PII Leak**: Read another user's profile and see their `email` or `guardianContact`.
9. **Immortality Breach**: Change `createdAt` on an existing profile.
10. **Shadow Update**: Update a profile and try to change `gender` after it's set.
11. **Spoofing Verification**: Update `isVerified` on someone else's profile.
12. **Double Interest**: Create multiple interest documents for the same pair (handled by application logic but rules should prevent unauthorized writes).

## Implementation Strategy
- Use `isValidProfile`, `isValidInterest`, and `isValidMessage` helpers.
- Use `existsAfter` to verify reciprocal interest for messaging.
- Role-based isolation for PII.
