# GitHub Copilot Instructions for JanisPropify

## 🏗 Project Architecture & Context
- **Framework**: Django 5.x (Monolith) with Django Rest Framework (DRF).
- **Database**: SQL Server (via `mssql-django`).
- **Environment**: Windows / PowerShell.
- **Key Apps**:
  - `properties`: Core real estate logic, **includes WhatsApp/Lead models**.
  - `users`: Custom auth (`CustomUser`), Roles, Departments.
  - `security`: Device verification & access control.
  - `whatsapp`: Webhook handling & integration logic.
- **Authentication**: 
  - Session Auth for Web Dashboard (`/dashboard/`).
  - JWT (`simplejwt`) for Mobile APIs (`/api/token/`).

## 🚀 Critical Developer Workflows
- **Terminal**: Use PowerShell syntax (e.g., `.\venv\Scripts\Activate.ps1`).
- **Run Server**: `python manage.py runserver`
- **Migrations**: `python manage.py makemigrations` / `python manage.py migrate`
- **Dependencies**: `pip install -r requirements.txt`

## 🔐 Security & PII Handling
- **Encryption**: Sensitive user data (names, phones) uses `django-encrypted-model-fields`.
  - ⚠️ **Warning**: You cannot perform standard database lookups (like `__icontains`) on `EncryptedCharField`.
  - Use `EncryptedCharField` for any new PII fields.
- **Device Verification**: Login flow includes device verification (`security` app). Users are `is_active=False` until approved/verified.

## 📱 WhatsApp CRM & UTM Tracking
- **Data Flow**: 
  1. `PropertyWhatsAppLink` generates a `wa.me` URL with UTM params & `tracking_id`.
  2. User clicks -> Opens WhatsApp -> Sends message.
  3. Meta Webhook hits `whatsapp.views.whatsapp_webhook`.
  4. System creates/updates `Lead` and `WhatsAppConversation`.
- **Models Location**: Note that `Lead`, `WhatsAppConversation`, and `PropertyWhatsAppLink` are defined in **`properties/models.py`**, NOT `whatsapp/models.py`.
- **Tracking**: `unique_identifier` in `PropertyWhatsAppLink` is key for attributing leads to specific campaigns/properties.

## 📝 Coding Conventions
- **Language**: Comments and documentation in **Spanish**. Variable/Function names in **English** (mostly), but some business logic terms may be Spanglish.
- **Text Normalization**: Use `TitleCaseMixin` for models with string fields to ensure consistent formatting.
- **Views**: 
  - `dashboard/` views often use standard Django templates.
  - `dashboard/api/` views use DRF Serializers.
  - Some "Ultra Simple" views return raw HTML strings (e.g., `simple_properties_view`).

## 📂 Key Files
- `properties/models.py`: Core domain models + WhatsApp CRM models.
- `whatsapp/views.py`: Webhook entry point.
- `janis_core3/settings.py`: Configuration (SQL Server, Apps, Auth).
- `WHATSAPP_CRM_README.md`: Detailed WhatsApp integration docs.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/propifyperu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
