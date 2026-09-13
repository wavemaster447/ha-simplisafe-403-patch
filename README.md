# SimpliSafe Home Assistant Workaround (403 `authCheck` Fix)

A temporary custom component overlay for Home Assistant to fix the SimpliSafe authentication failure returning `403 Forbidden` on `api/authCheck`.

---

## 🚨 The Issue

When attempting to set up or re-authenticate the built-in SimpliSafe integration in Home Assistant, the initial OAuth authorization flow succeeds in issuing an access token, but initialization fails with:

```text
403 Forbidden from api/authCheck
```

SimpliSafe's backend currently rejects calls to `api/authCheck` even when presented with a valid access token.

---

## 💡 The Workaround

This repository backports the fix proposed in [bachya/simplisafe-python#1159](https://github.com/bachya/simplisafe-python/pull/1159):

1. Catches the `403 Forbidden` response when `api/authCheck` is called.
2. Decodes the JWT `access_token` locally.
3. Extracts the numeric user ID claim (`http://simplisafe.com/uid`).
4. Sets `self.user_id` directly, bypassing the broken endpoint and allowing Home Assistant to finish setting up the integration.

---

## 📦 Installation

### Manual Installation

1. Download or clone this repository.
2. Copy the `simplisafe` directory into your Home Assistant configuration directory:
   ```text
   /config/custom_components/simplisafe/
   ```
3. Restart **Home Assistant**.
4. Go to **Settings** ➔ **Devices & Services** ➔ **Add Integration** ➔ **SimpliSafe**.
5. Complete the login flow.

---

## 🌐 Browser Authentication Tip (If SMS Submit Fails)

If the Auth0 browser login page stalls on the SMS verification code (inert "Continue" button):

1. Type in your SMS code.
2. Open Browser DevTools (**F12** or **⌘⌥J**) ➔ **Console**.
3. Paste and run this snippet:
   ```js
   const f = document.querySelector('form');
   const h = document.createElement('input');
   h.type = 'hidden'; h.name = 'action'; h.value = 'default';
   f.appendChild(h);
   HTMLFormElement.prototype.submit.call(f);
   ```
4. Copy the `code=XXXXX` parameter from the console log when it fails to open `com.simplisafe.mobile://...` and paste it into Home Assistant.

---

## 🔗 Credits & References

- Code and documentation from Gemini/Antigravity
- Home Assistant Core Issue: [#178577](https://github.com/home-assistant/core/issues/178577)
- `simplisafe-python` PR: [bachya/simplisafe-python#1159](https://github.com/bachya/simplisafe-python/pull/1159)
