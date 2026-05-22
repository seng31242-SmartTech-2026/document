# UC-01: Register Account

## Use Case ID
UC-01

## Use Case Name
Register Account

## Primary Actors
Buyer, Seller

## Preconditions
- User is not currently registered.
- User has access to a valid email address.

## Postconditions – Success
- A new user account is created.
- Verification email is sent.
- User account becomes active after verification.

## Postconditions – Failure
- No account is created.
- User receives an appropriate error message.

## Main Flow
1. User navigates to SmartTech homepage.
2. User clicks **Register**.
3. System displays registration form.
4. User enters details.
5. User clicks **Create Account**.
6. System validates input.
7. System creates account with status **Unverified**.
8. System sends verification email.
9. User clicks verification link.
10. System activates account.
11. User is redirected to dashboard.

## Alternative Flows
### AF-01
If email already exists, system shows an error.

### AF-02
If password is weak, system displays validation errors.

### AF-03
If verification token expires, system allows resend.

## Business Rules
- Email must be unique.
- Password must contain uppercase, number, and special character.
- Unverified users cannot transact.
