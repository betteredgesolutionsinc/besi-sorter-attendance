# BESI Sorter Attendance

Mobile-first sorter attendance portal for GitHub Pages + Supabase. This project is independent from the BESI App data. It uses email/password sign-in, Supabase password recovery, GPS-only warehouse geofence checks, separate employee and management views, and cutoff DTR/payroll calculations without undertime.

## What is ready now

- GitHub Pages front end with sorter and management sign-in screens.
- Supabase SQL schema with employee, profile, geofence, attendance, audit, and cutoff payroll views.
- Server-side SQL attendance function. It checks the active user, current location/radius and GPS accuracy, creates server timestamps, pairs Time Out with the open shift, and records accuracy.
- Management-only invitation Edge Function. It links an invited email to an existing roster row. Email addresses can be added later.
- Supabase native password recovery flow.
- Employee history and management cutoff views; overtime is calculated and undertime is not.

The portal is not connected to BESI-App tables and does not include employee names, emails, passwords, or secret keys in the public files.

## Configure Supabase

1. Create a separate Supabase project for BESI Sorter Attendance.
2. Run `supabase/schema.sql` in the Supabase SQL Editor.
3. In Authentication settings, enable email/password sign-in, configure the invite and password-recovery redirect URLs, and configure a production SMTP sender before inviting the full workforce.
4. Create the first management account in Supabase Auth. Copy its Auth user UUID, then add a management profile in SQL:

   ```sql
   insert into public.profiles (id, role)
   values ('PASTE_AUTH_USER_UUID', 'management');
   ```

5. Deploy the `invite-sorter` Edge Function. Set `SUPABASE_URL`, `SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`, and `APP_REDIRECT_URL` as Supabase function secrets. Never put the service role key in GitHub or `config.js`.
6. Create `config.js` from `config.example.js`; fill in the Supabase project URL and publishable key only.
7. In `public.geofence_settings`, update latitude, longitude, and radius to the confirmed warehouse pin. Defaults are only a starting point based on the warehouse location previously provided; test actual phones before rollout.

## Load the roster before emails are ready

The private `BESI_Sorter_Roster_OCW_Template.csv` contains the 136 active sorter names with the OCW number column left blank until you provide the OCW list. After matching and filling in every OCW number, import it through Supabase Table Editor into `sorter_employees`; it is not included in the public app ZIP. Or use the management interface to add roster rows individually. OCW numbers must be unique. Never commit the roster import to a public GitHub repository.

When emails are available, management selects an employee and invites their unique email. Supabase sends a secure setup link where the employee establishes their own password. Employees can use **Forgot password** later. They must use a mailbox they can access.

## Publish with GitHub Pages

1. Put the contents of `public/` at the repository's published root, or configure Pages to publish that folder with a workflow.
2. Commit only app source and `config.js` with the publishable key. Do not commit employee exports, temporary credentials, secret keys, or private data.
3. In Supabase Auth URL Configuration, add the live GitHub Pages URL to Site URL and Redirect URLs.
4. Publish the selected branch/folder in GitHub Pages. GitHub Pages hosts static files; Supabase provides authentication and database access.

## Attendance rules

- One attendance row per employee per Manila work date.
- Time Out closes the employee's open Time In, including overnight shifts.
- The database supplies timestamps; browser time is not used for payroll.
- Location is checked at each punch. It does not track employees continuously.
- The database saves GPS coordinates and the reported accuracy for each punch.
- A cutoff runs from day 1–15 or day 16 through month end.
- Rendered hours subtract configured break minutes. OT is the amount above scheduled hours. No undertime is calculated.
- Employees can read only their own profile and DTR. Management can read roster and attendance under row-level security.

## Test checklist

- Invite a test employee and accept the invitation.
- Sign in, reset the password, and verify that employee history is private.
- Test Time In/Out on a real phone inside and outside the configured radius.
- Test GPS accuracy indoors with Location and Wi-Fi enabled.
- Test an overnight shift and both cutoff ranges.
- Verify management can see all employees and employees cannot query other employees' rows.
- Confirm wage and OT rate before using exported figures for payroll.

The browser submits its GPS reading to the database function. This validates the geofence, but device GPS can be inaccurate or spoofed; the gatepass process remains a supporting operational control.
