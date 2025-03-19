# **Adding Google Social Login to Cat Collector (Step-by-Step Guide)**  

_This guide assumes you have completed Cat Collector and want to add Google social login. We will manually add Google credentials in the Django Admin panel instead of using an `.env` file._

---

## **🛠 Step 1: Install Required Packages**  
Run the following command to install `django-allauth` and its dependencies:

```sh
pipenv install django-allauth requests PyJWT cryptography
```

🔹 **Why?** Django-Allauth depends on these libraries, and missing them can cause errors.

---

## **🛠 Step 2: Update `settings.py`**
Modify `INSTALLED_APPS` to include the required apps:

```python
INSTALLED_APPS = [
    'django.contrib.sites',  # Required for Django-Allauth
    'allauth',
    'allauth.account',
    'allauth.socialaccount',
    'allauth.socialaccount.providers.google',  # Enable Google login
]
```
And **add these settings.py**:

```python
SITE_ID = 1  # Required for Django-Allauth

AUTHENTICATION_BACKENDS = [
    'django.contrib.auth.backends.ModelBackend',
    'allauth.account.auth_backends.AuthenticationBackend',
]

SOCIALACCOUNT_PROVIDERS = {
    'google': {
        'SCOPE': ['email', 'profile'],
        'AUTH_PARAMS': {'access_type': 'online'},
    }
}
```

🔹 **Why?**  
- `django.contrib.sites` is required for social authentication.
- `AUTHENTICATION_BACKENDS` ensures Django-Allauth works alongside Django’s default auth system.

---

## **🛠 Step 3: Apply Migrations in the Correct Order**
Run migrations, making sure `sites` is applied first:

```sh
python3 manage.py migrate sites  # Migrate sites first
python3 manage.py migrate  # Apply all other migrations
```

🔹 **Why?** Running `sites` first prevents migration errors.

---

## **🛠 Step 4: Add URLs for Social Login**  
Update `catcollector/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('main_app.urls')),

    # Include allauth before Django's built-in auth URLs
    path('accounts/', include('allauth.urls')),
    path('accounts/', include('django.contrib.auth.urls')),
]
```

🔹 **Why?**  
- `allauth.urls` provides social login routes like `/accounts/google/login/`.  
- It must come **before** `django.contrib.auth.urls` to avoid conflicts.

---

## **🛠 Step 5: Set Up Google OAuth Credentials**
1. **Go to [Google Developer Console](https://console.developers.google.com/)**.  
2. **Create a new project** (or use an existing one).  
3. **Enable OAuth 2.0 Client ID**:
   - Go to **Credentials** → **Create Credentials** → **OAuth 2.0 Client ID**.
   - Configure the **OAuth consent screen**.
   - Set **Authorized redirect URI** to:
     ```
     http://localhost:8000/accounts/google/login/callback/
     ```
4. **Copy the Client ID and Secret Key**.

---

## **🛠 Step 6: Manually Add Google Social Login in Django Admin**
1. **Run your Django server**:
   ```sh
   python3 manage.py runserver
   ```
2. **Go to Django Admin** (`http://localhost:8000/admin/`).
3. **Check if "Sites" exists**:
   - If missing, go to **Sites → Add Site**.
   - Set:
     - **Domain Name**: `localhost`
     - **Display Name**: `Localhost`
   - Save it.
4. **Add Google as a Social Application**:
   - Go to **Social Applications → Add**.
   - **Provider**: Google
   - **Name**: Google Login
   - **Client ID**: Paste from Google Developer Console
   - **Secret Key**: Paste from Google Developer Console
   - **Sites**: Select `localhost`
   - Save it.

🔹 **Why?**  
- This registers Google as an OAuth provider without modifying `settings.py`.

---

## **🛠 Step 7: Add a "Login with Google" Button**
Update your `home.html` template:

```html
{% load socialaccount %}
<a href="{% provider_login_url 'google' %}">Login with Google</a>
```

🔹 **Why?** This dynamically generates the correct Google login URL.

---

## **🛠 Step 8: Restart Django & Test Login**
```sh
python3 manage.py runserver
```
Click the **"Login with Google"** button and test authentication.

---

### **🎯 Done!**
Now, your Cat Collector app supports Google social login. 🚀 If you run into errors, check:
- That **all dependencies are installed**.
- That **Django migrations were applied in the correct order**.
- That **Google OAuth credentials are correctly entered in Django Admin**.

---

### **💡 Troubleshooting**
**Problem:** _"No module named requests"_
- **Fix:** Run `pipenv install requests`  

**Problem:** _"No module named cryptography"_
- **Fix:** Run `pipenv install cryptography`  

**Problem:** _"URL does not exist" for `{% provider_login_url 'google' %}`_
- **Fix:** Ensure `'allauth.socialaccount.providers.google'` is in `INSTALLED_APPS`  
- Restart Django and try again.

**Problem:** _"InconsistentMigrationHistory"_
- **Fix:** Run migrations in the correct order:  
  ```sh
  python3 manage.py migrate sites
  python3 manage.py migrate
  ```

Let me know if you need any clarifications! 🚀
