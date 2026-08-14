# key generation for self signing android packages

## prerequisites

The installed JDK must include keytool.

## 1. generate key file

```
keytool -genkeypair `
  -v `
  -keystore "<PATH>\<APP>-release.jks" `
  -alias <ALIAS>`
  -keyalg RSA `
  -keysize 4096 `
  -validity 10000
```
For the keytool password, I would keep it simple and robust:

Characters: Ideally, use ASCII characters, like this:
- A-Z
- a-z
- 0-9
- optional !#$%&*+-=?@_
- No umlauts (äöüß)
- No exotic Unicode characters.
- I would also avoid spaces.

## 2. generate base64 value

```
[Convert]::ToBase64String(
    [IO.File]::ReadAllBytes("<PATH>\<APP>-release.jks")
) | Set-Clipboard
```

This copies the content directly to the clipboard.

## gitHub secrets

RELEASE_KEYSTORE_PASSWORD must contain the password used when generating the key (step 1)
RELEASE_KEYSTORE_BASE64 must contain the base64 value (step 2)

## backup

Make sure to backup both the keyfile and the password!