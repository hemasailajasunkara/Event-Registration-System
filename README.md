# CampusConnect — Campus Event & Fest Registration

CampusConnect is a Java 17 + Spring Boot + H2 application for managing college fests and independently registerable fest activities.

## What's included

- Separate Admin workspace and Student portal (the role switch is a UI view switch, not authentication).
- Main fests plus independently registerable **Stalls** and **Activities**.
- Stall categories include Food, Jewellery, Water & Drinks, Clothing & Accessories and Other.
- Activity categories include Dance, Singing, Games, Technical, Cultural and Other.
- A student can register for a stall/activity without registering for the parent fest.
- Each fest, stall or activity has its own date, venue, capacity, registration fee and registration list.
- Free registrations are confirmed directly. Paid registrations use Razorpay Checkout and require backend signature verification.
- Capacity and remaining seats are shown for every registration option. Existing registrations are preserved when an item is archived or its capacity is changed.
- No Attendance module.

## Requirements

- Java 17
- Maven 3.8+
- Internet connection for Google Fonts and Razorpay Checkout script. The app itself serves the frontend locally.

## Run on Windows

1. Extract the ZIP.
2. Open the `event-registration-system` folder in VS Code or a terminal.
3. Run `run-windows.bat`, or run `mvn spring-boot:run`.
4. Open http://localhost:8080.

## Razorpay test configuration

Paid registrations require Razorpay test credentials. Set them as environment variables on your computer before starting Spring Boot:

```powershell
$env:RAZORPAY_KEY_ID="rzp_test_your_key_id"
$env:RAZORPAY_KEY_SECRET="your_test_key_secret"
mvn spring-boot:run
```

Never put the key secret in frontend JavaScript or commit it to GitHub. Without credentials, free registrations and the rest of the application work; paid checkout will show a configuration message.

## How to create a fest with stalls and activities

1. In Admin → Fests & activities, create a **Main fest / event** (example: TechnoCultural Fest).
2. Create a **Stall registration** and choose the parent fest. Select Food, Jewellery, Water & Drinks or another category.
3. Create an **Activity / competition** and choose the parent fest. Select Dance, Singing, Games, Technical or another category.
4. Set a separate capacity and fee for each item. Use fee `0` for free registration.
5. In Student portal, a student chooses their profile and registers directly for the desired item. Registering for the parent fest is not required.

## Database and existing data

The H2 file database is stored under `./data/eventdb`. The schema uses additive `ALTER TABLE ... ADD COLUMN IF NOT EXISTS` statements to add `event_type`, `parent_event_id` and `category` to an existing `events` table. Existing event rows are treated as Main fests, and existing registrations/payment history remain associated with those events.

**Back up the `data` folder before replacing project files or changing the schema.** Do not delete the H2 database if you want to preserve existing student, event, registration and payment records.

## Important limitations

- This is a project/demo application. Admin and Student are separate screens, but there is currently no login, role-based authorization or student identity verification. Do not expose it publicly with real student data until authentication and authorization are implemented.
- Razorpay integration is a test-mode flow. Configure test credentials and test it before any demonstration involving paid checkout.
- Payment verification confirms the Razorpay signature. A production deployment should additionally fetch the payment from Razorpay and validate captured status, amount and currency, implement webhook reconciliation and a refund workflow for edge cases.

## Separate Admin and Student Login

CampusConnect now uses server-side HTTP sessions and role checks on every `/api/**` endpoint. Students cannot open administrator API routes by typing a URL or calling the endpoint directly.

### First administrator login

On the first start, the application creates one administrator account if it does not already exist:

- Username: `admin` (or the `ADMIN_USERNAME` environment variable)
- Initial password: `ChangeMe@123` (or `ADMIN_PASSWORD` if set on the first start; use at least 10 characters)

For PowerShell, set a private password before the first launch:

```powershell
$env:ADMIN_USERNAME="admin"
$env:ADMIN_PASSWORD="Choose-A-Strong-Password"
mvn spring-boot:run
```

If the administrator account has already been created, changing the environment variable will not overwrite its existing password hash. Do not deploy using the default password.

### Student login setup

Student login uses the college roll number and password. In the Admin portal, open **Students** and create a student with an initial password of at least 8 characters. For students already present in an older database, choose **Edit** and set a password in **Initial / reset password**; leaving it blank keeps their current password. The password is stored as a salted PBKDF2 hash, never as plain text.

### Permissions

- Admin: event/fest/stall/activity management, student directory, all registrations, payment records and dashboard.
- Student: event discovery, their own registrations, and registration/payment checkout. Student IDs are taken from the signed-in session, not trusted from browser form values.
- Signing out invalidates the session. The H2 web console is disabled.

Existing student records are preserved during schema initialization, but they must be assigned a password by an administrator before those students can sign in.


## Demo payment mode

CampusConnect now uses a built-in simulated payment checkout for paid registrations. No Razorpay account, PAN verification, API keys, card details, or real money are required. In the student portal, choose a paid fest/stall/activity and continue to the demo checkout. Select **Simulate successful payment** to create a PAID payment record and confirm the registration, or **Simulate failed payment** to create a FAILED payment record without creating a registration. This is for academic demonstrations only and is not a real payment gateway.
