# RMACTS Learning Box

A Flutter starter project for a school learning library that targets **web and Android** from one codebase.

## Features in this starter

- Email/password sign-in (no public sign-up)
- Admin and student roles
- Students can browse and open learning materials
- Only an administrator can upload or delete material records/files
- Private Supabase Storage bucket with short-lived signed download links
- Responsive Material 3 interface
- PDF, Word, PowerPoint, images and MP4 upload selection (50 MB app-side limit)

This is a starter, not a fully audited production system. Add backups, account recovery, admin management, upload validation and testing before real-world deployment.

## 1. Create Supabase project

1. Create a project at https://supabase.com/dashboard.
2. Open SQL Editor and run `supabase/schema.sql`.
3. Open Storage, create a bucket named `learning-materials`, and make sure it is **Private**.
4. Set the bucket's file size limit to 50 MB. Set allowed MIME types to:
   - `application/pdf`
   - `application/msword`
   - `application/vnd.openxmlformats-officedocument.wordprocessingml.document`
   - `application/vnd.ms-powerpoint`
   - `application/vnd.openxmlformats-officedocument.presentationml.presentation`
   - `image/jpeg`
   - `image/png`
   - `video/mp4`
5. In Project Settings / API (or Connect), copy the **Project URL** and **publishable key**. Never put the `service_role` key or any secret key in the client app.

## 2. Create the first administrator

1. In Supabase, open Authentication → Users → Add user. Create an administrator account and confirm it as appropriate for your project settings.
2. Copy that user's UUID.
3. In SQL Editor, run this query, replacing the UUID and name:

```sql
insert into public.profiles (id, full_name, role)
values ('REPLACE-WITH-AUTH-USER-UUID', 'RMACTS Administrator', 'admin');
```

Create student accounts in Authentication → Users. For each student, add a profile row with role `student`:

```sql
insert into public.profiles (id, full_name, role)
values ('REPLACE-WITH-STUDENT-UUID', 'Student Name', 'student');
```

Do not allow users to edit their own role. Only trusted project administrators should manage profiles and roles. The client has no public sign-up screen by design.

## 3. Run online in GitHub Codespaces

Create a private GitHub repository, upload/extract this project, and open it in a Codespace. In its terminal, install Flutter on the cloud machine:

```bash
sudo apt-get update
sudo apt-get install -y git curl unzip xz-utils zip
git clone https://github.com/flutter/flutter.git -b stable "$HOME/flutter"
export PATH="$HOME/flutter/bin:$PATH"
flutter doctor
flutter config --enable-web
```

If Flutter already exists at `$HOME/flutter`, do not clone it again; just run the `export PATH=...` command.

In the project directory:

```bash
flutter create --platforms=web,android .
flutter pub get
```

Run the web app:

```bash
flutter run -d web-server \
  --web-hostname 0.0.0.0 \
  --web-port 8080 \
  --dart-define=SUPABASE_URL=https://YOUR_PROJECT.supabase.co \
  --dart-define=SUPABASE_PUBLISHABLE_KEY=YOUR_PUBLISHABLE_KEY
```

Replace both placeholders. In Codespaces, open the Ports tab and open port 8080 in a browser. Keep the port private unless you intentionally want to share the preview.

## 4. Publish the website

The simplest beginner-friendly route is to build the Flutter web files in the Codespace and deploy the `build/web` folder using a static host such as Cloudflare Pages or Netlify.

```bash
flutter build web \
  --dart-define=SUPABASE_URL=https://YOUR_PROJECT.supabase.co \
  --dart-define=SUPABASE_PUBLISHABLE_KEY=YOUR_PUBLISHABLE_KEY
```

Deploy the contents of `build/web` to your chosen static host. Configure the host's publish directory as `build/web` if it builds from the repository. Configure SPA fallback to `/index.html` if needed by your host. Add your production website URL to Supabase Authentication → URL Configuration / Redirect URLs.

The Supabase publishable key is designed for client use; security depends on Row Level Security and Storage policies, not on hiding this key.

## 5. Build the Android APK in the cloud

Flutter Android builds need the Android SDK and Java toolchain on the cloud machine. If your Codespace image does not include them, configure an Android-capable cloud build environment or use a CI service such as GitHub Actions.

Once Android SDK and Java are installed and accepted in the build environment:

```bash
flutter create --platforms=web,android .
flutter pub get
flutter build apk --release \
  --dart-define=SUPABASE_URL=https://YOUR_PROJECT.supabase.co \
  --dart-define=SUPABASE_PUBLISHABLE_KEY=YOUR_PUBLISHABLE_KEY
```

The APK is produced at:

`build/app/outputs/flutter-apk/app-release.apk`

Do not publish the APK until you have tested sign-in, student permissions, administrator upload restrictions, signed file links, and app behavior on actual Android devices. For Google Play, an Android App Bundle (`flutter build appbundle --release`) is usually preferred over an APK.

## Security notes

- Never ship a Supabase `service_role` key or secret key in Flutter.
- The app intentionally has no self-registration.
- Create/assign roles through the Supabase dashboard or trusted server-side administration.
- RLS and Storage policies enforce admin-only uploads; hiding the upload button alone is not security.
- Signed download links expire after five minutes.
- For production, add file scanning/validation, rate limits, audit logs, backups, password reset, account disable workflows, and tests.
